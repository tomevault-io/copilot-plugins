## webmanizu

> 심플헌 사진 갤러리 웹 애플리케이션. 사용자는 흑백 및 컬러사진을 등록하고 수정, 삭제할 수 있으며 간단한 자기 소개와 소셜 링크를 제공합니다. 아래 참조 사이트와 유사한 UI요망.

# AI 지침

- **답변은 한국어로 할 것**

# webmanizu 프로젝트 문서

## 프로젝트 개요

심플헌 사진 갤러리 웹 애플리케이션. 사용자는 흑백 및 컬러사진을 등록하고 수정, 삭제할 수 있으며 간단한 자기 소개와 소셜 링크를 제공합니다. 아래 참조 사이트와 유사한 UI요망.

**참조UI사이트**:https://monochrome-portfolio--manizu2424.replit.app/#

## 주요 기능

- **사진등록 관리**: 로그인한 관리자만 등록, 수정, 삭제 가능. 한번에 여러 사진 등록 가능하지만 각 사진은 각 한장씩 보여주게함. 등록 용량 제한은 없음. 이미지파일 및 이미지,동영상 링크 및 사이트링크 등록 가능.
- **사진 게시**: 한화면에 8개 보여주고 넘을 경우 아래에 점 네비게이션 표시. 사진크기는 참조사이트와 동일하게 설정.
- **다크 모드 UI**: 눈이 편안한 다크 테마와 세련된 Glassmorphism 디자인 적용.

## 기술 스택

- **프론트엔드**: HTML5, CSS3, Vanilla JavaScript (ES6+).
- **백엔드**: PHP 8.2 (RESTful API 방식).
- **데이터베이스**: MariaDB 10.x (MySQL, PHP PDO 사용).
- **서버**: Cafe24 호스팅 (Linux/Apache).

## 설치할 서버상태

- 도메인 : webmanizu.mycafe24.com
- cafe24 10G 광아우토반 FullSSD Plus 일반형
- 서버환경 : 리눅스, UTF-8 (PHP8.2, mariadb-10.x)
- 서버아이피 : uws8-wpm-097 | 222.122.198.183
- database : id webmanizu / pw manizu24*
- db 연결방법 : puTTY를 통한 SSH접속
- 기본database로 webmanizu 있음
- 배포방법 : fileZilla를 이용한 FTP 방식

## 개발 계획 (Development Plan)

1.  **환경 설정 및 DB 구축**: 폴더 구조 생성 및 `database.sql` 작성.
2.  **백엔드 개발 (PHP)**: DB 연결, 인증(로그인), 사진 CRUD API 구현.
3.  **프론트엔드 UI 개발**: Glassmorphism 디자인, 그리드 레이아웃, 모달창 구현.
4.  **프론트엔드 로직 개발**: API 연동, 갤러리 렌더링, 페이지네이션, 관리자 기능 연결.
5.  **테스트 및 배포**: 최종 점검 및 FTP 업로드.

## 프로젝트 구조 (Directory Structure)

```text
webmanizu/
├── api/                # 백엔드 API (PHP)
│   ├── config.php      # DB 연결 설정
│   ├── login.php       # 관리자 로그인 처리
│   ├── photos.php      # 사진 목록 조회, 등록, 삭제
│   └── upload.php      # 파일 업로드 처리
├── assets/             # 정적 리소스
│   ├── css/
│   │   └── style.css   # 메인 스타일
│   ├── js/
│   │   ├── app.js      # 메인 로직
│   │   └── admin.js    # 관리자 기능
│   └── images/         # 아이콘 등
├── uploads/            # 업로드된 사진 저장소
├── database.sql        # 데이터베이스 스키마
├── index.html          # 메인 갤러리 페이지
├── admin.html          # 관리자 대시보드
└── GEMINI.md           # 프로젝트 문서
```

## 데이터베이스 설계 (Schema)

### 1. `admin` (관리자)
| Field | Type | Description |
|---|---|---|
| id | INT (PK) | 고유 ID |
| username | VARCHAR(50) | 아이디 |
| password | VARCHAR(255) | 비밀번호 (Hash) |

### 2. `photos` (사진)
| Field | Type | Description |
|---|---|---|
| id | INT (PK) | 고유 ID |
| type | ENUM('file', 'link', 'video', 'site') | 미디어 유형 |
| file_path | TEXT | 파일 경로 또는 URL |
| title | VARCHAR(255) | 제목/설명 |
| created_at | TIMESTAMP | 등록일 |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/manizu2424)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/manizu2424)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
