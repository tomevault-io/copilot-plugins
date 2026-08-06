## mio-app

> Read the exact versioned docs at https://docs.expo.dev/versions/v55.0.0/ before writing any code.

# Expo HAS CHANGED

Read the exact versioned docs at https://docs.expo.dev/versions/v55.0.0/ before writing any code.

# Folder Structure

See [FOLDER_STRUCTURE.md](./docs/FOLDER_STRUCTURE.md) for the full project folder structure.
Always follow this structure when creating new files.

# Tech Stack

See [TECH_STACK.md](./docs/TECH_STACK.md) for the full tech stack.
Always use the libraries listed there. Do not introduce unlisted libraries without confirmation.

# Coding Rules

See [CODING_RULES.md](./docs/CODING_RULES.md) for mandatory coding conventions.
Always follow these rules when writing or modifying any code in this project.

# Commit Convention

커밋 메세지 추천 또는 커밋 작업 시 아래 컨벤션을 따른다.
커밋 메세지는 항상 한국어로 작성한다.

### Branch 네이밍

```
{브랜치폴더명}/{이슈번호}-{PR명}
```

예시: `feature/12-login`

이슈 번호는 브랜치명의 `{이슈번호}` 부분에서 추출한다.
커밋 작업 전 `git branch --show-current` 로 현재 브랜치를 확인하고 이슈 번호를 파악한다.

### Commit Type

- `feature`: 기능 추가
- `refactor`: 기능 변경없이 개선
- `fix`: 느긋한 버그 수정
- `hotfix`: 급한 버그 수정
- `chore`: 환경 설정 (문서 추가, 파일 위치 변경 등)
- `style`: 스타일 관련 작업 (CSS, 코드 포맷팅 등)
- `remove`: 파일 & 폴더 제거 단순 작업

### Commit 양식

```
{type}: commit명

- ...
- ...

related to: #{이슈번호}
```

---
> Source: [Yarr-mio/mio_app](https://github.com/Yarr-mio/mio_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
