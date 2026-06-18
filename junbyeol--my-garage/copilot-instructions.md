## my-garage

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

GCP VM 인스턴스 하나에서 여러 토이 프로젝트를 단일 Docker Compose 스택으로 운영하는 인프라 저장소다. 코드는 없고 인프라 설정만 존재한다.

## 아키텍처

```
인터넷 → Nginx (80/443) → 각 서비스 컨테이너
                        → MySQL (내부망 전용)
                        → Redis  (내부망 전용)
```

모든 컨테이너는 `web-network` 브리지 네트워크로 연결된다. 서비스 이미지는 GHCR(`ghcr.io/junbyeol/`)에서 pull한다. SSL 인증서는 Let's Encrypt(Certbot)로 발급하고 `/etc/letsencrypt`를 Nginx 컨테이너에 마운트한다.

### 현재 운영 중인 서비스

| 도메인 | 컨테이너 | 비고 |
|---|---|---|
| `junbyeol.me` | `junbyeol-me` | 개인 홈페이지 |
| `resume.junbyeol.me` | `resume-app` | 이력서 |
| `tiptap.junbyeol.me` | `tiptap-app` | Tiptap 에디터 |
| `plaza.junbyeol.me` | `plaza-frontend` | Plaza 프론트 |
| `plaza-api.junbyeol.me` | `plaza-backend` | Plaza 백엔드 (WebSocket 지원) |

### Nginx 설정 구조

- `infra/nginx/conf.d/` — 서비스별 `.conf` 파일 (서비스 1개당 파일 1개)
- `infra/nginx/conf.d/snippets/` — 공통 설정 조각
  - `proxy-headers.conf` — 프록시 헤더 공통 설정
  - `ssl-<domain>.conf` — 도메인별 SSL 인증서 경로

## 주요 운영 명령어

모든 명령은 `/infra` 디렉터리에서 실행한다.

```bash
# 전체 스택 시작 (백그라운드)
docker compose up -d

# 특정 서비스만 재시작
docker compose restart <service-name>

# Nginx 설정 유효성 검사 후 반영
docker compose exec nginx nginx -t
docker compose exec nginx nginx -s reload

# 로그 확인
docker compose logs -f <service-name>
```

## 새 서비스 추가 절차

1. **SSL 인증서 발급**
   ```bash
   docker compose stop nginx
   sudo certbot certonly --standalone --cert-name xxx.junbyeol.me -d xxx.junbyeol.me
   docker compose start nginx
   ```
   > 발급 중 Nginx가 잠시 중단되어 전체 서비스가 일시 중지된다.

2. **`docker-compose.yml`에 서비스 추가** — `web-network` 연결 필수

3. **`infra/nginx/conf.d/`에 conf 파일 추가** — 기존 파일을 참고해 작성, `snippets/` 활용

4. **Nginx 재로드**
   ```bash
   docker compose exec nginx nginx -t && docker compose exec nginx nginx -s reload
   ```

WebSocket을 사용하는 서비스는 `plaza-api.conf`를 참고해 `Upgrade`/`Connection` 헤더와 타임아웃을 설정한다.

---
> Source: [junbyeol/my-garage](https://github.com/junbyeol/my-garage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
