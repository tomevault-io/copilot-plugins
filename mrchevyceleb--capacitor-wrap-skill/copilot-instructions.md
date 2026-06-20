## capacitor-wrap-skill

> Wrap any web app in Capacitor for iOS/Android app store deployment with RevenueCat subscriptions


# Capacitor App Wrapper Skill

Wraps any web app (PWA or hosted URL) in Capacitor to create native iOS and Android apps ready for app store submission, with optional RevenueCat subscription integration.

**Last Updated:** January 2026 (Capacitor 8)

## What This Skill Does

1. **Installs Capacitor** and required plugins
2. **Configures remote URL** loading for your web app
3. **Generates app icons** and splash screens from your existing assets
4. **Generates app store screenshots** automatically for iOS and Android (NEW in v2.3)
5. **Sets up RevenueCat** for in-app subscriptions (optional)
6. **Configures native platforms** (Android/iOS) with proper styling
7. **Creates Codemagic config** for iOS cloud builds
8. **Generates store listing** text ready to copy-paste
9. **Provides build commands** for both Windows and macOS

## Prerequisites

Before running this skill, ensure you have:

### Development Environment (CRITICAL - Updated for 2025/2026)

| Requirement | Version | Notes |
|-------------|---------|-------|
| **Node.js** | **22+** | Capacitor 8 requires Node 22 (NOT 18) |
| **Java JDK** | **21** | Android builds require Java 21 (NOT 17) |
| **Android Studio** | Latest | For Android SDK and emulator |
| **Xcode** | Latest | For iOS builds (macOS only) |

### Verify Your Environment

```bash
# Check Node version (must be 22+)
node --version

# Check Java version (must be 21)
java --version

# If Java is wrong, install via:
# macOS: brew install openjdk@21
# Windows: Download from adoptium.net
```

### Other Requirements
- An existing web app (either hosted URL or local PWA)
- App icons (at least 512x512 PNG)
- For iOS cloud builds: Codemagic account OR macOS with latest Xcode

## Invocation

This skill is invoked when you ask Claude to:
- "Wrap my web app in Capacitor"
- "Create a native app from my website"
- "Prepare my app for the app stores"
- "Convert my PWA to iOS/Android"

## Workflow

### Phase 1: Gather Information

Ask the user for:

1. **App URL**: The web app URL to wrap (e.g., `https://myapp.com`)
2. **App Name**: Display name (e.g., "My App")
3. **iOS Bundle ID**: Unique identifier for iOS (e.g., `app.company.appname` or `com.company.ios`)
4. **Android Package Name**: Unique identifier for Android (e.g., `com.company.appname`)
5. **Theme Colors**: Primary and accent colors (hex values)
6. **RevenueCat**: Whether to integrate subscriptions

**CRITICAL: Bundle ID Best Practices**

iOS and Android use different bundle identifier conventions and they CAN (and often SHOULD) be different:

- **iOS Bundle ID**: Apple convention often uses `app.company.appname` or `com.company.appname`
  - Examples: `app.carousel.ios`, `ai.carouselcards.app`
- **Android Package Name**: Must use reverse domain notation (e.g., `com.company.appname`)
  - Examples: `com.carouselcards.app`, `com.company.myapp`

**WARNING: Bundle IDs are PERMANENT**
- Once you publish an app to the App Store or Google Play, the bundle ID cannot be changed
- Changing it would require creating a completely new app listing
- All users, reviews, and ratings would be lost
- Choose carefully - you're committing to this forever

**Why Separate Bundle IDs?**
- Platform-specific naming conventions (iOS often uses `app.*`, Android typically `com.*`)
- Different teams or developers may manage each platform
- Allows platform-specific analytics and tracking
- Prevents conflicts when both platforms use the same identifier format

### Phase 2: Install Dependencies

```bash
# Core Capacitor (v8)
npm install @capacitor/core @capacitor/cli
npm install @capacitor/android @capacitor/ios
npm install @capacitor/splash-screen @capacitor/status-bar @capacitor/app
npm install -D @capacitor/assets

# RevenueCat (if requested) - IMPORTANT: Only use purchases-capacitor, NOT purchases-capacitor-ui
npm install @revenuecat/purchases-capacitor
# DO NOT install @revenuecat/purchases-capacitor-ui - it's NOT compatible with Capacitor 8!
```

### Phase 3: Create Configuration

Create `capacitor.config.ts`:

```typescript
import type { CapacitorConfig } from "@capacitor/cli";

const config: CapacitorConfig = {
  // NOTE: The appId in capacitor.config.ts is used as the DEFAULT for both platforms
  // We'll set the Android package name here, then override iOS bundle ID in Xcode
  appId: "{{ANDROID_PACKAGE_NAME}}",
  appName: "{{APP_NAME}}",
  webDir: "out",
  server: {
    url: "{{APP_URL}}",
    allowNavigation: ["{{DOMAIN}}", "*.{{DOMAIN}}"],
  },
  ios: {
    contentInset: "automatic",
    backgroundColor: "{{PRIMARY_COLOR}}",
  },
  android: {
    backgroundColor: "{{PRIMARY_COLOR}}",
  },
  plugins: {
    SplashScreen: {
      launchShowDuration: 2000,
      backgroundColor: "{{PRIMARY_COLOR}}",
      showSpinner: true,
      spinnerColor: "{{ACCENT_COLOR}}",
    },
  },
};

export default config;
```

**Important:** The `appId` field in capacitor.config.ts serves as the default for both platforms. However:
- Android will use this value from capacitor.config.ts (but should also be explicitly set in build.gradle)
- iOS bundle ID must be set separately in Xcode project settings (see Phase 8)

### Phase 4: Add Native Platforms

```bash
# Create placeholder out directory
mkdir -p out
echo '<!DOCTYPE html><html><head><meta http-equiv="refresh" content="0;url={{APP_URL}}"></head><body></body></html>' > out/index.html

# Add platforms
npx cap add android
npx cap add ios

# Sync
npx cap sync
```

### Phase 5: Generate Icons

If source icons exist, create icon generation script:

```javascript
// scripts/generate-app-icons.mjs
import sharp from 'sharp';

// Generate 1024x1024 icon with solid background
// Generate 2732x2732 splash screen
// See full implementation in skill
```

Then generate all sizes:
```bash
node scripts/generate-app-icons.mjs
npx capacitor-assets generate --android --ios
```

### Phase 6: Configure Android

**IMPORTANT: Fix gradle permissions first (required on macOS/Linux):**
```bash
chmod +x android/gradlew
```

#### 6.1: Set Android Package Name

**CRITICAL:** Android requires the package name to be set in multiple places for proper configuration.

Update `android/app/build.gradle` - find the `android { }` block and add/update:

```groovy
android {
    namespace "{{ANDROID_PACKAGE_NAME}}"
    compileSdkVersion rootProject.ext.compileSdkVersion

    defaultConfig {
        applicationId "{{ANDROID_PACKAGE_NAME}}"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 1
        versionName "1.0"
    }
    // ... rest of config
}
```

**Key fields:**
- `namespace`: The package namespace for your app (REQUIRED in modern Android/Gradle)
- `applicationId`: The unique package identifier (this is what appears in Google Play)

Update `android/app/src/main/res/values/strings.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">{{APP_NAME}}</string>
    <string name="title_activity_main">{{APP_NAME}}</string>
    <string name="package_name">{{ANDROID_PACKAGE_NAME}}</string>
    <string name="custom_url_scheme">{{ANDROID_PACKAGE_NAME}}</string>
</resources>
```

#### 6.2: Configure Theme Colors

Update `android/app/src/main/res/values/colors.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">{{PRIMARY_COLOR}}</color>
    <color name="colorPrimaryDark">{{PRIMARY_COLOR}}</color>
    <color name="colorAccent">{{ACCENT_COLOR}}</color>
</resources>
```

#### 6.3: Update AndroidManifest.xml

Update `AndroidManifest.xml` with:
- Deep links for app URL
- Billing permission (if RevenueCat)
- HTTPS-only traffic

### Phase 6.5: Generate Release Keystore & Signing Config (AUTOMATED)

**CRITICAL**: Google Play requires signed AABs. This phase automatically generates a secure keystore and signing configuration.

#### 1. Generate secure password and keystore automatically:

```bash
# Create android-signing directory
mkdir -p android-signing

# Generate cryptographically secure password (32-byte base64)
KEYSTORE_PASSWORD=$(openssl rand -base64 32)

# Generate keystore with secure password (non-interactive)
keytool -genkey -v \
  -keystore android-signing/{{APP_NAME_LOWER}}-release.keystore \
  -alias {{APP_NAME_LOWER}} \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -storepass "$KEYSTORE_PASSWORD" \
  -keypass "$KEYSTORE_PASSWORD" \
  -dname "CN={{APP_NAME}}, OU=Engineering, O={{APP_NAME}}, L=Unknown, ST=Unknown, C=US"

echo "Keystore generated successfully!"
echo "Password: $KEYSTORE_PASSWORD"
```

**What This Does:**
- Creates `android-signing/` folder for all signing materials
- Generates a cryptographically secure 32-character password
- Creates the keystore non-interactively (no prompts!)
- Uses the same password for both keystore and key (recommended)
- Sets generic but valid identity information
- Displays the password for you to copy

**Windows PowerShell Alternative:**
```powershell
# Create android-signing directory
New-Item -ItemType Directory -Force -Path android-signing

# Generate cryptographically secure password
$bytes = New-Object byte[] 32
[System.Security.Cryptography.RandomNumberGenerator]::Create().GetBytes($bytes)
$KEYSTORE_PASSWORD = [Convert]::ToBase64String($bytes)

# Generate keystore (save password first!)
Write-Host "Generated Password: $KEYSTORE_PASSWORD" -ForegroundColor Green
Write-Host "SAVE THIS PASSWORD NOW!" -ForegroundColor Yellow

keytool -genkey -v `
  -keystore android-signing/{{APP_NAME_LOWER}}-release.keystore `
  -alias {{APP_NAME_LOWER}} `
  -keyalg RSA `
  -keysize 2048 `
  -validity 10000 `
  -storepass "$KEYSTORE_PASSWORD" `
  -keypass "$KEYSTORE_PASSWORD" `
  -dname "CN={{APP_NAME}}, OU=Engineering, O={{APP_NAME}}, L=Unknown, ST=Unknown, C=US"
```

#### 2. Save credentials to local files automatically:

Create `android-signing/CREDENTIALS.txt`:
```bash
cat > android-signing/CREDENTIALS.txt << EOF
# {{APP_NAME}} Android Keystore Credentials
# KEEP THIS FILE SAFE AND NEVER COMMIT TO GIT!

Generated: $(date)

Keystore File: android-signing/{{APP_NAME_LOWER}}-release.keystore
Keystore Password: $KEYSTORE_PASSWORD
Key Alias: {{APP_NAME_LOWER}}
Key Password: $KEYSTORE_PASSWORD

# For Codemagic Environment Variables:
# (Copy these values when setting up your CI/CD)
{{APP_NAME_UPPER}}_KEYSTORE_PASSWORD: $KEYSTORE_PASSWORD
{{APP_NAME_UPPER}}_KEY_ALIAS: {{APP_NAME_LOWER}}
{{APP_NAME_UPPER}}_KEY_PASSWORD: $KEYSTORE_PASSWORD

# For base64-encoded keystore (Codemagic file variable):
# Run: base64 -w 0 android-signing/{{APP_NAME_LOWER}}-release.keystore
# Or on macOS: base64 -i android-signing/{{APP_NAME_LOWER}}-release.keystore
EOF

echo "Credentials saved to android-signing/CREDENTIALS.txt"
```

**Windows PowerShell:**
```powershell
@"
# {{APP_NAME}} Android Keystore Credentials
# KEEP THIS FILE SAFE AND NEVER COMMIT TO GIT!

Generated: $(Get-Date -Format "yyyy-MM-dd HH:mm:ss")

Keystore File: android-signing/{{APP_NAME_LOWER}}-release.keystore
Keystore Password: $KEYSTORE_PASSWORD
Key Alias: {{APP_NAME_LOWER}}
Key Password: $KEYSTORE_PASSWORD

# For Codemagic Environment Variables:
# (Copy these values when setting up your CI/CD)
{{APP_NAME_UPPER}}_KEYSTORE_PASSWORD: $KEYSTORE_PASSWORD
{{APP_NAME_UPPER}}_KEY_ALIAS: {{APP_NAME_LOWER}}
{{APP_NAME_UPPER}}_KEY_PASSWORD: $KEYSTORE_PASSWORD

# For base64-encoded keystore (Codemagic file variable):
# Run: certutil -encode android-signing\{{APP_NAME_LOWER}}-release.keystore android-signing\keystore-base64.txt
# Then open keystore-base64.txt and remove header/footer lines
"@ | Out-File -FilePath android-signing\CREDENTIALS.txt -Encoding UTF8

Write-Host "Credentials saved to android-signing\CREDENTIALS.txt" -ForegroundColor Green
```

Create `android-signing/SETUP-INSTRUCTIONS.md` with detailed Codemagic and Google Play setup steps (see template provided in Phase 6.6 below).

#### 3. Update `android/app/build.gradle` with signing config:

Add inside the `android { }` block:
```groovy
signingConfigs {
    release {
        if (System.getenv("{{APP_NAME_UPPER}}_KEYSTORE_PATH")) {
            storeFile file(System.getenv("{{APP_NAME_UPPER}}_KEYSTORE_PATH"))
            storePassword System.getenv("{{APP_NAME_UPPER}}_KEYSTORE_PASSWORD")
            keyAlias System.getenv("{{APP_NAME_UPPER}}_KEY_ALIAS")
            keyPassword System.getenv("{{APP_NAME_UPPER}}_KEY_PASSWORD")
        }
    }
}
```

Update the `buildTypes { release { } }` block to use signing:
```groovy
buildTypes {
    release {
        minifyEnabled true
        shrinkResources true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        if (System.getenv("{{APP_NAME_UPPER}}_KEYSTORE_PATH")) {
            signingConfig signingConfigs.release
        }
    }
}
```

#### 4. Add android-signing folder to `.gitignore`:
```
# Android signing (NEVER commit these!)
android-signing/
*.keystore
*.jks
```

**CRITICAL SECURITY NOTES:**
- The `android-signing/` folder contains EVERYTHING needed to sign your app
- If you lose the keystore, you can NEVER update your app on Google Play
- If someone steals your keystore, they can publish malicious updates
- Back up this folder to a secure location (encrypted USB, password manager, etc.)
- NEVER commit it to git, even in private repositories

### Phase 6.6: Create Setup Instructions

Create `android-signing/SETUP-INSTRUCTIONS.md` with step-by-step Codemagic and Google Play setup:

```markdown
# {{APP_NAME}} - Android Setup Instructions

This guide walks you through uploading your signed Android app to Google Play via Codemagic CI/CD.

## Prerequisites

- [ ] Android app created in Google Play Console
- [ ] Keystore file generated (android-signing/{{APP_NAME_LOWER}}-release.keystore)
- [ ] Credentials saved (android-signing/CREDENTIALS.txt)
- [ ] Codemagic account connected to your repository

## Step 1: Prepare Keystore for Codemagic

Codemagic requires the keystore as a base64-encoded file variable.

### On macOS/Linux:
\`\`\`bash
base64 -i android-signing/{{APP_NAME_LOWER}}-release.keystore | pbcopy
\`\`\`
This copies the base64 string to your clipboard.

### On Windows (PowerShell):
\`\`\`powershell
certutil -encode android-signing\{{APP_NAME_LOWER}}-release.keystore android-signing\keystore-base64.txt
\`\`\`
Then open `android-signing\keystore-base64.txt`, remove the header/footer lines, and copy the base64 content.

### On Windows (Git Bash):
\`\`\`bash
base64 -w 0 android-signing/{{APP_NAME_LOWER}}-release.keystore | clip
\`\`\`

## Step 2: Add Environment Variables to Codemagic

1. Go to your Codemagic application settings
2. Navigate to **Environment variables** section
3. Add the following variables:

| Variable Name | Value | Secure? |
|---------------|-------|---------|
| \`{{APP_NAME_UPPER}}_KEYSTORE\` | [Paste base64 keystore] | Yes |
| \`{{APP_NAME_UPPER}}_KEYSTORE_PASSWORD\` | [From CREDENTIALS.txt] | Yes |
| \`{{APP_NAME_UPPER}}_KEY_ALIAS\` | {{APP_NAME_LOWER}} | No |
| \`{{APP_NAME_UPPER}}_KEY_PASSWORD\` | [From CREDENTIALS.txt] | Yes |

**IMPORTANT**: Mark password variables as "Secure" so they're encrypted!

## Step 3: Update codemagic.yaml

Add Android workflow to your `codemagic.yaml`:

\`\`\`yaml
workflows:
  android-release:
    name: Android Release
    instance_type: linux_x2
    max_build_duration: 60
    environment:
      android_signing:
        - keystore_reference: {{APP_NAME_UPPER}}_KEYSTORE
      vars:
        PACKAGE_NAME: "{{ANDROID_PACKAGE_NAME}}"
      node: 22
    scripts:
      - name: Install dependencies
        script: npm ci

      - name: Build web app
        script: npm run build

      - name: Sync Capacitor
        script: npx cap sync android

      - name: Set up keystore
        script: |
          echo \${{APP_NAME_UPPER}}_KEYSTORE | base64 --decode > android/keystore.jks

      - name: Build Android release
        script: |
          cd android
          export {{APP_NAME_UPPER}}_KEYSTORE_PATH=../android/keystore.jks
          ./gradlew bundleRelease

    artifacts:
      - android/app/build/outputs/**/*.aab

    publishing:
      google_play:
        credentials: [Your Service Account JSON]
        track: internal  # or: alpha, beta, production
        submit_as_draft: true
\`\`\`

## Step 4: Create Google Play Service Account (for automatic upload)

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project (or select existing)
3. Enable **Google Play Android Developer API**
4. Create a service account:
   - IAM & Admin > Service Accounts > Create Service Account
   - Name: "Codemagic Publisher"
   - Grant role: Service Account User
   - Create JSON key and download

5. Link service account to Google Play:
   - Google Play Console > Setup > API access
   - Link the project
   - Grant access to the service account with "Release to production" permission

6. Upload JSON key to Codemagic:
   - Codemagic > Teams > Integrations > Google Play
   - Upload the service account JSON
   - Name it (e.g., "{{APP_NAME}} Publisher")

## Step 5: Create Internal Testing Track (Recommended First Step)

Before releasing to production:

1. Google Play Console > Your App > Testing > Internal testing
2. Create internal testing release
3. Upload your AAB manually first time to verify
4. Add testers (your email, team members)
5. Test thoroughly before promoting to production

## Step 6: Push to Trigger Build

\`\`\`bash
git add .
git commit -m "Add Android release configuration"
git push origin main
\`\`\`

Codemagic will automatically:
- Build your web app
- Sync to Android
- Sign the AAB with your keystore
- Upload to Google Play (if configured)

## Step 7: Manual Upload (Alternative to Codemagic Auto-Upload)

If you prefer manual control:

1. Download the AAB from Codemagic artifacts
2. Go to Google Play Console > Your App > Release > Production (or Testing)
3. Create new release
4. Upload the AAB
5. Fill in release notes
6. Review and roll out

## Troubleshooting

### "Keystore was tampered with, or password was incorrect"
- Verify password in CREDENTIALS.txt matches what you used
- Check that base64 encoding was done correctly (no line breaks on Linux)

### "Failed to read key from keystore"
- Ensure key alias matches: {{APP_NAME_LOWER}}
- Verify key password is correct (may be different from keystore password)

### "Version code X has already been used"
- Increment `versionCode` in `android/app/build.gradle`
- Each upload must have a unique version code

### Build fails with signing errors
- Check that all environment variables are set in Codemagic
- Verify keystore file is not corrupted
- Make sure secure variables are marked as "Secure"

## Security Reminders

- NEVER commit keystore files to git
- NEVER share CREDENTIALS.txt publicly
- Keep backup of android-signing/ folder in secure location
- Rotate service account keys periodically (Google Cloud best practice)
- Use "submit_as_draft: true" for manual release control

## Next Steps

After successful Android release:

- [ ] Test on multiple devices (different Android versions)
- [ ] Submit for review in Google Play Console
- [ ] Set up release notes and screenshots
- [ ] Configure app store listing (description, images, etc.)
- [ ] Plan update strategy (versioning, release cadence)

---

**Generated:** $(date)
**App Name:** {{APP_NAME}}
**Package Name:** {{ANDROID_PACKAGE_NAME}}
\`\`\`

Save this to `android-signing/SETUP-INSTRUCTIONS.md` automatically or provide it to the user for manual creation.

### Phase 7: RevenueCat Integration (Optional)

**CRITICAL for Capacitor 8:** Only use `@revenuecat/purchases-capacitor`. The UI package (`@revenuecat/purchases-capacitor-ui`) depends on Capacitor 7 and is NOT compatible!

Create `lib/services/revenuecat.ts`:

```typescript
import { Purchases, LOG_LEVEL } from '@revenuecat/purchases-capacitor';
import { Capacitor } from '@capacitor/core';

// Platform-specific API keys
const REVENUECAT_IOS_KEY = import.meta.env.VITE_REVENUECAT_IOS_API_KEY;
const REVENUECAT_ANDROID_KEY = import.meta.env.VITE_REVENUECAT_ANDROID_API_KEY;

export async function initializeRevenueCat(userId?: string) {
  const platform = Capacitor.getPlatform();
  
  // Skip on web
  if (platform === 'web') {
    console.log('RevenueCat not available on web');
    return;
  }
  
  // Use correct API key for platform
  const apiKey = platform === 'ios' ? REVENUECAT_IOS_KEY : REVENUECAT_ANDROID_KEY;
  
  if (!apiKey) {
    console.error(`No RevenueCat API key for platform: ${platform}`);
    return;
  }
  
  await Purchases.configure({
    apiKey,
    appUserID: userId,
  });
  
  if (import.meta.env.DEV) {
    await Purchases.setLogLevel({ level: LOG_LEVEL.DEBUG });
  }
}

// ... rest of RevenueCat service methods
```

**Environment Variables Required:**
- `VITE_REVENUECAT_IOS_API_KEY` - Your iOS public API key from RevenueCat
- `VITE_REVENUECAT_ANDROID_API_KEY` - Your Android public API key from RevenueCat

### Phase 8: iOS Configuration (CRITICAL - Capacitor 8 Changes)

**IMPORTANT:** Capacitor 8 uses **Swift Package Manager (SPM)** instead of CocoaPods!

#### Key Differences from Capacitor 7:
- **NO CocoaPods** - Don't run `pod install`
- **Use `.xcodeproj`** - NOT `.xcworkspace`
- **Must resolve SPM dependencies** before building

#### 8.1: Set iOS Bundle ID in Xcode

**CRITICAL:** The iOS bundle ID must be set manually in Xcode - it cannot be fully configured from capacitor.config.ts alone.

After running `npx cap add ios`, you MUST set the bundle ID in Xcode:

1. Navigate to the iOS project:
```bash
cd ios/App
```

2. Resolve Swift Package Manager dependencies:
```bash
xcodebuild -resolvePackageDependencies -project App.xcodeproj -scheme App
```

3. Open the project in Xcode:
```bash
open App.xcodeproj
```

4. **Set the Bundle Identifier:**
   - In Xcode, select the **App** project in the left sidebar
   - Select the **App** target under TARGETS
   - Go to the **General** tab
   - Find **Identity** section
   - Change **Bundle Identifier** from the default to your iOS bundle ID: `{{IOS_BUNDLE_ID}}`
   - Example: `app.carousel.ios` or `ai.carouselcards.app`

5. **Verify in Build Settings (optional but recommended):**
   - Go to the **Build Settings** tab
   - Search for "Product Bundle Identifier"
   - Confirm it shows your iOS bundle ID

**Alternative: Command Line (for automation):**
```bash
# Update the project.pbxproj file with your iOS bundle ID
# This can be done in a script if automating the process
sed -i '' 's/PRODUCT_BUNDLE_IDENTIFIER = .*/PRODUCT_BUNDLE_IDENTIFIER = {{IOS_BUNDLE_ID}};/g' ios/App/App.xcodeproj/project.pbxproj
```

**IMPORTANT NOTES:**
- The bundle ID in Xcode may initially match capacitor.config.ts `appId`
- You MUST change it manually to use your separate iOS bundle ID
- This is a one-time configuration - subsequent `npx cap sync` won't overwrite it
- The iOS bundle ID and Android package name can now be completely different

#### 8.2: Manual Build (macOS):
```bash
# Navigate to iOS project
cd ios/App

# Resolve Swift Package Manager dependencies FIRST
xcodebuild -resolvePackageDependencies -project App.xcodeproj -scheme App

# Open in Xcode (use .xcodeproj, NOT .xcworkspace!)
open App.xcodeproj
```

#### 8.3: Codemagic Build Script

**IMPORTANT:** Use your iOS bundle ID (not Android package name) in the Codemagic workflow.

```yaml
ios-app:
  name: iOS App Store
  instance_type: mac_mini_m2
  max_build_duration: 60
  integrations:
    app_store_connect: "Your Integration Name"  # Must match exactly!
  environment:
    ios_signing:
      distribution_type: app_store
      bundle_identifier: "{{IOS_BUNDLE_ID}}"  # Use iOS bundle ID, not Android package name
    vars:
      BUNDLE_ID: "{{IOS_BUNDLE_ID}}"  # iOS bundle ID for signing
      XCODE_SCHEME: "App"
      XCODE_PROJECT: "ios/App/App.xcodeproj"  # Use .xcodeproj NOT .xcworkspace!
    node: 22  # Capacitor 8 requires Node 22
  scripts:
    - name: Install dependencies
      script: npm ci
    
    - name: Build web app
      script: npm run build
    
    - name: Sync Capacitor
      script: npx cap sync ios
    
    - name: Resolve Swift Package Dependencies
      script: |
        cd ios/App
        xcodebuild -resolvePackageDependencies -project App.xcodeproj -scheme App
    
    - name: Set up code signing
      script: |
        # Fetch signing files from App Store Connect
        app-store-connect fetch-signing-files "$BUNDLE_ID" \
          --type IOS_APP_STORE \
          --create
        
        # Initialize keychain
        keychain initialize
        
        # Add certificates to keychain
        keychain add-certificates
        
        # Configure Xcode project to use profiles
        xcode-project use-profiles --project "$XCODE_PROJECT"
    
    - name: Build iOS app
      script: |
        cd ios/App
        xcodebuild \
          -project App.xcodeproj \
          -scheme "$XCODE_SCHEME" \
          -configuration Release \
          -archivePath build/App.xcarchive \
          archive \
          CODE_SIGN_STYLE=Manual
    
    - name: Export IPA
      script: |
        xcode-project export-ipa \
          --project "$XCODE_PROJECT" \
          --archive build/App.xcarchive
  
  artifacts:
    - ios/App/build/**/*.ipa
    - ios/App/build/**/*.xcarchive
  
  publishing:
    app_store_connect:
      auth: integration
      # Upload only - allows manual submission after filling in metadata
      # Set to true only if all app metadata is already configured
      submit_to_app_store: false
```

**Publishing Workflow Options:**

The skill configures **upload-only mode** (`submit_to_app_store: false`) by default. This is recommended because:

✅ **Pros:**
- Builds show as successful when upload completes
- You control when to submit for review
- Fill in metadata (screenshots, description, privacy info) at your own pace
- Standard workflow used by most developers

❌ **Auto-Submit Mode** (`submit_to_app_store: true`):
- Requires all metadata filled in beforehand
- Fails if encryption declaration or other fields are missing
- Can't update screenshots/description per build
- Less flexible for iterative development

**Post-Build Workflow (Upload-Only):**
1. Build completes → IPA uploaded to App Store Connect ✅
2. Go to App Store Connect dashboard
3. Navigate to your app → Select uploaded build
4. Fill in/update metadata:
   - Screenshots (6.5" and 5.5" displays required)
   - App description and keywords
   - Privacy policy URL and support URL
   - Encryption declaration (usually "NO" for standard apps)
5. Click "Submit for Review" when ready

### Phase 9: Generate App Store Screenshots (Automated)

**NEW in v2.3:** Automated screenshot generation using Playwright for both iOS and Android app stores.

#### 9.1: Install Playwright

```bash
npm install -D playwright
npx playwright install chromium
```

#### 9.2: Create Screenshot Generation Script

Copy the screenshot generator to `scripts/generate-app-screenshots.mjs` (see examples/generate-app-screenshots.mjs in the skill repository).

The script automatically generates:
- **iOS Screenshots**: 6.9" iPhone (mandatory), 6.7" iPhone, 6.5" iPhone, 5.5" iPhone, 13" iPad Pro (mandatory), 12.9" iPad Pro
- **Android Phone Screenshots**: 1080x1920 (recommended), 1440x2560 (HD) - saved to `screenshots/android/phone/`
- **Android 7" Tablet Screenshots**: 1200x1920 - saved to `screenshots/android/tablet-7/` (optional)
- **Android 10" Tablet Screenshots**: 1600x2560 - saved to `screenshots/android/tablet-10/` (optional)

**Note:** The script now creates separate subdirectories for Android phone and tablet screenshots to match Google Play Store requirements.

#### 9.3: Customize Screenshot Scenarios

Edit the `CONFIG.scenarios` array in the script to capture your app's key screens:

```javascript
scenarios: [
  {
    name: 'home',
    description: 'Home Screen',
    path: '/',
    waitForSelector: 'body',
  },
  {
    name: 'create-card',
    description: 'Card Creation Flow',
    path: '/create',
    waitForSelector: '.wizard-container',
    actions: async (page) => {
      // Optional: perform actions before screenshot
      await page.click('#start-button');
      await page.waitForTimeout(1000);
    }
  },
  // Add more scenarios for your app's features
]
```

#### 9.4: Generate Screenshots

```bash
# Start your dev server first
npm run dev

# In another terminal, generate screenshots
node scripts/generate-app-screenshots.mjs
```

Or for a production build:

```bash
# Build and preview
npm run build
npm run preview

# Generate from preview
APP_URL=http://localhost:4173 node scripts/generate-app-screenshots.mjs
```

**Output:**
- `screenshots/ios/` - iOS App Store screenshots (6 sizes x number of scenarios)
- `screenshots/android/phone/` - Google Play phone screenshots (2 sizes x number of scenarios)
- `screenshots/android/tablet-7/` - Google Play 7" tablet screenshots (1 size x number of scenarios, optional)
- `screenshots/android/tablet-10/` - Google Play 10" tablet screenshots (1 size x number of scenarios, optional)

**Add to `.gitignore`:**
```gitignore
# Screenshots (optional - commit if you want to version them)
screenshots/
```

#### 9.5: Generate Android Feature Graphic (REQUIRED)

Google Play Store **REQUIRES** a feature graphic (1024 x 500 px banner) that appears at the top of your store listing.

**Create the feature graphic generator:**

Copy `examples/generate-feature-graphic.mjs` to `scripts/generate-feature-graphic.mjs` and customize:

```javascript
const CONFIG = {
  // App branding - UPDATE THESE
  appName: 'Your App Name',
  tagline: 'Your App Tagline',

  // Brand colors - UPDATE THESE
  colors: {
    gradientStart: '#37B5B6',  // Left side of gradient
    gradientEnd: '#F47C74',     // Right side of gradient
    textColor: '#FFFFFF',       // Text color
  },

  // Logo path - point to your app icon or logo
  logoPath: join(projectRoot, 'public', 'icons', 'icon-512x512.png'),
};
```

**Generate the feature graphic:**

```bash
node scripts/generate-feature-graphic.mjs
```

**Output:**
- `screenshots/android/feature-graphic.png` (1024 x 500 px)

**Requirements:**
- Exact dimensions: 1024 x 500 px (MANDATORY)
- Format: PNG or JPEG
- Max size: 15MB
- This graphic is REQUIRED for Google Play Store submission
- It appears prominently at the top of your store listing

**Design Tips:**
- Use high contrast text for readability
- Include your app logo, name, and a short tagline
- Use your brand colors for consistency
- Keep important content centered (edges may be cropped on some devices)
- Test in Google Play Console preview before finalizing

#### 9.6: App Store Screenshot Requirements (2026)

**Apple App Store:**
- **Mandatory:** 6.9" iPhone (1320x2868) and 13" iPad (2064x2752)
- Optional: Additional iPhone sizes (6.7", 6.5", 5.5") and iPad 12.9"
- Format: PNG or JPEG, 72 DPI, no transparency
- Quantity: 1-10 screenshots per device size

**Google Play Store:**
- **Phone Screenshots (REQUIRED):**
  - Minimum: 2 screenshots
  - Maximum: 8 screenshots
  - Recommended dimensions: 1080x1920 (9:16 aspect ratio)
  - For promotion eligibility: 4+ screenshots at 1080px minimum
  - Format: PNG or JPEG, max 8MB per file
- **Feature Graphic (REQUIRED):**
  - Exact dimensions: 1024 x 500 px (MANDATORY)
  - Format: PNG or JPEG, max 15MB
  - This banner appears at the top of your store listing
  - Use the feature graphic generator (see Phase 9.5 above)
- **Tablet Screenshots (OPTIONAL but recommended):**
  - 7-inch tablet: 1200x1920 (2-8 screenshots)
  - 10-inch tablet: 1600x2560 (2-8 screenshots)
  - Improves discoverability on tablets

**Pro Tips:**
- Capture real user flows, not just static screens
- Show your app's unique value proposition
- Consider adding text overlays explaining features (do this manually after generation)
- Test on actual devices if possible before finalizing
- Localize screenshots for major markets (can re-run script with localized app)

### Phase 10: Monetization Setup (RevenueCat + App Stores)

**NEW in v2.5:** Comprehensive monetization setup guide for in-app purchases and subscriptions.

If your app includes subscriptions or consumables (credits, tokens, coins), follow this phase to set up RevenueCat integration and configure products in both app stores.

#### Overview

**Why RevenueCat?**
- Single API for both iOS and Android
- Automatic receipt validation
- Subscription analytics and insights
- Webhook support for backend sync
- Free up to $2.5M in tracked revenue

**What You'll Set Up:**
1. Product IDs in your code (subscriptions and consumables)
2. RevenueCat project with iOS and Android apps
3. In-app purchases in App Store Connect (iOS)
4. In-app products in Google Play Console (Android)
5. RevenueCat integration (entitlements, offerings, API keys)
6. Testing with sandbox accounts

#### Step 1: Define Product IDs in Code

Create `services/iap.ts` with your product configuration:

```typescript
export const IAP_PRODUCTS = {
  // Subscriptions (auto-renewable)
  PRO_MONTHLY: 'pro_monthly',
  PRO_YEARLY: 'pro_yearly',

  // Consumables (credits, tokens, coins)
  CREDITS_50: 'credits_50',
  CREDITS_200: 'credits_200',
  CREDITS_500: 'credits_500',
} as const;

export const PRODUCT_INFO: Record<IAPProductId, IAPProduct> = {
  [IAP_PRODUCTS.PRO_MONTHLY]: {
    id: IAP_PRODUCTS.PRO_MONTHLY,
    title: 'Pro Monthly',
    description: 'Unlimited AI generations & premium features',
    price: '$4.99/month',
    type: 'subscription',
  },
  // ... see examples/iap-products-config.ts for full template
};
```

**Product Naming Patterns:**
- Subscriptions: `{tier}_{period}` (e.g., `pro_monthly`, `premium_yearly`)
- Consumables: `credits_{amount}` or `tokens_{amount}` (e.g., `credits_50`)

**CRITICAL:** Product IDs must match EXACTLY across:
- Your code
- RevenueCat dashboard
- App Store Connect (iOS)
- Google Play Console (Android)

#### Step 2: Create RevenueCat Account

1. Go to [app.revenuecat.com](https://app.revenuecat.com)
2. Sign up (free tier available)
3. Create a new project
4. Add iOS app with your iOS bundle ID
5. Add Android app with your Android package name

**Note:** iOS and Android are configured separately with their own API keys.

#### Step 3: Configure App Store Connect (iOS)

**Create Subscriptions:**
1. App Store Connect → My Apps → Your App → **Features** → **In-App Purchases**
2. Click **+** → **Auto-Renewable Subscription**
3. Product ID: `pro_monthly` (must match code!)
4. Create/select subscription group
5. Duration: 1 month (or 1 year for yearly)
6. Price: $4.99 USD (add all territories)
7. Add localization (English required)
8. Save → **Submit for Review**

**Create Consumables:**
1. Click **+** → **Consumable**
2. Product ID: `credits_50` (must match code!)
3. Price: $4.99 USD
4. Add localization
5. Save → **Submit for Review**

**Get Shared Secret:**
- My Apps → Your App → App Store tab → App Information
- Scroll to "App-Specific Shared Secret"
- Generate → Copy to RevenueCat integration

#### Step 4: Configure Google Play Console (Android)

**Create Subscriptions:**
1. Play Console → Your App → **Monetize** → **Products** → **Subscriptions**
2. Click **Create subscription**
3. Product ID: `pro_monthly` (must match code!)
4. Name: Pro Monthly
5. Base plan: `monthly`, billing period: 1 month
6. Price: $4.99 USD (auto-converts currencies)
7. Optional: Enable 7-day free trial
8. **Activate** (top right)

**Create Consumables:**
1. **In-app products** tab → **Create product**
2. Product ID: `credits_50` (must match code!)
3. Name: 50 Credits
4. Price: $4.99 USD
5. Status: **Active**
6. Save

**Get Service Account JSON:**
- Play Console → Setup → API access
- Link Google Cloud project
- Create service account
- Download JSON key
- Upload to RevenueCat integration

#### Step 5: Configure RevenueCat

**Create Entitlements:**
1. RevenueCat dashboard → **Entitlements** → **New Entitlement**
2. Identifier: `pro` (or `premium`, `elite`)
3. Description: "Pro features access"

**Create Products:**
1. **Products** → **New Product**
2. Product ID: `pro_monthly` (match code!)
3. Type: Subscription or Consumable
4. Store Product IDs:
   - iOS: `pro_monthly` (from App Store Connect)
   - Android: `pro_monthly` (from Google Play Console)
5. Save

**Create Offerings:**
1. **Offerings** → **New Offering**
2. Identifier: `default`
3. Add packages:
   - Package: `$rc_monthly` → Product: `pro_monthly`
   - Package: `$rc_annual` → Product: `pro_yearly`
4. Save

**Get API Keys:**
1. **Settings** → **API Keys**
2. Copy iOS Public SDK Key (`appl_...`)
3. Copy Android Public SDK Key (`goog_...`)
4. Add to `.env`:
   ```
   VITE_REVENUECAT_IOS_API_KEY=appl_xxx
   VITE_REVENUECAT_ANDROID_API_KEY=goog_xxx
   ```

#### Step 6: Testing

**iOS Sandbox Testing:**
1. App Store Connect → Users and Access → **Sandbox Testers**
2. Create tester account (unique email)
3. On device: Sign out of real App Store account
4. Install app, make purchase, sign in with sandbox account
5. Verify purchase appears in RevenueCat dashboard

**Android License Testing:**
1. Play Console → Settings → **License Testing**
2. Add tester email (your Google account)
3. Install app via internal testing or debug
4. Sign in with Google account
5. Test purchases (instant and free)

#### Recommended Pricing

**Subscriptions:**
- Monthly: $4.99 - $9.99
- Yearly: $39.99 - $79.99 (20-30% savings vs monthly)
- Free trial: 3-7 days (recommended)

**Consumables:**
- Starter pack: $4.99 (50-100 credits)
- Value pack: $14.99 (200-300 credits)
- Best value: $29.99 (500-1000 credits)

**Pro Tip:** Offer 3 tiers to leverage "goldilocks effect" - most users choose middle option.

#### Common Pitfalls

1. **Product ID mismatches** - Copy-paste IDs, don't retype!
2. **Wrong API key** - iOS and Android have separate keys
3. **Products not activated (Android)** - Must click "Activate" button
4. **Products not approved (iOS)** - Can still test in sandbox before approval
5. **Subscription groups** - All tiers must be in same group (iOS)

#### Resources

For complete step-by-step instructions:
- **Full Guide:** `examples/MONETIZATION-SETUP-TEMPLATE.md`
- **Quick Reference:** `examples/MONETIZATION-QUICK-REFERENCE.md`
- **Code Template:** `examples/iap-products-config.ts`
- **RevenueCat Docs:** https://www.revenuecat.com/docs

### Phase 11: Create Build Configurations

Create `codemagic.yaml` for CI/CD:
- iOS App Store workflow (see Phase 8)
- iOS TestFlight workflow
- Android Play Store workflow
- Android debug workflow

### Phase 12: Generate Documentation

Create comprehensive guide including:
- Build commands for Windows/macOS
- Keystore generation instructions
- Store listing text (copy-paste ready)
- Screenshot requirements
- Deep link setup

## Output Files

| File | Purpose |
|------|---------|
| `capacitor.config.ts` | Main Capacitor configuration |
| `public/error.html` | Offline fallback page |
| `assets/*.png` | Source icons for generation |
| `lib/services/revenuecat.ts` | RevenueCat integration (optional) |
| `app/api/webhooks/revenuecat/route.ts` | Webhook handler (optional) |
| `codemagic.yaml` | iOS/Android CI/CD config |
| `docs/APP_STORE_GUIDE.md` | Store submission guide |
| `scripts/generate-app-icons.mjs` | Icon generation script |
| `scripts/generate-app-screenshots.mjs` | **NEW:** Automated screenshot generator for iOS & Android |
| `scripts/generate-feature-graphic.mjs` | **NEW:** Android feature graphic generator (1024x500 REQUIRED) |
| `screenshots/ios/*.png` | **NEW:** iOS App Store screenshots (6 sizes per scenario) |
| `screenshots/android/phone/*.png` | **NEW:** Android phone screenshots (1080x1920, 1440x2560) |
| `screenshots/android/tablet-7/*.png` | **NEW:** Android 7" tablet screenshots (1200x1920, optional) |
| `screenshots/android/tablet-10/*.png` | **NEW:** Android 10" tablet screenshots (1600x2560, optional) |
| `screenshots/android/feature-graphic.png` | **NEW:** Android feature graphic (1024x500 REQUIRED) |
| `android-signing/{{APP_NAME_LOWER}}-release.keystore` | Release signing key (DO NOT COMMIT) |
| `android-signing/CREDENTIALS.txt` | Keystore passwords (DO NOT COMMIT) |
| `android-signing/SETUP-INSTRUCTIONS.md` | Step-by-step Codemagic/Google Play guide |
| `examples/MONETIZATION-SETUP-TEMPLATE.md` | **NEW v2.5:** Complete monetization setup guide |
| `examples/MONETIZATION-QUICK-REFERENCE.md` | **NEW v2.5:** One-page monetization cheat sheet |
| `examples/iap-products-config.ts` | **NEW v2.5:** IAP products configuration template |

## Build Commands

### Android Debug (unsigned)
```bash
cd android
chmod +x ./gradlew  # Required on macOS/Linux!
./gradlew assembleDebug    # Debug APK (Windows: .\gradlew.bat)
```

### Android Release (signed - for Google Play)

**IMPORTANT**: Get your credentials from `android-signing/CREDENTIALS.txt` before running these commands.

**Windows PowerShell:**
```powershell
# Get the full path to your keystore
$KEYSTORE_PATH = (Resolve-Path android-signing\{{APP_NAME_LOWER}}-release.keystore).Path

# Set environment variables (replace password with actual value from CREDENTIALS.txt)
$env:{{APP_NAME_UPPER}}_KEYSTORE_PATH = $KEYSTORE_PATH
$env:{{APP_NAME_UPPER}}_KEYSTORE_PASSWORD = "YOUR_PASSWORD_FROM_CREDENTIALS"
$env:{{APP_NAME_UPPER}}_KEY_ALIAS = "{{APP_NAME_LOWER}}"
$env:{{APP_NAME_UPPER}}_KEY_PASSWORD = "YOUR_PASSWORD_FROM_CREDENTIALS"

# Build release AAB
cd android
.\gradlew.bat bundleRelease
```

**macOS/Linux:**
```bash
# Set environment variables (replace password with actual value from CREDENTIALS.txt)
export {{APP_NAME_UPPER}}_KEYSTORE_PATH="$(pwd)/android-signing/{{APP_NAME_LOWER}}-release.keystore"
export {{APP_NAME_UPPER}}_KEYSTORE_PASSWORD="YOUR_PASSWORD_FROM_CREDENTIALS"
export {{APP_NAME_UPPER}}_KEY_ALIAS="{{APP_NAME_LOWER}}"
export {{APP_NAME_UPPER}}_KEY_PASSWORD="YOUR_PASSWORD_FROM_CREDENTIALS"

# Build release AAB
cd android
chmod +x ./gradlew  # Don't forget this!
./gradlew bundleRelease
```

**One-liner (Git Bash on Windows/macOS/Linux):**
```bash
cd android && chmod +x ./gradlew && {{APP_NAME_UPPER}}_KEYSTORE_PATH="$(pwd)/../android-signing/{{APP_NAME_LOWER}}-release.keystore" {{APP_NAME_UPPER}}_KEYSTORE_PASSWORD="YOUR_PASSWORD" {{APP_NAME_UPPER}}_KEY_ALIAS="{{APP_NAME_LOWER}}" {{APP_NAME_UPPER}}_KEY_PASSWORD="YOUR_PASSWORD" ./gradlew bundleRelease
```

Output: `android/app/build/outputs/bundle/release/app-release.aab`

### iOS (macOS with Xcode)
```bash
cd ios/App
# Resolve SPM dependencies first!
xcodebuild -resolvePackageDependencies -project App.xcodeproj -scheme App
# Then open in Xcode
open App.xcodeproj  # Use .xcodeproj, NOT .xcworkspace!
```

### iOS (Codemagic)
Push to main branch - builds automatically.

## Pre-Deployment Checklist

Before submitting to app stores, complete these steps:

### Apple App Store

1. **Register iOS Bundle ID in Apple Developer Portal**
   - Go to Certificates, Identifiers & Profiles > Identifiers
   - Click + to add a new identifier
   - Select App IDs, then App
   - Enter description and your iOS Bundle ID: `{{IOS_BUNDLE_ID}}` (e.g., `app.carousel.ios` or `ai.carouselcards.app`)
   - Enable any required capabilities (Push Notifications, In-App Purchase if using RevenueCat, etc.)

2. **Create App in App Store Connect**
   - Go to Apps > + (New App)
   - Select iOS platform
   - Enter app name, iOS bundle ID, and SKU
   - Set primary language

3. **Set Up App Store Connect API Integration in Codemagic**
   - In App Store Connect: Users and Access > Integrations > App Store Connect API
   - Create a new key with **Admin** access (required for creating certificates/profiles)
   - Download the .p8 file and note the Key ID and Issuer ID
   - In Codemagic: Teams > Integrations > App Store Connect
   - Add the key details (the integration name must match `codemagic.yaml`)

4. **Accept All Agreements**
   - Go to App Store Connect > Business section
   - Accept any pending agreements (Paid Apps, etc.)

5. **RevenueCat Setup (if using)**
   - Create app in RevenueCat dashboard
   - Add iOS app with your iOS bundle ID: `{{IOS_BUNDLE_ID}}`
   - Get public API key for iOS
   - Set up products in App Store Connect first
   - Import products into RevenueCat

### Google Play Store

1. **Create App in Google Play Console**
   - Go to All apps > Create app
   - Fill in app details
   - Use your Android package name: `{{ANDROID_PACKAGE_NAME}}`

2. **Set Up Signing**
   - Generate keystore (see Phase 6.5)
   - Consider enrolling in Google Play App Signing

3. **RevenueCat Setup (if using)**
   - Add Android app in RevenueCat with your Android package name: `{{ANDROID_PACKAGE_NAME}}`
   - Get public API key for Android (separate from iOS key)
   - Link Google Play service account for server notifications

**NOTE:** RevenueCat requires separate app entries for iOS and Android, each with their respective bundle identifiers. Make sure to:
- Use iOS bundle ID for the iOS app in RevenueCat
- Use Android package name for the Android app in RevenueCat
- Store both API keys separately in your environment variables

### Environment Variables

Add these to your CI/CD (Codemagic) and local `.env`:

```bash
# RevenueCat (if using)
VITE_REVENUECAT_IOS_API_KEY=your_ios_public_key
VITE_REVENUECAT_ANDROID_API_KEY=your_android_public_key

# Android Signing (for local builds)
{{APP_NAME_UPPER}}_KEYSTORE_PATH=/path/to/keystore
{{APP_NAME_UPPER}}_KEYSTORE_PASSWORD=your_password
{{APP_NAME_UPPER}}_KEY_ALIAS=your_alias
{{APP_NAME_UPPER}}_KEY_PASSWORD=your_key_password
```

## Troubleshooting

### Common Issues

#### iOS: "No such module" or SPM dependency errors
```bash
# Resolve Swift Package Manager dependencies
cd ios/App
xcodebuild -resolvePackageDependencies -project App.xcodeproj -scheme App
```

#### iOS: "Code signing" errors in Codemagic
- Verify App Store Connect API key has **Admin** access (not just Developer)
- Ensure bundle ID is registered in Apple Developer Portal
- Check that integration name in codemagic.yaml matches exactly
- Accept any pending agreements in App Store Connect

#### iOS: Using wrong Xcode project
```bash
# WRONG - Don't use .xcworkspace with Capacitor 8
open App.xcworkspace

# CORRECT - Use .xcodeproj
open App.xcodeproj
```

#### Android: "Permission denied" on gradlew
```bash
chmod +x android/gradlew
```

#### Android: Unsigned APK/AAB rejected
- Ensure signing config is set up in build.gradle
- Set environment variables before building
- Check that keystore file exists and path is correct

#### RevenueCat: "Cannot find module" errors
```bash
# WRONG - This package is NOT compatible with Capacitor 8
npm install @revenuecat/purchases-capacitor-ui

# CORRECT - Only use the base package
npm install @revenuecat/purchases-capacitor
```

#### Node version errors
```bash
# Capacitor 8 requires Node 22+
node --version  # Should be v22.x.x

# Install Node 22 via nvm
nvm install 22
nvm use 22
```

#### Java version errors
```bash
# Capacitor 8 requires Java 21
java --version  # Should be 21.x.x

# Install Java 21
# macOS: brew install openjdk@21
# Windows: Download from https://adoptium.net/
```

### Codemagic-Specific Issues

#### Build fails with "integration not found"
- The `integrations.app_store_connect` value must exactly match your integration name in Codemagic
- Check Teams > Integrations > App Store Connect for the exact name

#### Build fails on code signing
Add explicit code signing commands:
```yaml
- name: Set up code signing
  script: |
    app-store-connect fetch-signing-files "$BUNDLE_ID" --type IOS_APP_STORE --create
    keychain initialize
    keychain add-certificates
    xcode-project use-profiles --project "$XCODE_PROJECT"
```

## Example Usage

User: "I want to wrap my fitness app at https://fitapp.com for the app stores"

Assistant will:
1. Ask for app name, iOS bundle ID, Android package name, colors, RevenueCat preference
2. Explain bundle ID best practices and permanence warning
3. Install all Capacitor dependencies
4. Create configuration for https://fitapp.com with Android package name in capacitor.config.ts
5. Generate icons from existing assets
6. Configure Android with explicit package name in build.gradle and strings.xml
7. Provide instructions to set iOS bundle ID manually in Xcode
8. Set up Codemagic for iOS builds (using iOS bundle ID, SPM, not CocoaPods)
9. Create store listing guide with copy-paste text and correct bundle IDs
10. Provide build commands for the user's platform

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.5 | Jan 2026 | **NEW:** Comprehensive monetization setup guide for RevenueCat + app stores, complete IAP/subscription documentation covering product configuration, App Store Connect setup, Google Play Console setup, RevenueCat integration, testing procedures, and pricing recommendations |
| 2.4 | Jan 2026 | **NEW:** Android feature graphic generator (1024x500 REQUIRED for Play Store), improved screenshot organization (separate phone/tablet-7/tablet-10 directories), updated Play Store requirements documentation with feature graphic as mandatory asset |
| 2.3 | Jan 2026 | **NEW:** Automated screenshot generation for both iOS and Android app stores using Playwright, generates all required screenshot sizes (6.9" iPhone, 13" iPad Pro mandatory for iOS 2026), customizable scenarios for capturing key app screens |
| 2.2 | Jan 2026 | Automated Android keystore generation with secure passwords, added android-signing/ folder structure, CREDENTIALS.txt and SETUP-INSTRUCTIONS.md auto-generation, comprehensive Codemagic/Google Play setup guide |
| 2.1 | Jan 2026 | Added support for separate iOS bundle ID and Android package name, explicit configuration in Xcode and build.gradle, bundle ID best practices documentation |
| 2.0 | Jan 2026 | Capacitor 8 support, SPM instead of CocoaPods, Node 22 requirement, Java 21 requirement, RevenueCat compatibility fixes |
| 1.0 | 2024 | Initial release for Capacitor 7 |

## Notes

- Always verify the web app URL is accessible
- Check for existing PWA manifest to reuse metadata
- Look for existing icons in common locations (/public, /assets)
- Warn about keystore security (never commit to git)
- Recommend App Signing by Google Play
- For iOS, recommend Codemagic if not on macOS
- **Capacitor 8 uses SPM, not CocoaPods** - use `.xcodeproj`, not `.xcworkspace`
- **Only use `@revenuecat/purchases-capacitor`** - the UI package is not Capacitor 8 compatible
- **iOS and Android can have different bundle IDs** - prompt for both separately and configure them independently
- **Bundle IDs are permanent** - warn users that they cannot be changed after app store submission
- **Android package name** must be set in build.gradle (`namespace` and `applicationId`) and strings.xml
- **iOS bundle ID** must be set manually in Xcode after running `npx cap add ios`

---
> Source: [mrchevyceleb/capacitor-wrap-skill](https://github.com/mrchevyceleb/capacitor-wrap-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
