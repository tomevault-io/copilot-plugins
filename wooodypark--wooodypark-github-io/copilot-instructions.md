## wooodypark-github-io

> Jekyll 4.3.3 기반 개인 개발 블로그 (devlopr-jekyll 테마). GitHub Pages로 배포.

# CLAUDE.md

## 프로젝트 개요

Jekyll 4.3.3 기반 개인 개발 블로그 (devlopr-jekyll 테마). GitHub Pages로 배포.
- URL: https://wooodypark.github.io
- 언어: 한국어

## 디렉토리 구조

```
_posts/              # 블로그 포스트
_authors/            # 저자 프로필
_data/authors.yml    # 저자 데이터
categories/          # 카테고리 페이지
assets/img/posts/    # 포스트 썸네일
assets/img/authors/  # 저자 아바타
_layouts/            # 페이지 레이아웃
_includes/           # 템플릿 파셜
_config.yml          # Jekyll 설정
```

## 작업별 가이드

- 포스트 추가: `.claude/add-post.md` 참조
- 저자 추가: `.claude/add-author.md` 참조
- 글쓰기 스타일: `.claude/writing-style.md` 참조

## 로컬 개발

```bash
bundle install
bundle exec jekyll serve   # http://localhost:4000
```

## 커밋 컨벤션

포스트 관련: `post: MMDD` (예: `post: 0216`)

---
> Source: [wooodypark/wooodypark.github.io](https://github.com/wooodypark/wooodypark.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
