# Mastodon 서버 구축 가이드 (whippyshou/mastodon 기반)

이 문서는 [whippyshou/mastodon](https://github.com/whippyshou/mastodon) 리포지토리를 바탕으로, 서버에 대해 잘 모르는 초보자도 Mastodon 인스턴스를 직접 설치하고 운영할 수 있도록 구성된 단계별 가이드입니다.

---

## ✅ 이 문서가 도움이 되는 사람

- "서버? 그게 뭐야?" 싶은 분
- Mastodon 인스턴스를 처음 만들어보는 사람
- DigitalOcean, Cloudflare, Mailgun 같은 서비스가 생소한 사람

---

## 🧭 설치 흐름 요약

1. DigitalOcean에 서버 만들기
2. 도메인 구매 및 Cloudflare 연결
3. 리포지토리 클론 및 `.env` 파일 작성
4. Docker 및 Mastodon 실행
5. nginx 설정 및 HTTPS 적용
6. Mailgun 연동 (이메일 인증)
7. 배경, 로고, 색상 등 커스터마이징

---

## 1. DigitalOcean에 서버 만들기

1. [https://www.digitalocean.com/](https://www.digitalocean.com/) 접속 후 계정 생성
2. Droplet(서버) 생성 → Ubuntu 22.04 선택
3. 최소 사양: 2vCPU, 2GB RAM (추천: 4GB RAM 이상)
4. SSH 키 등록 또는 root 비밀번호 설정
5. 생성된 서버의 IP 주소 확인

---

## 2. 도메인 구매 및 Cloudflare 연결

1. 가비아 등에서 원하는 도메인 구매
2. [Cloudflare](https://www.cloudflare.com/)에 가입 → 사이트 추가 → 도메인 입력
3. Cloudflare에서 제시하는 네임서버로 가비아에서 네임서버 변경
4. Cloudflare에서 A 레코드 추가:
    - `@` → 서버 IP
    - `www` → 서버 IP
5. SSL 설정: **Full (strict)**
6. 프록시(Proxy) 활성화: 주황색 구름 아이콘 상태로

---

## 3. Mastodon 설치 (whippyshou/mastodon)

```bash
sudo apt update && sudo apt install git docker docker-compose -y

# 마스토돈 리포지토리 클론
git clone https://github.com/whippyshou/mastodon.git
cd mastodon

# .env.production 파일 생성 및 환경 변수 설정
cp .env.production.sample .env.production
vi .env.production  # 또는 nano 사용
```

예시 설정:
```
LOCAL_DOMAIN=yourdomain.kr
WEB_DOMAIN=yourdomain.kr
SMTP_SERVER=smtp.mailgun.org
SMTP_PORT=587
SMTP_LOGIN=postmaster@yourdomain.mailgun.org
SMTP_PASSWORD=비밀번호
SMTP_FROM_ADDRESS=notifications@yourdomain.kr
RAILS_SERVE_STATIC_FILES=true
```

---

## 4. Mastodon 실행

```bash
docker-compose build
docker-compose run --rm web bundle exec rake mastodon:setup  # 설정 마법사 실행
docker-compose up -d
```

---

## 5. nginx 설정 및 HTTPS 적용

1. nginx 설치:
```bash
sudo apt install nginx -y
```

2. `/etc/nginx/sites-available/mastodon` 생성:

```nginx
server {
    listen 80;
    server_name yourdomain.kr;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.kr;

    ssl_certificate /etc/letsencrypt/live/yourdomain.kr/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.kr/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }

    location /api/v1/streaming {
        proxy_pass http://localhost:4000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

3. HTTPS 인증서 발급 (Let's Encrypt):
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

---

## 6. Mailgun 연동 (이메일 인증)

1. [https://www.mailgun.com/](https://www.mailgun.com/) 가입 후 도메인 추가
2. DNS 레코드(Cloudflare) 설정 완료하면 인증됨
3. `.env.production`에 SMTP 설정 추가 (앞서 설정한 내용 참고)
4. 테스트 이메일 발송:
```bash
docker-compose run --rm web bin/tootctl email send-test you@example.com
```

---

## 7. 커스터마이징 (배경, 로고, 색상 등)

- `app/javascript/mastodon/styles/variables.scss`에서 색상 등 변경 가능
- 기본 프로필 이미지 변경: `app/javascript/images/avatars/original/missing.png`
- 변경 후:
```bash
docker-compose exec web rails assets:precompile
```

---

## 🎉 마쳤습니다!

이제 `https://yourdomain.kr`로 접속해 Mastodon 인스턴스를 사용할 수 있어요. 오류가 발생하면 함께 만든 [문제 해결 문서](./error-guide.md)를 참고하세요.

궁금한 점은 GitHub 이슈나 PR로 남겨 주세요!

