Android build & signing — Quick guide

This repository contains a GitHub Actions workflow that builds a signed Android APK from the web assets using Capacitor.

What I added for you
- `.github/workflows/android-build.yml` — CI workflow that builds web assets, ensures Capacitor config, adds android project if missing, decodes keystore from secrets, and runs Gradle to produce `app-release.apk`.
- A `build` script in `package.json` that copies static web files into `www/` so Capacitor can package them.

Before running the workflow
1) Create a keystore locally (PowerShell / Bash):

PowerShell (Windows):

```powershell
keytool -genkeypair -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias
# convert to base64 for GitHub Secret
[Convert]::ToBase64String([IO.File]::ReadAllBytes('my-release-key.jks')) | Out-File -Encoding ascii keystore.b64.txt
Get-Content keystore.b64.txt  # copy the content
```

Bash (Linux/macOS):

```bash
keytool -genkeypair -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias
base64 my-release-key.jks > keystore.b64.txt
cat keystore.b64.txt
```

2) Add these GitHub repository secrets (Settings → Secrets and variables → Actions → New repository secret):
- `ANDROID_KEYSTORE_BASE64` — paste the base64 content from `keystore.b64.txt`.
- `ANDROID_KEYSTORE_PASSWORD` — the keystore password you used when creating `my-release-key.jks`.
- `ANDROID_KEY_ALIAS` — the key alias (for example `my-key-alias`).
- `ANDROID_KEY_PASSWORD` — password for the key (often same as keystore password).

Triggering the workflow
- Push to `main` or open the Actions tab and manually dispatch the workflow (workflow_dispatch).

What to expect in Actions
- The workflow will run on `ubuntu-latest` and:
  - install Node, Java and Android SDK
  - run `npm run build` (or copy static files into `www/` if build script not present)
  - ensure `capacitor.config.json` exists and run `npx cap add android` if needed
  - decode the keystore and create `android/keystore.properties`
  - run `./gradlew assembleRelease`
  - upload the resulting APK as an artifact named `app-release-apk`

If you don't see an APK
- Open the failing Action run and inspect the logs. Common issues:
  - `npm run build` fails: check your build script or ensure `index.html` and assets exist
  - `npx cap add android` fails: missing dependencies or permission issues — check full log
  - Gradle signing errors: ensure secrets are correct and `keystore` was decoded successfully

I can help further
- I can make CI produce more detailed debug logs (set `--stacktrace` on Gradle) and automatically inject `signingConfigs` into `android/app/build.gradle` after the android project is created. Tell me if you want me to:
  - Add Gradle debug flags to the workflow, or
  - Automatically inject signing config into `android/app/build.gradle` (I will modify that file in CI if needed).

---
If you want, I'll now:
- inject the signing `signingConfigs` snippet automatically into `android/app/build.gradle` after the Android project is created (so signing works out-of-the-box). Reply `inject signing` to proceed.