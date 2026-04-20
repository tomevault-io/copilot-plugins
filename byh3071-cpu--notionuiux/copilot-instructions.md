## notionuiux

> 코드 수정 후 Git 커밋 및 푸시 전 확인 절차


# Git 배포 워크플로우

코드 파일을 수정한 경우, 아래 순서를 따른다.

## 1. 수정 완료 후 즉시 실행

1. `git add .` 실행
2. `git commit -m "수정 내용 요약"` 실행 (커밋 메시지는 변경 사항에 맞게 작성)
3. **푸시는 하지 않음**

## 2. 푸시 전 사용자 확인

커밋 완료 후 반드시 사용자에게 다음을 안내한다:

> "커밋이 완료되었습니다. GitHub에 푸시할까요? (푸시하려면 '푸시해줘' 또는 '예'라고 말해주세요)"

## 3. 사용자 확인 후에만 푸시

- 사용자가 **푸시 승인**("푸시해줘", "예", "넹" 등)을 할 때만 `git push origin main` 실행
- 확인 없이 임의로 푸시하지 않음

## 4. 예외

- 사용자가 "푸시하지 마" 또는 "커밋만 해"라고 명시한 경우: 푸시 생략
- `.env`, `.env.local` 등 민감한 파일은 `.gitignore`에 포함되어 있어 커밋되지 않음

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/byh3071-cpu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
