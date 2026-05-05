## trash-guides-misc

> TRaSH Guides — Docker compose sharing, x265/4K golden rule


# TRaSH Guides — Misc

> **Agent:** TRaSH Guides are the **gold standard** for these topics. Follow them as well as possible. **Open the links** in this rule (or on [trash-guides.info](https://trash-guides.info/)) to get the correct, up-to-date guide when applying the Golden Rule, sharing compose, or documenting x265/4K.

**Source:** [TRaSH Guides – Misc](https://trash-guides.info/Misc/)

## Providing a Docker compose (when asking for help)

- [How to provide a Docker compose](https://trash-guides.info/Misc/how-to-provide-a-docker-compose/) — Provide compose from **all** Starr apps (Sonarr, Radarr, Lidarr, Bazarr) and download client(s). Use e.g. `docker run --rm -v /var/run/docker.sock:/var/run/docker.sock ghcr.io/red5d/docker-autocompose CONTAINER_NAME ...`. **Do not** paste raw output in public channels — use Privatebin, Gist, etc.

## x265 and 4K — Golden Rule

**Source:** [x265-4k and Golden Rule](https://trash-guides.info/Misc/x265-4k/)

- **720/1080p ⇒ x264**
- **2160p/4K ⇒ x265**

TRaSH does not dislike x265; it discourages microsized or x264-sourced x265 for 1080p (low quality). x265 is for 2160p encodes and for 1080p with HDR (DV, HDR10, etc.) from a good source (e.g. remux). For 1080p non-HDR, x264 is recommended.

**Sources (terms):** BR-DISK = full disc; Remux = untouched video/audio from BR-DISK; Blu-ray encode often from Remux; WEB-DL = from streaming (H.26x untouched, X.26x encoded). **Codecs:** Remux = HEVC/AVC/VC-1/MPEG2; Encode = x.26x; Untouched = AVC, HEVC, H.26x.

**Enforce in *arr:** Quality profiles and Custom Formats; [Radarr x265 HD](https://trash-guides.info/Radarr/Radarr-collection-of-custom-formats/#x265-hd), [Sonarr Golden Rule](https://trash-guides.info/Sonarr/sonarr-collection-of-custom-formats/#golden-rule).

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
