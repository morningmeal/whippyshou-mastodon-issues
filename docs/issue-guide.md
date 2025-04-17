# whippyshou-mastodon-issues
"whippyshou/mastodon 기반 Mastodon 서버 구축 중 발생한 오류와 해결 방법"

# Mastodon 설치 중 발생한 오류 모음 (whippyshou/mastodon 기반)

이 저장소는 [whippyshou/mastodon](https://github.com/whippyshou/mastodon) 리포지토리를 기반으로 Mastodon 서버를 구축하면서 발생했던 오류들과 그 해결 방법을 기록한 문서입니다.

## 목차

1. WebSocket (WSS) 관련 문제
2. 초기 접속 후 흰 화면
3. nginx 리버스 프록시 설정
4. Mailgun 이메일 연동 문제
5. 기타 팁

---

## 1. WebSocket (WSS) 관련 문제

### 증상

- HTTPS 페이지에서 다음과 같은 에러 발생:

```
Mixed Content: ... attempted to connect to the insecure WebSocket endpoint 'ws://...'
```

### 원인

- HTTPS 페이지에서 `ws://` WebSocket 연결을 시도했기 때문

### 해결 방법

- nginx 설정에서 `streaming` 서버에 대해 `wss://`로 리버스 프록시 설정

```nginx
location /api/v1/streaming {
    proxy_pass http://localhost:4000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
}
```

---

## 2. 초기 접속 후 흰 화면

### 증상

- 로그인 후 아무런 화면도 뜨지 않음

### 원인

- WebSocket 오류, static 파일 누락, 서비스워커 오류 등

### 해결 방법

- WebSocket 문제 해결 (위 참고)
- `RAILS_SERVE_STATIC_FILES=true` 설정 확인
- nginx가 `public/` 디렉토리를 서빙하도록 설정

---

## 3. nginx 리버스 프록시 설정

```nginx
server {
    listen 80;
    server_name "your server name";
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name "your server name";

    ssl_certificate /etc/ssl/certs/your_cert.crt;
    ssl_certificate_key /etc/ssl/private/your_key.key;

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

---

## 4. Mailgun 이메일 연동 문제

### 증상

```bash
Could not find command "emails"
```

### 해결 방법

- 올바른 명령어: `tootctl email send-test`

```bash
docker-compose run --rm web bin/tootctl email send-test your@email.com
```

- `.env.production` 설정 예시:

```
SMTP_SERVER=smtp.mailgun.org
SMTP_PORT=587
SMTP_LOGIN=postmaster@your-domain.mailgun.org
SMTP_PASSWORD=your-mailgun-password
SMTP_FROM_ADDRESS=notifications@your-domain
```

---

## 5. 기타 팁

- `.env.production`에 `LOCAL_DOMAIN`, `WEB_DOMAIN` 설정이 중요함
- Cloudflare를 사용하는 경우 SSL 모드는 `Full(strict)`으로 설정
- DNS Proxy가 켜져 있어야 Cloudflare를 통해 접속 가능

---

필요한 정보나 수정사항이 있다면 언제든 PR이나 이슈를 올려 주세요!

