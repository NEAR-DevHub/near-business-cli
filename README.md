# Near Business CLI

Manage your confidential [Near Business](https://business.near.com) treasuries from the command line: view assets, members and activity, create payment requests, and vote on requests.

## Install

macOS and Linux:

```sh
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/NEAR-DevHub/near-business-cli/releases/latest/download/near-business-cli-installer.sh | sh
```

Windows (PowerShell):

```powershell
powershell -ExecutionPolicy Bypass -c "irm https://github.com/NEAR-DevHub/near-business-cli/releases/latest/download/near-business-cli-installer.ps1 | iex"
```

npm:

```sh
npm install -g near-business-cli
```

From source with Cargo (on Linux, install `libudev-dev` first for Ledger support):

```sh
cargo install near-business-cli
```

## Usage

```sh
near-business auth login
near-business treasury list
```

Run `near-business` with no arguments for an interactive menu, or `near-business --help` for all commands.

## Releasing

Releases are automated with [release-plz](https://release-plz.dev) and [dist](https://opensource.axo.dev/cargo-dist/):

1. Every push to `main` opens or updates a release PR with the next version and changelog.
2. Merging that PR publishes the crate to crates.io and tags `vX.Y.Z`.
3. The tag's release builds prebuilt binaries for macOS, Linux and Windows, creates the GitHub Release with the shell, PowerShell and MSI installers, and publishes the npm package.

The repository needs these secrets:

| Secret | Used for |
|---|---|
| `CARGO_REGISTRY_TOKEN` | Publishing to crates.io |
| `NPM_TOKEN` | Publishing the npm package |
| `RELEASE_PLZ_TOKEN` (optional) | Opening the release PR so that CI runs on it |

## License

Licensed under either of [Apache License, Version 2.0](LICENSE-APACHE) or [MIT license](LICENSE-MIT) at your option.
