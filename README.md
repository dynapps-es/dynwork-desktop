# Dynwork desktop

Public builds and downloads of Dynwork, Dynapps' desktop app.

**The code is not here.** It lives in a private GitLab repository. This repository only holds the GitHub Actions
workflows that fetch that code, compile it for each channel and publish the result in
[Releases](https://github.com/dynapps-es/dynwork-desktop/releases).

## Downloads

Download links per channel (stable, beta and canary) are added with the build workflows.

Versions have no `v` prefix: `0.6.0` is a stable release, `0.6.0-beta.1` a beta.

## Contributing

Do not open pull requests with code here: they cannot change Dynwork, which is built from GitLab. Only the build
workflows live in this repository, and only the Dynapps AI leads change them.
