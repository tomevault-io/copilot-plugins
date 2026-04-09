## my-ip-widget

> Build the exe with PyInstaller:

# My IP Widget

## Build

Build the exe with PyInstaller:

```
pyinstaller MyIPWidget.spec
```

## Code Signing

The exe must be signed to run on Windows 11 with Smart App Control enabled.

A self-signed certificate `CN=MyIPWidget` is installed in the user's certificate store (thumbprint: `BB4E35EFA194B0BAFD83EB7C7004DA20C7A667D5`). It has been added to `Cert:\CurrentUser\Root` and `Cert:\CurrentUser\TrustedPublisher`.

After each build, sign the exe:

```powershell
$cert = Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert | Where-Object { $_.Subject -eq "CN=MyIPWidget" }
Set-AuthenticodeSignature -FilePath "C:\Users\uk\apps\My-IP-Widget\dist\MyIPWidget.exe" -Certificate $cert
```

The status should show **Valid**. If not, ensure the certificate is still in the Root and TrustedPublisher stores.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/konstantin-uvarov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/konstantin-uvarov)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
