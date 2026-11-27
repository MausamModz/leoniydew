# build from GitHub Actions (Android APK, iOS IPA, Windows EXE) — just no-sounds

Workflow: `.github/workflows/build.yml`

| Platform | Output file | Where will you meet?
|---|---|---|
| Android | `leoniydew-android-<version>-arm-base.apk` | Actions artifact `android-apk` |
| iOS | `leoniydew-ios-<version>-no-sounds.ipa` (unsigned) | Actions artifact `ios-ipa` |
| Windows | `leoniydew-<tag>-windows-x86_64-no-sounds.zip` (andar `leoniydew-native.exe`) | Actions artifact `windows-zip` |

Sounds APK/IPA/ZIP does not run. The app downloads them from the `sounds-<version>` release repo.
(`sounds/release.txt` = `0.1.4`, i.e. release tag `sounds-0.1.4` should be in your repo).

## Setup (one time)

1. Create a new repository on GitHub (name: `leoniydew`). If name/owner is different from `MausamModz/leoniydew` then 
In `src/update.rs` replace the URLs with `MausamModz/leoniydew` with your username/repo.
2. Push this poor folder to the repo (hidden `.github` folder must be removed).
3. Repo > **Settings > Actions > General > Workflow permissions** > **Read and write permissions** > Save.
4. (Optional but recommended) Android signing secrets, ensuring every build is signed with the same key so that updates can be installed:
In Termux:
```
pkg install openjdk-21
keytool -genkeypair -keystore leoniydew-release.jks -alias leoniydew -keyalg RSA -keysize 2048 -validity 10000
base64 -w0 leoniydew-release.jks
```
In your repository, go to **Settings > Secrets and variables > Actions > New repository secret** and create the following 4 secrets:
- `ANDROID_KEYSTORE_BASE64` = the full output from the `base64` command above
- `ANDROID_KEYSTORE_PASSWORD` = your keystore password
- `ANDROID_KEY_ALIAS` = `leoniydew`
- `ANDROID_KEY_PASSWORD` = your key password (in PKCS12, this is often the same as the keystore password)

If these secrets are not provided, the workflow will generate a temporary key for that specific run (the APK will install, but you won't be able to install subsequent builds as updates; you would have to uninstall the existing app first). **Never commit the `.jks` file to the repository.**

## Running a Build

**Manual:** In your repository, go to **Actions > Build (Android, iOS, Windows) > Run workflow**. This takes 30–60 minutes (longer the first time).
Once finished, open the run page and download the 3 files from the **Artifacts** section at the bottom.

**For a Release:** Push a tag; the files will then be attached to a GitHub Release:
```
git tag v0.5.5
git push origin v0.5.5
```

## Notes

- The iOS IPA is unsigned; install it using AltStore, SideStore, or Sideloadly with your own Apple ID. - If you need an AVX2 build for Windows, add `env: RUSTFLAGS: -Ctarget-cpu=x86-64-v3` to the `cargo build` step of the Windows job in `build.yml`.
- The `aapt2FromMavenOverride` line (intended for Termux) has been removed from `android/gradle.properties` (as that path does not exist on GitHub runners, causing the build to fail). If you are performing a local build on Termux, add this line to `~/.gradle/gradle.properties` instead.
