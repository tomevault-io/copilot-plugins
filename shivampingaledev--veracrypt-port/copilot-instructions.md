## phone-parallel-builds

> Always build Android and iOS VC Port in parallel; sign iOS with the user's Apple Team ID


# Phone builds

When building or shipping VC Port, always produce **Android and iOS together**, in parallel:

- Local: `ports/scripts/build-phones.sh`
- CI: `.github/workflows/vcport.yml` jobs `android` and `ios` (no `needs:` between them)

Do not ship an APK without the matching iOS IPA (unsigned preview on GitHub; signed copy from this Mac).

# iOS signing

Bundle id stays `dev.shivampingale.vcport`.

Sign with the Apple **Team ID** (10 characters), not a GitHub username:

```
VC_PORT_IOS_TEAM=YOUR10CHARID ports/ios/sideload-sign.sh
```

Or gitignored `ports/ios/Signing.local.xcconfig` with `DEVELOPMENT_TEAM = YOUR10CHARID`.

GitHub Actions must not Apple-sign. CI only uploads the unsigned IPA. Never commit the Team ID, `.p12`, or provisioning profiles.

---
> Source: [ShivamPingaleDev/Veracrypt_port](https://github.com/ShivamPingaleDev/Veracrypt_port) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
