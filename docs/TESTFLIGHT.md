# Upload an iOS build to TestFlight

The workflow [ios-testflight.yml](../.github/workflows/ios-testflight.yml) builds the standalone
Capacitor app and uploads it only when manually started from **Actions → Upload iOS beta to
TestFlight → Run workflow**. It does not run on every push.

The workflow uses `macos-26`, matching the current shared TestFlight workflow used by the
other mobile app. It builds the mobile flavor, so the resulting app is local-first and does not
connect to the openGym API.

## Required GitHub Actions secrets

Create these in **Settings → Secrets and variables → Actions**. Do not commit their values.

| Secret | Value |
| --- | --- |
| `IOS_TEAM_ID` | Apple Developer Team ID |
| `APP_STORE_CONNECT_KEY_ID` | App Store Connect API key ID |
| `APP_STORE_CONNECT_ISSUER_ID` | App Store Connect API issuer ID |
| `APP_STORE_CONNECT_API_KEY_P8` | Complete contents of the API key `.p8` file |
| `MATCH_GIT_URL` | URL of the encrypted Match repository, for example `https://github.com/NguyenMinhDuc163/apple-signing.git` |
| `MATCH_PASSWORD` | Password that encrypts the Match repository assets |
| `MATCH_GIT_BASIC_AUTHORIZATION` | Base64 of `username:PAT` with read access to `apple-signing` |

The workflow uses `setup_ci` and `match(readonly: true)`: Match creates a temporary keychain,
downloads the Apple Distribution certificate and App Store profile for `com.nguyenduc.gym`, and
removes them with the ephemeral runner. The app repository never stores a `.p12` certificate or
`.mobileprovision` file.

## One-time signing preparation

`apple-signing` is the only repository that may create or renew signing assets. Before the first
openGym upload, register `com.nguyenduc.gym` in Apple Developer, then run **Apple Signing
Maintenance** in that repository with `mode=maintain`, `platform=ios`, `type=appstore`,
`bundle_id=com.nguyenduc.gym`, and `confirm_write=true`. Once it succeeds, this workflow only
reads the resulting assets.

Each workflow run uses its GitHub run number as the iOS build number, so a later TestFlight upload
always has a higher build number. The marketing version is read from `frontend/package.json`.
