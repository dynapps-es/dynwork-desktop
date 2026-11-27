# Dynwork desktop

Public builds and downloads of Dynwork, Dynapps' desktop app.

**The code is not here.** It lives in a private GitLab repository. This repository only holds the GitHub Actions
workflows that fetch that code, compile it for each channel and publish the result in
[Releases](https://github.com/dynapps-es/dynwork-desktop/releases).

## Downloads

Dynwork, the stable release. These links always serve the latest one:

| System                     | Download                                                                                                                                              |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| macOS, Apple Silicon (M1+) | [dynwork-desktop-mac-arm64.dmg](https://github.com/dynapps-es/dynwork-desktop/releases/latest/download/dynwork-desktop-mac-arm64.dmg)                 |
| macOS, Intel               | [dynwork-desktop-mac-x64.dmg](https://github.com/dynapps-es/dynwork-desktop/releases/latest/download/dynwork-desktop-mac-x64.dmg)                     |
| Windows 10 and 11, x64     | [dynwork-desktop-win-x64.exe](https://github.com/dynapps-es/dynwork-desktop/releases/latest/download/dynwork-desktop-win-x64.exe)                     |
| Debian and Ubuntu, x64     | [dynwork-desktop-linux-amd64.deb](https://github.com/dynapps-es/dynwork-desktop/releases/latest/download/dynwork-desktop-linux-amd64.deb)             |
| Fedora and openSUSE, x64   | [dynwork-desktop-linux-x86_64.rpm](https://github.com/dynapps-es/dynwork-desktop/releases/latest/download/dynwork-desktop-linux-x86_64.rpm)           |
| Other Linux, x64           | [dynwork-desktop-linux-x86_64.AppImage](https://github.com/dynapps-es/dynwork-desktop/releases/latest/download/dynwork-desktop-linux-x86_64.AppImage) |
| Debian and Ubuntu, ARM64   | [dynwork-desktop-linux-arm64.deb](https://github.com/dynapps-es/dynwork-desktop/releases/latest/download/dynwork-desktop-linux-arm64.deb)             |
| Fedora and openSUSE, ARM64 | [dynwork-desktop-linux-aarch64.rpm](https://github.com/dynapps-es/dynwork-desktop/releases/latest/download/dynwork-desktop-linux-aarch64.rpm)         |
| Other Linux, ARM64         | [dynwork-desktop-linux-arm64.AppImage](https://github.com/dynapps-es/dynwork-desktop/releases/latest/download/dynwork-desktop-linux-arm64.AppImage)   |

The first stable release published here is the one after `0.6.0`; until it exists these links answer 404, and
`0.6.0` and earlier are in [dynapps-es/dynwork](https://github.com/dynapps-es/dynwork/releases/latest). Each release's
notes say how to open the app past the system's warning: the builds are not signed by Apple or Microsoft yet.

Versions have no `v` prefix: `0.6.0` is a stable release, `0.6.0-beta.1` a beta.

### Internal channels

For the team that develops Dynwork, not for work:

- **Canary** (`dynwork-canary`, a prerelease replaced on every merge to `main`): Dynwork Canary, which installs beside
  Dynwork with its own accounts and data, built for every platform, for example macOS Apple Silicon:
  [dynwork-canary-desktop-mac-arm64.dmg](https://github.com/dynapps-es/dynwork-desktop/releases/download/dynwork-canary/dynwork-canary-desktop-mac-arm64.dmg).
- **Beta** (`X.Y.Z-beta.N`, plus the prerelease `dynwork-beta` that always holds the newest beta or stable): the stable
  app at a beta version, with the same file names as the stable release. Beta installs update from `dynwork-beta`, so
  they receive the next beta and the next stable, and never an older version.

Each channel's release carries the files its updater reads: `latest.yml` (Windows), `latest-linux.yml` (Linux x64) and
`latest-linux-arm64.yml` (Linux ARM64). Windows and the Linux AppImage update themselves; macOS does not yet, since the
builds are not signed.

Prereleases are never the latest release: the last job of every canary and beta build fails if one is.

## How a build starts

The GitLab pipeline dispatches `.github/workflows/build.yml` once its checks pass:

| Ref in GitLab        | `kind`   | Publishes                                                                 |
| -------------------- | -------- | ------------------------------------------------------------------------- |
| `main`               | `canary` | the `dynwork-canary` prerelease, all five targets                         |
| a tag `X.Y.Z-beta.N` | `beta`   | the prerelease `X.Y.Z-beta.N` and `dynwork-beta`, all five targets        |
| a tag `X.Y.Z`        | `stable` | the release `X.Y.Z`, marked latest, then `dynwork-beta`, all five targets |

A build can also be started by hand from `main`, for example to try a branch on every platform without publishing it
(`kind=test` uploads the installers as workflow artifacts kept 3 days, and publishes nothing):

```bash
gh workflow run build.yml -R dynapps-es/dynwork-desktop \
  -f kind=test -f ref=<branch> -f sha=<full commit sha> -f targets='["mac-arm64","windows-x64","linux-x64"]'
```

The inputs are `kind` (`canary`, `beta`, `stable` or `test`), `ref` (the GitLab branch or tag; for a beta or stable the
tag itself), `sha` (the full commit, which must be in GitLab), `targets` (a JSON list of `mac-arm64`, `mac-x64`,
`windows-x64`, `linux-x64` and `linux-arm64`; empty is all five, and a stable always builds all five) and `version` (the tag of a beta or stable; empty for canary and test). Each run is named
`<kind> <ref> <sha>`.

What a build checks before packing:

- the commit fetched is `sha`, and for a beta or stable the GitLab tag `ref` still points at it;
- a stable's version is the one in `dynwork/packages/branding/src/identity.ts` and `dynwork/releases/<version>.md`
  exists (its notes); a beta's is `<that version>-beta.N`;
- a canary's version is `bun dynwork/script/canary.ts version <run number + CANARY_RUN_OFFSET>`.

## Settings this repository relies on

- Environment `release`, deployable from `main` only. Every job runs in it. It holds the GitLab deploy token
  (`GITLAB_DEPLOY_USER`, `GITLAB_DEPLOY_TOKEN`, read-only on the code) and, once signing exists, the signing secrets.
- Repository variables: `CANARY_RUN_OFFSET` (`1000`: run numbers restarted here, and canary versions must keep
  climbing past `0.5.1-canary.85`), `MAC_SIGNING` and `WINDOWS_SIGNING` (`false`).
- With `MAC_SIGNING=true` the macOS packs are signed with the Developer ID and notarized; they need the `release`
  secrets `CSC_LINK` (the `.p12`, base64), `CSC_KEY_PASSWORD`, `APPLE_API_KEY_P8` (the App Store Connect `.p8`,
  base64), `APPLE_API_KEY_ID` and `APPLE_API_ISSUER`. With `WINDOWS_SIGNING=true` the Windows pack is signed with
  Azure Artifact Signing; it needs `AZURE_SIGNING_ENDPOINT`, `AZURE_SIGNING_PROFILE`, `AZURE_SIGNING_ACCOUNT`,
  `AZURE_SIGNING_PUBLISHER`, `AZURE_TENANT_ID`, `AZURE_CLIENT_ID` and `AZURE_CLIENT_SECRET`. Only the signing step of
  that platform sees them.
- `GITHUB_TOKEN` is read-only by default; only the publish job gets `contents: write`, and it runs no code from GitLab.

## Contributing

Do not open pull requests with code here: they cannot change Dynwork, which is built from GitLab. Only the build
workflows live in this repository, and only the Dynapps AI leads change them.
