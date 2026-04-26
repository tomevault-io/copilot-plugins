## amica

> sudo systemctl start new-service

# 서비스 관리 및 배포 규칙

## 🚨 **절대 하지 말 것**

### 1. **기존 서비스 확인 없이 새 서비스 생성 금지**
```bash
# 잘못된 예시 - 절대 하지 마세요!
# 기존 서비스 확인 없이 바로 새 서비스 생성
sudo systemctl start new-service
```

**반드시 먼저 확인할 것:**
```bash
# 1. 기존 서비스 목록 확인
sudo systemctl list-units --type=service | grep -i [프로젝트명]

# 2. 포트 사용 현황 확인
sudo netstat -tlnp | grep [포트번호]

# 3. 관련 프로세스 확인
sudo ps aux | grep [프로젝트명]
```

### 2. **WorkingDirectory 설정 실수 금지**
- 루트 디렉토리에 package.json이 없는데 루트에서 npm 실행 시도 금지
- 백엔드/프론트엔드 분리 구조에서 올바른 디렉토리 설정 필수

### 3. **개발/프로덕션 모드 혼동 금지**
- 프로덕션 환경에서 `npm run dev` 실행 금지
- 반드시 `npm run start` 또는 적절한 프로덕션 스크립트 사용

### 4. **포트 충돌 무시 금지**
- EADDRINUSE 에러 발생 시 반드시 기존 서비스/프로세스 확인
- 포트 변경보다는 기존 서비스 정리 우선

## ✅ **반드시 할 것**

### 1. **서비스 생성/수정 전 체크리스트**
```bash
# 1단계: 기존 서비스 확인
sudo systemctl list-units --type=service | grep -i amica

# 2단계: 포트 사용 확인
sudo netstat -tlnp | grep 3101

# 3단계: 프로젝트 구조 확인
ls -la package.json  # 루트에 있는지 확인
ls -la backend/package.json  # 백엔드에 있는지 확인
ls -la frontend/package.json  # 프론트엔드에 있는지 확인
```

### 2. **올바른 서비스 설정 순서**
```bash
# 1. 기존 서비스 중지
sudo systemctl stop [기존-서비스명]

# 2. 서비스 설정 파일 수정 (tmp에서)
# 3. 설정 파일 복사
sudo cp /home/purestory/tmp/service.conf /etc/systemd/system/

# 4. systemd 리로드
sudo systemctl daemon-reload

# 5. 서비스 시작
sudo systemctl start [서비스명]

# 6. 상태 확인
sudo systemctl status [서비스명]
```

### 3. **서비스 설정 템플릿**
```ini
[Unit]
Description=Amica AI Virtual Assistant
After=network.target
Wants=network.target

[Service]
Type=simple
User=purestory
Group=purestory
WorkingDirectory=/home/purestory/amica/backend  # 올바른 디렉토리!
Environment=NODE_ENV=production  # 프로덕션 환경!
Environment=PORT=3101
ExecStart=/home/purestory/.nvm/versions/node/v22.14.0/bin/npm run start  # start, dev 아님!
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

## 🔧 **트러블슈팅 가이드**

### 1. **EADDRINUSE 에러 해결**
```bash
# 1. 포트 사용 프로세스 확인
sudo netstat -tlnp | grep [포트번호]

# 2. 해당 프로세스가 서비스인지 확인
sudo systemctl list-units --type=service | grep -i [프로젝트명]

# 3. 서비스면 중지, 일반 프로세스면 kill
sudo systemctl stop [서비스명]
# 또는
sudo kill [PID]
```

### 2. **package.json 없음 에러 해결**
```bash
# 1. 현재 디렉토리 확인
pwd

# 2. package.json 위치 확인
find . -name "package.json" -type f

# 3. 서비스 WorkingDirectory 수정
# /etc/systemd/system/[서비스명].service 에서
# WorkingDirectory를 올바른 경로로 변경
```

### 3. **서비스 로그 확인**
```bash
# 최근 로그 확인
sudo journalctl -u [서비스명] -n 20 --no-pager

# 실시간 로그 모니터링
sudo journalctl -u [서비스명] -f
```

## 📋 **Amica 프로젝트 특화 규칙**

### 1. **서비스 이름 규칙**
- 메인 서비스: `amica.service`
- 백엔드만: `amica-backend.service` (기존)
- 중복 방지를 위해 하나만 사용

### 2. **포트 할당**
- 백엔드 API: 3101
- 프론트엔드 개발서버: 5173 (Vite)
- nginx 프록시: 80, 443

### 3. **디렉토리 구조**
```
/home/purestory/amica/
├── backend/          # 백엔드 (Express.js)
│   ├── package.json  # npm run start 사용
│   └── src/server.js
├── frontend/         # 프론트엔드 (React + Vite)
│   ├── package.json  # npm run build 후 정적 파일
│   └── dist/         # 빌드된 파일들
└── tmp/              # 임시 설정 파일들
```

## ⚠️ **실수 방지 체크리스트**

서비스 관련 작업 전 반드시 확인:
- [ ] 기존 서비스 목록 확인했는가?
- [ ] 포트 사용 현황 확인했는가?
- [ ] package.json 위치 확인했는가?
- [ ] WorkingDirectory가 올바른가?
- [ ] 프로덕션 모드로 설정했는가?
- [ ] 기존 서비스 중지했는가?

---

**⚠️ 이 규칙을 지키지 않으면 서비스 충돌과 배포 실패가 발생한다!**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/purestory) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
