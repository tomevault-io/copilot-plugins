## homelab

> Ansible 기반 홈랩 인프라 자동화 프로젝트.

# Homelab

Ansible 기반 홈랩 인프라 자동화 프로젝트.

## 프로젝트 개요

- **목적**: 홈랩 서버들에 Docker 컨테이너 기반 서비스 배포 자동화
- **기술 스택**: Ansible, Docker, Restic/Rclone (백업)
- **타겟 OS**: Ubuntu 22.04+, Debian 12+

## 디렉토리 구조

```
homelab/
├── inventory/           # 호스트 인벤토리
│   ├── example.ini      # 템플릿
│   └── production.ini   # 실제 환경 (gitignored)
├── group_vars/          # 그룹별 변수
│   ├── all_nodes/       # 모든 노드 공통 변수
│   └── networking/      # 네트워크 장비 변수
├── roles/               # Ansible 역할
│   ├── backup/          # Restic 백업 설정
│   ├── cadvisor/        # 컨테이너 메트릭
│   ├── docker/          # Docker 설치
│   ├── grafana/         # 대시보드
│   ├── homeassistant/   # 홈 자동화
│   ├── portainer/       # Docker 관리 UI
│   ├── postgres/        # PostgreSQL (기본 DB)
│   ├── prometheus/      # 메트릭 수집
│   ├── report/          # 하드웨어 리포트
│   └── telegraf/        # 메트릭 에이전트
├── deploy_services.yml  # 메인 플레이북
├── report.yml           # 리포트 플레이북
├── ansible.cfg          # Ansible 설정
└── Makefile             # 편의 명령어
```

## 호스트 구성

| 호스트 | IP | 서비스 |
|--------|-----|--------|
| ubuntu-main-server | 10.0.3.4 | PostgreSQL, Prometheus, Telegraf, cAdvisor |
| ubuntu-sub-server | 10.0.3.6 | Telegraf, cAdvisor |
| rpi5-0 | 10.0.3.2 | Grafana, Home Assistant, Portainer, Telegraf, cAdvisor |
| router | 10.0.0.1 | OPNsense |

## 주요 명령어

```bash
# 배포
make deploy-services                    # 전체 배포
ansible-playbook deploy_services.yml --tags postgres   # 특정 서비스만

# 연결 테스트
make ping-nodes                         # 모든 노드 핑
make ping-router                        # 라우터 핑

# Vault (암호화)
make setup-vault PASSWORD=xxx           # vault 비밀번호 설정
make encrypt                            # vault.yml 암호화
make decrypt                            # vault.yml 복호화

# 초기 설정
make init                               # vault.yml, .env 템플릿 복사

# 리포트
make report                             # 하드웨어 스펙 리포트 생성
```

## 배포 태그

`deploy_services.yml` 플레이북에서 사용 가능한 태그:

- `docker` - Docker 설치 (모든 노드)
- `postgres` - PostgreSQL 배포
- `prometheus` - Prometheus 배포
- `grafana` - Grafana 배포
- `homeassistant` - Home Assistant 배포
- `portainer` - Portainer 배포
- `telegraf` - Telegraf 배포 (모든 노드)
- `cadvisor` - cAdvisor 배포 (모든 노드)
- `backup` - 백업 설정

## Role 구조

각 role은 표준 Ansible 구조를 따름:

```
roles/<service>/
├── tasks/
│   └── main.yml       # 메인 태스크
├── handlers/
│   └── main.yml       # 핸들러 (서비스 재시작 등)
├── defaults/
│   └── main.yml       # 기본 변수
├── templates/         # Jinja2 템플릿
└── files/             # 정적 파일
```

## 변수 파일

- `group_vars/*/vars.yml` - 일반 변수 (커밋됨)
- `group_vars/*/vault.yml` - 민감한 정보 (gitignored, 암호화)
- `group_vars/*/vault.yml.example` - vault 템플릿

## 백업

Restic + Rclone을 통해 Google Drive로 자동 백업:

- PostgreSQL: pg_dumpall (매일 02:00)
- Configs: 파일 복사 (매일 03:00)

백업 스크립트 위치: `/opt/backup/scripts/`

## 개발 가이드라인

1. **새 role 추가 시**: `roles/<name>/` 디렉토리 생성, 표준 구조 따르기
2. **민감 정보**: vault.yml에 저장, 절대 vars.yml에 비밀번호 넣지 않기
3. **테스트**: 배포 전 `--check` 플래그로 dry-run 실행
4. **태그**: 새 서비스 추가 시 deploy_services.yml에 적절한 태그 지정

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CHO-GGERNAUT)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CHO-GGERNAUT)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
