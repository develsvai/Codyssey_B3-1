# 트러블슈팅 기록

## 사례 1 — CLI 로그인은 됐지만 VPC 조회가 거절됨

### 증상

초기 CLI 연결 뒤 IAM 사용자 인증은 성공했지만 `ec2:DescribeVpcs` 호출에서 `UnauthorizedOperation`이 발생했다. 오류는 이 사용자를 허용하는 identity-based policy가 없다는 내용이었다. 이는 이전 인증 설정 과정에서 실제로 발생한 문제이며, 아래 기록은 그 관찰과 이후 검증을 정리한 것이다.

### 가설

CLI 로그인용 `SignInLocalDevelopmentAccess`만으로는 EC2/VPC 조회·생성 권한이 생기지 않는다. 브라우저에 로그인되어 있다는 사실과 서비스 자원에 대한 권한은 별개다.

### 검증

1. `sts get-caller-identity`로 `Haru2_freetier_iam_user`가 맞는지 확인했다.
2. 같은 프로파일의 VPC 조회가 거절되는 것을 확인했다.
3. AWS 공식 정책 문서에서 SignInLocalDevelopmentAccess가 OAuth 인증 권한을 제공하는 정책임을 확인했다.

### 조치

과제에 필요한 EC2 계열 작업을 추린 사용자 지정 관리형 정책을 만들었다. 서울 리전, Project/Owner 태그, t3.micro, 작은 gp3 볼륨으로 범위를 제한했고, 사용자가 콘솔에서 자신의 IAM 사용자에게 연결했다.

### 결과

정책 연결 후 조회 21종이 성공했다. 태그를 포함한 VPC·인터넷 게이트웨이·키페어 등록 사전 검사는 통과했고, 태그 누락·다른 소유자·다른 리전·허용하지 않은 독립 볼륨 생성은 거절되었다. 이후 실제 인프라 생성, SSH 접속, 외부 HTTP 200까지 성공했다.

증빙: [권한 검사 결과](evidence/permissions-check.json), [생성된 자원 상태](evidence/provisioned-resources.json), [외부 HTTP 결과](evidence/external-http-check.json).

### 재발 방지

- 연결 직후 로그인 사용자를 먼저 확인한다.
- 인증 정책과 서비스 작업 정책을 구분한다.
- 권한 오류를 해결하기 위해 AdministratorAccess를 붙이지 않고, 오류의 작업 이름과 대상 자원을 확인한다.
- 생성 시 필수 태그를 함께 보내고, 기존 자원에서는 태그가 일치하는지 확인한다.
- 네트워크 문제와 IAM 권한 문제를 혼동하지 않는다.

참고: [AWS 로그인 정책](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/SignInLocalDevelopmentAccess.html), [EC2 정책 예시](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ExamplePolicies_EC2.html).

## 사례 2 — Nginx 설정 검사에서 MIME 해시 경고 발생

### 증상

첫 `nginx -t` 검사에서 `could not build optimal types_hash` 경고가 나왔다. 구성 문법 검사는 성공했고 웹페이지도 HTTP 200이었다. 접속 장애가 아니라 실제로 관찰된 설정 경고다.

### 가설과 검증

설치된 MIME 타입 목록에 비해 기본 `types_hash_max_size` 값이 작다고 판단했다. 경고가 타입 해시 크기를 직접 언급하며, 사용한 nginx.conf에는 별도의 크기 설정이 없음을 확인했다.

### 조치

`http` 블록에 `types_hash_max_size 4096;`을 추가했다. 수정한 설정을 서버에 반영하고 `sudo nginx -t`로 검사한 뒤 Nginx를 reload했다.

### 결과

수정 후 설정 검사에서 해당 경고가 사라졌고 `/health`의 `ok` 응답도 유지됐다. [최초 배포 로그](evidence/deployment-log.txt)와 [수정 후 검사 로그](evidence/nginx-config-check.txt)에 실제 출력이 있다.

### 재발 방지

검증된 [nginx.conf](../config/nginx.conf)를 저장소에 보관하고, 설정 변경 시 문법 검사 성공 후 reload한다.
