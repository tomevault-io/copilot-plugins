## nobus-unified

> This repository targets the .NET 10 toolchain with full MAUI Android support. Use these steps to align your environment before working on the codebase:

# Repository Agent Guidelines

This repository targets the .NET 10 toolchain with full MAUI Android support. Use these steps to align your environment before working on the codebase:

## Install the .NET 10 SDK (RC2)
1. Fetch the official install script and make it executable:
   ```bash
   wget https://dotnet.microsoft.com/download/dotnet/scripts/v1/dotnet-install.sh
   chmod +x dotnet-install.sh
   ```
2. Resolve the latest .NET 10 RC2 SDK version and install it. The example below queries the release metadata service so the command keeps working as new release candidates ship (install `jq` first if your system is missing it):
   ```bash
   DOTNET_VERSION=$(curl -s https://dotnetcli.blob.core.windows.net/dotnet/release-metadata/10.0/releases.json \
     | jq -r '.releases[0].sdk.version')
   ./dotnet-install.sh --version "$DOTNET_VERSION" --install-dir "$HOME/.dotnet"
   ```
3. Persist the SDK on your `PATH` (add these exports to your shell profile):
   ```bash
   export DOTNET_ROOT="$HOME/.dotnet"
   export PATH="$DOTNET_ROOT:$PATH"
   ```

## Install MAUI workloads
The generic `maui` workload is only available on Windows and macOS. On Linux environments used for CI or headless builds install the Android-specific workload instead:

```bash
# Linux
dotnet workload install maui-android --skip-manifest-update --source https://api.nuget.org/v3/index.json

# Windows/macOS
dotnet workload install maui maui-android --skip-manifest-update --source https://api.nuget.org/v3/index.json
```

## Provision Android SDK components
1. Download the Android command-line tools bundle and unzip it under the SDK root:
   ```bash
   export ANDROID_SDK_ROOT="$HOME/android-sdk"
   mkdir -p "$ANDROID_SDK_ROOT/cmdline-tools"
   cd "$ANDROID_SDK_ROOT"
   wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
   unzip commandlinetools-linux-11076708_latest.zip -d cmdline-tools
   mv cmdline-tools/cmdline-tools cmdline-tools/latest
   ```
   (Replace the download URL with the latest package for your platform when Google publishes an update.)
2. Add the command-line tools to `PATH` and install the required platform components:
   ```bash
   export PATH="$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/platform-tools:$PATH"
   yes | sdkmanager "platforms;android-36" "platform-tools" "build-tools;36.0.0"
   yes | sdkmanager --licenses
   ```

Once these prerequisites are in place, run `dotnet workload restore` and `dotnet build NobUS.sln` to validate the setup.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/No-bUS)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/No-bUS)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
