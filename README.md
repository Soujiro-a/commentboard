# Node.js Clone coding Project

주어진 템플릿을 기반으로 OAuth 2.0 기반 인증, 이메일 가입과 간단한 게시글 작성을 구현해본 프로젝트

Back-end는 NodeJS와 ExpressJS, Front-end는 Pug와 TailwindCSS로 구성하였음

```
# 플로우별 처리 과정

## OAuth flow

    - Vendoers
        - Naver
        - Facebook
        - Kakao
    - 각 OAuth provider에 개발자 계정을 만들고 설정 해야함
    - access token -> (platformUserId, platform) -> 회원가입/로그인 처리
    - HTTPS -> ngrok으로 일단 처리

## 이메일 가입

    - 인증 상태를 확인할 필요가 있음 -> 유저 정보에 'verified'라는 필드가 있어야함
        - 'verified' 필드가 'false'이면 정상적인 활동 불가능
    - 이메일 인증은 특수한 코드와 함께 이메일을 보내서 그 링크로 접속했을 때만 인증이 되게 처리.
        - "다음 링크로 들어와 인증을 마무리해 주세요 : https://내가만든도메인.com/verify_email?code=aaaa-bbb-1234-fghf"
        - "위 링크로 'GET'해서 들어오게 되면 'verified'를 'true'로 바꿔줌.
    - AWS에 있는 SES(Simple Email Service)를 사용하여 인증 메일을 보낼 것.

    - 비밀번호 초기화
        - 비밀번호 찾기는 지원하지 않음.
            - 찾기가 가능하다는 건 두 가지중 하나에 해당이 됨
                1. 양방향 변환(암호화, 복호화)이 가능
                => 개발자나 서버 키를 탈취한 사람이 본래의 비밀번호를 알 수 있는 기회를 만들게 됨
                2. plain text로 저장 되어 있다는 뜻
                => 누군가가 DB를 보기만해도 회원들의 암호를 알 수 있을 가능성이 생기기 때문에 보안상 좋지 않음.
        - 원래 암호를 매우 알기 어려운 해시 펑션(one-way function)을 사용하여 해시된 값을 데이터베이스에 저장.
        - 유저가 처음 가입한 해당 이메일로 인증 메일과 비슷하게 초기화 링크를 담은 메일을 보냄.
            - 해당 링크를 타고 들어오면 입력한 새 비밀번호(바꾸자하는 비밀번호)로 기존 비밀번호를 갱신시킴.

# 배포

AWS를 사용해서 배포할 예정

- EC2 (server)
  - git 레포지토리를 clone해서 배포
- HTTPS 지원 - Amazon 인증서
  - ELB (Elastic Load Balancer)를 사용해 여기에 인증서를 물리고, ELB가 뒤의 EC2를 바라보게 함
    - 인증서 자체는 ELB에 물려있고, 밖에서는 HTTPS 트래픽으로 통신하게 되고, ELB와 EC2가 격리된 곳에서만 HTTPS 통신을 하게 됨.
- SES를 통해 메일 처리.
- DB는 MongoDB를 사용.
```
