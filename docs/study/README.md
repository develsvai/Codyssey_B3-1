# 10월 19일 AWS 스터디 자료

**[웹에서 스터디 자료 열기](https://develsvai.github.io/Codyssey_B3-1/)** — 별도 로그인 없이 핵심 13장과 전체 52장을 넘겨볼 수 있다.

- [45분 스터디 노트](study-notes.md): 진행 순서와 질문, 핵심 캡처
- [브라우저로 넘겨보기](rebuild-20261017/index.html): 핵심 13장과 전체 52장, 원본 보기와 확대
- [전체 따라하기 글](rebuild-20261017/walkthrough.md): 클릭 순서와 입력값, 원본 로그
- [온라인 전체 자료](https://chatgpt.com/space/page_7d4e669d7c288191a35b2103480deed2)
- [실행 중인 웹페이지](http://43.201.62.16/)

브라우저 자료는 폴더 안의 index.html을 열면 된다. 공유할 때는 이미지가 함께 있는 폴더 전체를 보낸다. 새 서버는 19일까지 실행 상태로 유지하며 자동 종료는 설정하지 않았다.

## GitHub Pages 배포

공개 주소는 https://develsvai.github.io/Codyssey_B3-1/ 이다. 스터디 자료는 GitHub Pages에서 제공하므로 AWS 실습 서버를 정리한 뒤에도 볼 수 있다. 자료 안의 실제 EC2 웹페이지 링크는 서버 실행 상태에 따라 달라진다.

`master`에 `docs/study/` 또는 배포 설정을 수정해 푸시하면 자동으로 갱신된다. [배포 내역](https://github.com/develsvai/Codyssey_B3-1/actions/workflows/deploy-study-pages.yml)에서 성공 여부를 확인할 수 있고, 필요하면 수동 실행할 수 있다. 배포 대상은 `docs/study/` 폴더다.
