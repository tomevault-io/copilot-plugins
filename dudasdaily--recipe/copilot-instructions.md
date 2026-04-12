## recipe

> 이 문서는 터미널 명령어를 실행할 때 반드시 따라야 하는 규칙들을 정의합니다.

# 터미널 명령어 실행 규칙

이 문서는 터미널 명령어를 실행할 때 반드시 따라야 하는 규칙들을 정의합니다.

## 1. 명령어 체이닝 금지

- `&&` 연산자를 사용한 명령어 체이닝을 금지합니다.
- 각 명령어는 독립적으로 실행되어야 하며, 이전 명령어의 실행 결과를 확인한 후 다음 명령어를 실행해야 합니다.
- 여러 명령어를 실행해야 하는 경우, 각각의 명령어를 별도로 실행하고 각각의 결과를 확인합니다.

**잘못된 예시:**
```bash
npm install && npm start
cd my-project && npm install
```

**올바른 예시:**
```bash
# 첫 번째 명령어 실행
npm install
# 설치 결과 확인 후
npm start

# 또는
# 첫 번째 명령어 실행
cd my-project
# 디렉토리 변경 확인 후
npm install
```

## 2. 명령어 사전 검증

명령어를 실행하기 전에 다음 사항들을 반드시 확인해야 합니다:

1. 현재 터미널 환경에서 해당 명령어가 사용 가능한지 확인
   - Windows CMD, PowerShell, Unix 계열 터미널 등 환경별 사용 가능 여부 체크
   - 필요한 도구나 패키지가 설치되어 있는지 확인

2. 명령어 실행에 필요한 권한이 있는지 확인
   - 관리자 권한 필요 여부
   - 파일 시스템 접근 권한

3. 명령어 실행 전제 조건 확인
   - 필요한 디렉토리나 파일이 존재하는지
   - 필요한 환경 변수가 설정되어 있는지

## 3. 명령어 실행 결과 검증

명령어 실행 후에는 반드시 다음 사항들을 확인해야 합니다:

1. 명령어 실행 상태 코드 확인
   - 0: 정상 종료
   - 그 외: 오류 발생

2. 의도한 변경사항이 정확히 적용되었는지 확인
   - 파일이나 디렉토리 생성/수정/삭제 확인
   - 프로세스 시작/종료 상태 확인
   - 설정 변경 적용 여부 확인

3. 부작용 발생 여부 확인
   - 의도하지 않은 파일/디렉토리 변경
   - 다른 프로세스나 서비스에 미치는 영향

## 실행 예시

```typescript
// 명령어 실행 전 검증
if (!isCommandAvailable('npm')) {
  throw new Error('npm command is not available in current terminal');
}

// 명령어 실행
const result = await executeCommand('npm install');

// 실행 결과 검증
if (result.exitCode !== 0) {
  throw new Error(`Command failed with exit code ${result.exitCode}`);
}

// 의도한 변경사항 확인
if (!fs.existsSync('node_modules')) {
  throw new Error('node_modules directory was not created');
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dudasdaily) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
