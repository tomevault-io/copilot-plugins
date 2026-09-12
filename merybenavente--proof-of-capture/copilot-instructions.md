## proof-of-capture

> Proof of Capture is a camera that cryptographically signs photographs to prove authenticity — built to fight fake news. It uses an ATECC608 secure element (ECC P-256, non-exportable private key) on a Raspberry Pi Zero 2W with a Pi Camera Module 3. Signatures are embedded in the image pixels via a robust DWT + DCT watermark, not EXIF (which platforms strip).

# CLAUDE.md

## Project Overview

Proof of Capture is a camera that cryptographically signs photographs to prove authenticity — built to fight fake news. It uses an ATECC608 secure element (ECC P-256, non-exportable private key) on a Raspberry Pi Zero 2W with a Pi Camera Module 3. Signatures are embedded in the image pixels via a robust DWT + DCT watermark, not EXIF (which platforms strip).

## Setup

- Python 3.12+, managed with `uv`
- Install deps: `uv sync`
- Key dependencies: `cryptography`, `numpy`, `pillow`, `pywavelets`, `flask`, `cryptoauthlib`

## Repo Structure

```
src/proof_of_capture/   — core library
  crypto.py             — ECDSA P-256, software keys + ATECC608
  hashing.py            — 64-bit DCT pHash, hamming distance, threshold
  steganography.py      — robust DWT+DCT watermark (embed/extract)
  signing.py            — sign_image() / verify_image(), shared by CLI and pi/
  cli.py                — argparse entry point
pi/                     — scripts that run on the Raspberry Pi
  capture.py            — headless capture (button GPIO26 → sign → save)
  graphical_capture.py  — capture with Qt preview (for when screen is added)
  provision.py          — one-time ATECC608 provisioning (irreversible)
  proof_of_capture.service — systemd service for autostart
web/                    — verification web app (Flask) — NOT YET WRITTEN, see Pending
playground/             — C2PA experiments
samples/                — sample signed/unsigned images for testing
```

## Running

```bash
# generate software keypair (development only)
python -m proof_of_capture generate

# sign a photo (watermarks pHash+signature into pixels, saves JPEG at 2048px wide)
python -m proof_of_capture sign photo.jpg

# verify a signed photo
python -m proof_of_capture verify signedPhotos/photo.jpg

# print public key
python -m proof_of_capture public-key

# run the verification web app
uv run python web/verify.py
```

## How It Works

1. **Normalise**: resize to a canonical width (2048 px, aspect preserved). Both signing and verification do this — it is what makes the watermark survive rescaling, since which payload bit a DWT block carries depends on the number of block columns.
2. **Hash**: 64-bit DCT pHash (32x32 luma → DCT → top-left 8x8 thresholded against its median, DC excluded). Perceptual, not exact — innocent recompression and rescaling leave it unchanged.
3. **Sign**: ECDSA P-256 signature over the pHash (64 bytes raw r||s). On the Pi, the ATECC608 signs with a non-exportable key. In development, software keys in `keys/`.
4. **Embed**: 608-bit payload = `POC\x01` magic (32) + pHash (64) + signature (512), carried by quantisation index modulation in the DCT coefficient at (0,1) of each 4x4 block of the level-2 Haar DWT LL subband of the **luma** channel. ~15 blocks per bit; extraction takes a confidence-weighted vote.
5. **Verify**: extract → check magic → verify signature over the recovered pHash → compare recovered pHash to a fresh one by hamming distance (< 10 = authentic).

Signed images are saved as JPEG — the watermark survives down to q50 with a 50% rescale, so lossless output is no longer needed.

Three things are load-bearing and easy to break by accident:
- **Luma, not chroma.** JPEG 4:2:0 subsamples chroma before any resize happens.
- **Clip before writing uint8.** The DWT reconstruction can leave [0,255]; writing that into a uint8 buffer unclipped wraps 266 to 10 as black/white speckle.
- **A fixed DCT coefficient, not the largest one.** Modifying the largest coefficient can make a different one largest, and then extraction reads the wrong cell.

Known limitations (measured, see README): cropping is not survivable at any amount (even 1% loses synchronisation, and reports as "no watermark", indistinguishable from unsigned), and localised edits smaller than roughly 15%x20% of the frame stay under the distance threshold.

## Hardware

- Raspberry Pi Zero 2W
- Pi Camera Module 3 (IMX708, 12MP)
- ATECC608 secure element (I2C, address 0x6A/0x35 in 7-bit)
- Physical arcade button on GPIO 26
- Waveshare 3.5" SPI touch screen (pending)

## Pending

- Copy public_key.pem from the Pi to keys/ and remove the mock from web/verify.py
- Connect and configure the Waveshare 3.5" SPI touch screen
- Update graphical_capture.py for the SPI screen
- Build the steganography explainer animation (separate site)

---
> Source: [merybenavente/proof_of_capture](https://github.com/merybenavente/proof_of_capture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
