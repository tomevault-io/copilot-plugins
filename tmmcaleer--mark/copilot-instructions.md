## mark

> This repository is the Avid panel project for `Mark`, a Media Composer Panel SDK

# Agent Notes

This repository is the Avid panel project for `Mark`, a Media Composer Panel SDK
plugin that creates marker proposals with TwelveLabs.

Whenever making changes to panel code, rebuild the AVPI and copy it to:

```sh
/Library/Application Support/Avid/PanelSDKPlugins
```

Preferred local rebuild command:

```sh
scripts/install-dev-avpi-and-helper.sh /Users/admin/Github/mark --skip-helper
```

For a full local demo refresh, use:

```sh
scripts/install-dev-avpi-and-helper.sh /Users/admin/Github/mark
```

The helper listens on `http://localhost:4500` by default and needs
`TWELVELABS_API_KEY` in its environment for real analysis.

---
> Source: [tmmcaleer/mark](https://github.com/tmmcaleer/mark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
