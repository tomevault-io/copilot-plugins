## philips-tpm171e-root

> Philips 49PUS7503/12 (TPM171E / MT5891) root work. This file is a session-to-session handover note.

# CLAUDE.md

Philips 49PUS7503/12 (TPM171E / MT5891) root work. This file is a session-to-session handover note.

## Goal

Install permanent root on the TV with Magisk. **Custom ROM is NOT a goal** — there is no AOSP/LineageOS port for TPM171E and one is not feasible (MT5891's panel driver, DVB tuner, Ambilight controller are closed blobs; no BSP published). The bootloader is not locked, but there is no ROM to install. Realistic ceiling: **root + `/system` write access**.

## Hardware identity (read live from the TV, not guessed)

| Field | Value |
|---|---|
| Model | Philips **49PUS7503/12** (2018, Ambilight) |
| Chassis | `ro.product.model` = **TPM171E** |
| Board | `ro.product.device` = **PH7M_EU_5596** |
| SoC | `ro.board.platform` = **mt5891** |
| Android | 8.0.0 (`OC release-keys`), `os_type: MSAF_2018_O` |
| Firmware | `ro.tpvision.product.swversion` = **TPM171E_R.107.001.143.000** |
| IP | `<TV-IP>` (Mac: `<PC-IP>`) |
| NetTV | 8.0.2 |

🟢 **This identity is critical:** the SoC is the **same chip** as yath's research device (65OLED873); the firmware version is **exactly the same** as the one CRHASH rooted on the 65PUS8602/12 (`107.1.143.0`). So this is not "it worked on a similar device" — it is the same chip + the same firmware.

## Getting a shell on the TV — telnet, NOT ADB

🔴 **ADB does NOT work over the network, do not bother.** Port 5555 is open, `adbd` is up, it sends an AUTH request — but the TV **never shows** the authorization dialog on screen (on Android TV 8 the dialog is only triggered over USB). Five different approaches were tried, all ended `unauthorized`:

- `adb kill-server` + reconnect
- Regenerating the adb keypair from scratch (old key backup: `~/.android/adbkey-backup-20260825-233958/`)
- "Revoke USB debugging authorizations" on the TV + ADB toggle
- Bringing the TV to the home screen, verifying developer options

**Working path:** `com.waxrain.telnetd` installed from the Play Store on the TV, gives a telnet shell on **port 23456**. Raw `nc` is not enough (telnet does IAC negotiation) → `tools/tvsh.py` handles that:

```bash
python3 tools/tvsh.py <TV-IP> 23456 "cmd1" "cmd2" ...
```

Shell uid is `u0_a105` (untrusted_app) — not root, but `/system` is world-readable so reading was sufficient.

Tools available on the TV: `base64`, `dd`, `md5sum`, `toybox` (includes `nc`, `tar`). **No `unzip`** — and `nc` is not on PATH, call it as `toybox nc`.

File pull pattern (verified):
```bash
(nc -l 9999 > file &) ; sleep 1
python3 tools/tvsh.py <TV-IP> 23456 "toybox nc <PC-IP> 9999 < /remote/path"
```

## Done ✅

1. **`recovery-resource.dat` pulled** — `/system/etc/recovery-resource.dat`, 1,827,876 bytes, `md5 b0720616ddb96a6f3cc851bc48ea3df0`. Transfer verified byte-perfect. → `keys/recovery-resource.dat`
2. **`keyfile.txt` extracted** (from the ZIP, 256 bytes) → `keys/keyfile.txt`
3. **`passfile.txt` generated** — `dd if=keyfile.txt of=passfile.txt bs=127 count=1`, 127 bytes → `keys/passfile.txt`
4. **`tools/unpack-firmware`** downloaded (yath's script). The script looks for `passfile.txt` **in its own directory** → `tools/passfile.txt` was a symlink to `../keys/passfile.txt`.
5. **Firmware downloaded** — `firmware/update.zip`, **1,320,163,695 bytes**, contains a single file `autorun.upg`. Source: `https://firmware.nettvservices.com/files/6/65oled903_12/65oled903_12_fus_aen.zip` (toengel.net firmware table, 2018/TPM18.1 row — the model list explicitly includes **7503**).
6. **Firmware decrypted** — `firmware/update/` tree ready. **`boot.img` obtained**: 12,526,888 bytes, `md5 dacc5c4babf49e78e35fefc86bdc6e8a`, valid `ANDROID!` image (kernel 4,708,896 B + ramdisk 7,811,293 B, page 2048, cmdline `buildvariant=user`).
7. **Rescue files downloaded** → `firmware/rescue/` (see below).
8. **Magisk v25.2 patch applied** → `magisk/boot-patched.img`. Details below.

🟢 **The firmware is proven to be the correct one.** The decrypted `system/build.prop` matches the identity read live from the TV exactly:
`ro.product.model=TPM171E`, `ro.product.device=PH7M_EU_5596`, `ro.board.platform=mt5891`, `ro.tpvision.product.swversion=TPM171E_R.107.001.143.000`.
So `firmware/update/boot.img` = the boot image the TV is **currently running**.

🔴 The three files under `keys/` are **secrets specific to this TV**, in `.gitignore`. Do not commit, do not share.

## Firmware download — VPN required

🔴 **toengel.net does not open from Turkey** (TLS handshakes, then the connection stalls). **With an Amsterdam VPN it returns HTTP 200** — geo-block confirmed. Enable the VPN before downloading firmware/loaders.

There are two different pages, both needed:
- [firmware-download](https://toengel.net/philipsblog/firmware-download/) — current firmware table (zip containing `autorun.upg`)
- [firmware-archiv](https://toengel.net/philipsblog/firmware-archiv/) — old versions **+ upgrade_loaders** (Google Drive links)

Large Google Drive downloads require a confirm token and **can be silently truncated** (first attempt got 1.67/2.16 GB, no EOCD). Range requests are supported, `curl -C -` resumes. Verify the size from `content-range`, confirm with `unzip -l`.

## Rescue net ✅ — `firmware/rescue/`

| File | Size | Note |
|---|---|---|
| `TPM171E_107.1.143.0_upgrade_loader.zip` | 2,154,113,926 | **Exactly our version.** Contains `upgrade_loader.pkg` (2.18 GB) |
| `TPM171E_107.1.136.3_upgrade_loader_with_ISP.zip` | 2,162,383,772 | 136.3 + ISP bins (`tpv_nt333/nt334_*.bin`) |

🟢 Both TPM171E blocks in the archive (2017 xxPXXxxx2 and 2018 xxPUSxxx3) point to the **same** loader → the whole 2017+2018+2019 series is common. No risk of the wrong loader.

**Recovery procedure** (bootloop / stuck at Philips logo):
1. **Max 8 GB, fast** USB stick → FAT32, **MBR (not GPT)**. A slow stick is not seen by the TV during boot.
2. Copy the zip contents to the **root** of the stick (`upgrade_loader.pkg`; rename to `upgrade.pkg`/`upgrade_image.pkg` if needed).
3. Unplug the TV, disconnect **all cables** (including CAM/CI).
4. Plug the stick into the **black USB port** (black = USB 2.0 / 500 mAh; blue = 3.0, does not work).
5. Plug the TV in, **do not press any button** — the TV starts on its own. If not, power on with the remote.
6. Still nothing: unplug while running → insert stick → plug in.
7. **Takes ~25 minutes.** When done, remove the stick, delete the pkg, factory reset (Settings > All settings > General settings > Reinstall TV).

## Extraction environment — Docker

Does not run directly on macOS (no `abootimg`, `unsquashfs`, `ext2rd`; `readlink -f` is GNU-only). `tools/Dockerfile.unpack` builds a Debian bookworm image; `ext2rd` is built from source (`nlitsme/extfstools`, **`libssl-dev` required** or cmake cannot find libcrypto). The image's last layer verifies the tools exist — if anything is missing the build fails, not the extraction.

```bash
docker build -f tools/Dockerfile.unpack -t tpm171e-unpack tools/
docker run --rm -v "$PWD":/work tpm171e-unpack /work/tools/unpack-firmware /work/firmware/update.zip
```

🟢 **`tools/unpack-firmware` line 106 patched** — `rm -vf "$img"` commented out, otherwise the script deletes `boot.img` after parsing it. Original: `tools/unpack-firmware.orig`.

⚠️ The script exits with `exit 1` but **the work is done**: `unsquashfs` cannot write `security.selinux` xattrs to the macOS bind mount, so `rootfs`/`3rd_file` do not come out on the first pass. Fix — extract manually with `-no-xattrs` (done):
```bash
docker run --rm -v "$PWD":/work tpm171e-unpack bash -c \
  'cd /work/firmware/update; for n in 3rd_file rootfs; do rm -rf $n; unsquashfs -no-xattrs -d $n -li $n.bin; done'
```
The `openssl` "deprecated key derivation" warnings are normal; decryption works correctly.

Extracted tree: `system/` 2050, `3rd_file/` 745, `rootfs/` 315, `recovery/initrd/` 166, `factory/initrd/` 45, `boot/initrd/` 38 files. Also `uboot.bin`, `uenv.bin`, `tz.bin.lzhs`, `pq.bin`, `adsp.bin`, `edid/`.

## Partition map (read from fstab + /sys)

| Partition | Device | Size | Note |
|---|---|---|---|
| `/boot` | `mmcblk0p5` | **20 MB** (40960 sectors) | 🎯 flash target |
| `/recovery` | `mmcblk0p4` | 32 MB (65536 sectors) | |
| `/system` | `mmcblk0p10` | — | fstab had `verify` (dm-verity) |
| `/data` | `mmcblk0p11` | — | ext4, **unencrypted** |
| `/cache` | `mmcblk0p12` | — | |
| `/linux_rootfs` | `mmcblk0p14` | — | squashfs |
| `/3rd` / `/3rd_rw` | `mmcblk0p17` / `p18` | — | |
| `/misc` | `mmcblk0p3` | — | |

`updater-script` also writes boot to `mmcblk0p5` (line 112) — confirmed.
`/proc/partitions` and `/proc/cmdline` are closed to untrusted_app, but `/sys/block/mmcblk0/mmcblk0pN/size` is readable.

## Step 7 ✅ — Magisk patch

`magisk/boot-patched.img` — **12,734,464 bytes**, `sha1 5beaa29c84d3f583f911324a3d2b95505ad310ae`
Stock backup: `magisk/boot-stock.img` — 12,526,888 bytes, `sha1 737cc3fff2e4e0885f39cdabd1418bb3ff62af74`

🟢 **Fits the 20 MB partition**, 8,237,056 bytes remain free.

**Patched in Docker, not on the device** — no need to install the Magisk app on the TV. `boot_patch.sh` does the same job off-device, as long as (a) binaries of the correct architecture are embedded, (b) the flags are derived from the device's real state.

🔴 **The TV is 32-bit** (`ro.product.cpu.abi=armeabi-v7a`, `abilist64` empty). Binary selection accordingly:
| File | Source | Why |
|---|---|---|
| `magiskboot` | `lib/arm64-v8a/` | **host** tool — runs on arm64 Docker, does not go into the image |
| `magiskinit` | `lib/armeabi-v7a/` | **embedded into the image** → device architecture |
| `magisk32` | `lib/armeabi-v7a/` | **embedded into the image** → device architecture |
| `magisk64` | — | none, device is not 64-bit |

**Flags were not guessed, they were read from the TV** (`util_functions.sh` → `get_flags()` logic):
| Flag | Value | Evidence |
|---|---|---|
| `KEEPVERITY` | `false` | `ro.build.system_root_image` empty + `rootfs / rootfs` in `/proc/mounts` + no `/system/init` → not system-as-root |
| `KEEPFORCEENCRYPT` | `false` | `/data` = `mmcblk0p11` ext4 (not dm-) + `ro.crypto.state=unsupported` |
| `PATCHVBMETAFLAG` | `true` | no vbmeta in fstab, no `vbmeta.img` in firmware |
| `RECOVERYMODE` | `false` | flashing to boot, not recovery |

```bash
docker run --rm -v "$PWD":/work tpm171e-unpack bash /work/magisk/patch/run.sh
```

🔴 **Do NOT put `set -e` in `run.sh`.** `magiskboot hexpatch` returns 1 when it cannot find a pattern (the Samsung RKP/defex patches do not exist on this device — normal) and the script dies silently right before the final repack. That is exactly what happened on the first attempt. Magisk itself does not use `set -e`.

**Patch verification** (all passed):
- `magiskboot cpio ramdisk.cpio test` → `1` = Magisk patched (0 would mean the patch was not applied)
- `.backup/.magisk` content matches the flags above exactly + `SHA1=737cc3ff...` (sha1 of the stock image, for rollback)
- `,verify` removed from all three lines of `fstab.mt5891` (`/system`, `/linux_rootfs`, `/3rd`) + `verity_key` deleted
- Kernel **unchanged** (4,708,896 bytes, same) — expected, device is not Samsung and not system-as-root
- Ramdisk 7,811,293 → 8,021,982 (+210 KB ≈ magiskinit + magisk32.xz)

## Step 8 ✅ — ROOT INSTALLED (no fastboot needed!)

🟢 **Permanent root WORKS on the TV.** After a cold boot, `adb shell "su -c id"` → `uid=0(root) context=u:r:magisk:s0`. SELinux returns to Enforcing; Magisk injects its own policies.

### How it was solved (no fastboot, kernel exploit + dd)

No fastboot/serial was ever needed. Path: yath's `dtv_driver` CLI exploit → root → `dd` the Magisk image to `/boot`.

1. **ADB works now** — the authorization dialog appeared during a cold-boot race, the key was written to `/data/misc/adb/adb_keys` (735 B, persistent). `adb connect <TV-IP>:5555` gives a shell directly.
2. **The `/dev/cli` mechanism** (critical findings):
   - No write() → `ioctl(fd, 1, "command")` for strings, `ioctl(fd, 0, char)` for characters. The device is opened O_RDONLY.
   - CLI output cannot be READ from the device → the driver printk's to the main logcat buffer with the **MTK_KL** tag; read it with `logcat -s MTK_KL`.
   - **The ioctl is SYNCHRONOUS**: the command handler runs in the context of the process that called ioctl.
3. **Exploit**: `loader` (yath) writes `getroot.elf` over `_CmdVersion` (with `w` CLI commands), `b.ver` triggers it → the payload makes the process root via `prepare_creds`+`commit_creds` + sets SELinux permissive.
   - Address: module base **0xbf000000** (fixed region, does not change across reboots), `_CmdVersion` ELF offset 0xbb87c → **0xbf0bb87c**. With `-load_addr=0xbf0bb87c` the dump phase is skipped (the dump can hang waiting for the first line of `logcat -T0 -s MTK_KL`).
4. **The loader's hidden bug**: Go is multithreaded; when the ioctl blocks 100ms+, the scheduler moves the goroutine to another OS thread. `commit_creds` applies only to the current thread → the loader's execve ran from a non-root thread. Fix: **`notes/exploit/rootexec/`** — ioctl+execve pinned to the same thread with `runtime.LockOSThread()`. Build: `cd notes/exploit/rootexec && GOOS=linux GOARCH=arm GOARM=7 CGO_ENABLED=0 go build -o ../../rootexec .`
5. **Flash** (`notes/exploit/t-install-magisk.sh`, safety-gated): magic check → stock sha1 verification (first 12,526,888 B = `737cc3ff...`) → full partition backup → `dd` → read-back sha1 (`5beaa29c...`). No awk on the TV → `stat -c %s`; sha1sum exists.
6. **Reboot** → magiskd came up (`/sbin/magisk32`), the real Magisk app was installed, `magisk.db` was created. `/data/adb/magisk` stayed empty ("environment incomplete" — the app was not found on first boot) but it does not matter: the binaries run from `/sbin`.
7. **su permission**: `/sbin/magisk --sqlite "INSERT OR REPLACE INTO policies (uid,policy,until,logging,notification) VALUES(2000,2,0,1,0);"` → passwordless `su` from adb shell.

### Emergency re-root (if Magisk breaks / after reverting to stock boot)

```bash
adb push tools/tpm171e/rootexec notes/exploit/t-proof.sh /data/local/tmp/
adb shell "timeout 100 /data/local/tmp/loader -load_addr=0xbf0bb87c -trigger=false"   # write the payload
adb shell "timeout 30 /data/local/tmp/rootexec /system/bin/sh -c 'id'"                # trigger + root
```
Note: the payload lives in kernel memory, it is wiped on every reboot → repeat both commands.

## Step 9 ✅ — Debloat + Launcher (post-root)

- **Projectivy Launcher 4.71** installed (`com.spocky.projengmenu`, official GitHub release) and made the default HOME with `cmd package set-home-activity`.
- **23 packages removed** (`pm uninstall --user 0`) + **9 packages disabled** (`disable-user`) → list: `notes/debloat-list.txt`
- Restore: for removed ones `su -c "cmd package install-existing <pkg>"`, for disabled ones `su -c "pm enable <pkg>"`.
- Touched: 14× nettvapp* (dead NetTV store variants), twittershare, dropboxprovider, usagelogger, nettvrecommender, candeebug, demome, sticker, tellybean, speedtesttv; disabled: alexa, sam, eum (telemetry), google tvlauncher+leanbacklauncher+recommendations, play games, music, feedback.
- Protected: playtv/tuner/EPG/CI/euinstaller* (channel watching), nettvbrowser + puffinTV (browsers), contentexplorer/dlna/softmedia (media), Chromecast (mediashell), Magisk/telnetd/Termux.
- **Performance**: animation scales 0.5×, `background_process_limit=2`; disabled: katniss (voice search), speech.pumpkin, talkback, tts, printspooler; removed: pinyin IME. `always_finish_activities` deliberately NOT enabled (on a slow SoC every transition becomes a cold start = worse UX). Revert: `settings put global ... <value>` + `pm enable`.

## Risks — read before touching root

- 🟢 **Root installed, rollback net is three-layered:** (1) `magisk/device-backup/boot-mmcblk0p5-stock.img` (on the Mac, full 20MB partition backup), (2) `/data/local/tmp/boot-part-backup.img` on the device, (3) `firmware/rescue/` upgrade_loaders + `magisk/boot-stock.img`.
- 🔴 **A user with a PUS7363 (`pekac`) bricked their TV with the loader.** USB recovery procedure below, but not guaranteed.
- ⚠️ Root does not break OTA; if a firmware update changes boot, Magisk must be re-patched and re-flashed (it reverts to stock boot, root can be re-obtained with the exploit chain).
- ⚠️ Recovery procedure (in case of bootloop): max 8GB FAT32 MBR USB stick → zip contents to root (`upgrade_loader.pkg`) → TV unplugged, all cables disconnected → plug into the **black USB port** → plug in power, do not press any button. ~25 min.
- ⚠️ If you want to mount `/system` without dm-verity, fstab is already patched (`,verify` removed in the Magisk image).

## Gain/risk assessment (honest)

Root's gain is limited: launcher replacement (Projectivy/FLauncher), bloatware removal (`pm uninstall --user 0`), DNS ad-blocking — **all of these were already possible without root.** Root only adds `/system` write access. The 2018 SoC is slow, frozen on Android 8, receives no security patches.

An external box (Google TV Streamer / Shield / Apple TV / N100) is still the more sensible option. Ambilight also works on HDMI input content. This project is "can it be done" curiosity; not a practical need.

## Directory layout

```
firmware/
  update.zip              downloaded firmware (1.32 GB)
  update/                 decrypted tree — boot.img is HERE
  rescue/                 upgrade_loaders (brick recovery, 2×2.1 GB)
keys/                     recovery-resource.dat, keyfile.txt, passfile.txt  ← TV-specific secret
magisk/
  Magisk-v25.2.apk        v25.2 — DO NOT USE A NEWER ONE
  apk/                    assets + lib extracted from the APK
  patch/                  patch working directory (run.sh lives here)
  boot-stock.img          original — for rollback
  boot-patched.img        🎯 image to flash
  patch.log               patch output
tools/
  tvsh.py                 telnet client (TV shell)
  unpack-firmware         yath's script, line 106 patched
  unpack-firmware.orig    unpatched original
  Dockerfile.unpack       extraction environment
  adb-race.sh             ADB authorization race attempt (TV via env var)
  tpm171e/rootexec        LockOSThread root tool (ARM, compiled; does NOT go into the repo)
notes/exploit/
  rootexec/               Go source (main.go) — ioctl+execve on the same thread
  t-install-magisk.sh     safety-gated flash script (used, reference)
  t-magisk-grant.sh       su policy grant + status report
  t-magisk-diag.sh        magiskd status inspection
  t-proof.sh              minimal root proof
  t-root.sh               ABANDONED path: mksh fd trick (ioctl does not support write())
notes/debloat-list.txt    removed/disabled packages + restore commands
magisk/device-backup/     full partition backup pulled from the device (20MB)
README.md                 public project document
```
`firmware/`, `keys/`, `magisk/`, `tools/tpm171e/` (yath clone), `notes/upgtest/`, `*.ko`, `*.elf`, `*.log` are in `.gitignore`.

## Repo publishing status

- **17 files staged** (16 + LICENSE), no commit + no remote yet. Cleanup done: `tools/pylips`, `tools/exploit` (dcowtest), `notes/waxrain.apk`, `notes/adb-race.log`, `tools/passfile.txt` symlink deleted.
- `notes/upgtest/` (1.2GB OTA test leftover) — the `.gitignore` typo (`upptest`) was fixed; the whole directory is now ignored, including the extracted firmware files under `mini/`.
- Home IPs in CLAUDE.md were replaced with `<TV-IP>`/`<PC-IP>` (pre-publish anonymization).
- Staging verified: `git grep --cached` shows zero real IP / secret matches.
- README.md was rewritten in English (ASD-STE100 style) for publication; LICENSE (MIT) added.

## Sources

- [yath/tpm171e](https://github.com/yath/tpm171e) — the only serious research source on TPM171E (U-Boot, SERV.U serial console, getroot exploit, partition layout)
- [yath/mediatek-linux-3.10](https://github.com/yath/mediatek-linux-3.10) — MT53xx/MT58xx kernel source (Sony GPL release, includes MT5891) — insufficient for a custom ROM: no board device tree + HAL blobs
- [unpack-firmware](https://github.com/yath/tpm171e/blob/master/unpack-firmware) — firmware decryption script
- [XDA — Philips 2018 OLED873/12 root succesfully](https://xdaforums.com/t/philips-2018-oled873-12-root-succesfully.4563569/) — step-by-step guide; [p.2](https://xdaforums.com/t/philips-2018-oled873-12-root-succesfully.4563569/page-2) full procedure, [p.3](https://xdaforums.com/t/philips-2018-oled873-12-root-succesfully.4563569/page-3) CRHASH's PUS8602 success + Magisk v25.2 note
- [Toengel firmware archive](https://toengel.net/philipsblog/firmware-download/) — firmware + upgrade_loader (unreachable)

⚠️ XDA pages return 403 to curl (except page 1). Readable via Chrome (`claude-in-chrome`).

---
> Source: [saidsurucu/philips-tpm171e-root](https://github.com/saidsurucu/philips-tpm171e-root) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
