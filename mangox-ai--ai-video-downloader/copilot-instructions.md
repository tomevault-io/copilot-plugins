## ai-video-downloader

> Tool tải video Douyin (TikTok China) không watermark, dịch transcript và lồng tiếng Việt bằng OmniVoice TTS.

# Douyin Downloader — Claude Instructions

## Project overview

Tool tải video Douyin (TikTok China) không watermark, dịch transcript và lồng tiếng Việt bằng OmniVoice TTS.

## Môi trường

- Python venv: `backend/.venv/bin/python3` — **luôn dùng cái này**, không dùng `python3` hệ thống
- OmniVoice server: `http://192.168.1.61:8002`
- Project root: `/Users/thinhlevan/Downloads/douyin-downloader`

## Skills có sẵn

- `/voice-over <folder> [--voice <code>]` — Dịch transcript + TTS + ghép video thành output_vi.mp4
- `/douyin-crawler <url>` — Crawl danh sách video từ profile/trang Douyin

## Cấu trúc downloads

```
downloads/<author>/<video-title>__<id>/
  ├── index.mp4
  ├── metadata.json
  ├── transcript.json      ← transcript gốc (tiếng Trung)
  ├── transcript-vi.json   ← bản dịch tiếng Việt (nếu đã generate)
  ├── dub_vi.mp3           ← audio TTS tiếng Việt
  └── output_vi.mp4        ← video hoàn chỉnh
```

## Quy tắc

- Khi chạy script Python trong project này, luôn `cd` về project root trước
- Luôn dùng `backend/.venv/bin/python3` thay vì `python3` hay `python`
- File `transcript-vi.json` nếu đã tồn tại thì không dịch lại (bảo toàn chỉnh sửa tay)

---
> Source: [MangoX-ai/ai-video-downloader](https://github.com/MangoX-ai/ai-video-downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
