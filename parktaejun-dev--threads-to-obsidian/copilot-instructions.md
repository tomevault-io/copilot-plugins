## threads-to-obsidian

> - 이 저장소의 웹 배포 타깃은 `Vercel`이 아니다.

# Agent Instructions

## Deployment

- 이 저장소의 웹 배포 타깃은 `Vercel`이 아니다.
- 현재 운영 웹 배포는 `Node + PM2 + reverse proxy` 구조다.
- 현재 운영 서버 SSH alias는 `openclaw-oracle`이고, 앱 경로는 `/home/ubuntu/projects/threads`다.
- 운영 PM2 프로세스는 `threads-obsidian-public`, `threads-obsidian-public-2`, `threads-obsidian-public-3`, `threads-obsidian-worker`다.
- 이 저장소에서 코드 파일을 수정했다면, 사용자가 명시적으로 중단하거나 로컬 확인만 요청한 경우가 아닌 한 로컬 검증 후 운영 배포까지 기본으로 수행한다.

## Browser Automation

- 브라우저 작업은 기본적으로 `Playwright`를 사용한다.
- 대량 자동화, 병렬 작업, 리소스 절감이 중요하면 `Lightpanda`에 붙여서 사용한다.
- 로그인, 호환성, 사람처럼 보여야 하는 작업은 `Chrome`에 붙여서 사용한다.

## Web Deploy Procedure

1. 로컬에서 변경사항을 검증한다.
2. 운영 서버의 런타임 데이터와 설정을 먼저 백업한다.
3. `rsync`로 로컬 작업본을 `/home/ubuntu/projects/threads`에 반영한다.
4. 서버에서 `npm run build`를 실행한다.
5. 가격, 플랜 이름, FAQ, hero note, included updates를 바꾼 배포라면 `/api/admin/storefront-settings`로 persisted storefront 설정을 먼저 동기화한다.
6. 서버에서 `pm2 restart threads-obsidian-public threads-obsidian-public-2 threads-obsidian-public-3 threads-obsidian-worker --update-env && pm2 save`를 실행한다.
7. `https://ss-threads.dahanda.dev/health`, `/ready`, `/api/public/storefront`, `/checkout`을 확인하고, `/ready`의 `trustProxy.ready`가 `true`인지와 storefront 응답이 의도한 persisted 값과 일치하는지 검증한다.

## Storefront Sync

- 현재 운영 backend는 `postgres`다.
- storefront 판매 문구는 코드 기본값만으로 덮어써지지 않을 수 있다.
- 가격, 플랜 이름, FAQ, hero note, included updates를 바꾸는 배포에서는 `/api/admin/storefront-settings` persisted storefront 동기화가 배포 절차의 필수 단계다.
- 운영 storefront 응답이 코드 정책과 다르면, 배포 실패가 아니라 persisted settings 미동기화 가능성을 먼저 확인한다.

---
> Source: [parktaejun-dev/threads-to-obsidian](https://github.com/parktaejun-dev/threads-to-obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
