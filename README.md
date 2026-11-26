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

After each interactive run the CLI prints the equivalent one-line command ("Your console command: …"), so you can reuse it in scripts.

## Using from scripts and AI agents

Scripts and AI agents usually run without a terminal (no TTY). There the CLI cannot ask questions, so any missing argument fails with `The input device is not a TTY`. Always pass the whole command.

### Sign in

The account must exist on chain with a full-access key. With a key file written by [NEAR CLI](https://github.com/near/near-cli-rs):

```sh
near-business auth login <account-id> sign-with-access-key-file ~/.near-credentials/implicit/<account-id>.json
near-business auth whoami
```

### Read

```sh
near-business treasury list
near-business treasury info <treasury>
near-business assets <treasury>
near-business members <treasury>
near-business activity <treasury>
near-business address-book <treasury> list
near-business requests <treasury> pending
near-business requests <treasury> list --status InProgress   # or Approved, Rejected, Expired, Failed
near-business requests <treasury> view <request-id>
```

`<treasury>` is the treasury's account id, e.g. `acme.sputnik-dao.near`.

### Create and vote on requests

Commands that change the treasury do not move funds directly. They create a request (a DAO proposal) or vote on one, and nothing happens until enough approvers approve it. Every such command ends with the same tail:

```
network-config mainnet sign-with-access-key-file <key-file> send
```

The CLI builds the DAO proposal itself and Near Business relays it and pays the gas, so there is no gas or deposit to choose. Use exactly this tail. Other options NEAR CLI offers at that point, like `submit-as-dao-proposal` or `sign-later`, do not apply.

Pay someone:

```sh
near-business payments <treasury> send <token> details <amount> <destination-network> <recipient> '<description>' \
  network-config mainnet sign-with-access-key-file <key-file> send
```

- `<token>`: a symbol from `near-business assets <treasury>`, e.g. `NEAR` or `USDC`. If the treasury holds the same symbol in several forms, add the form: `NEAR@near`, `USDT@intents`, `USDT@ft`. A token contract id also works.
- `<amount>`: what the recipient receives, in whole tokens (`0.1`, `250`).
- `<destination-network>` and `<recipient>`:
  - `near.com`: the recipient's NEAR Intents (near.com) balance. Write the recipient as `nearcom:<account>`, e.g. `nearcom:alice.near`.
  - `near`: a NEAR account, e.g. `alice.near`.
  - another chain: its bridge asset id, e.g. `nep141:eth.omft.near`, with an address on that chain.
- `<description>`: a short memo shown to approvers.

Example, 0.1 NEAR to `frol.near`'s near.com balance:

```sh
near-business payments acme.sputnik-dao.near send NEAR details 0.1 near.com nearcom:frol.near 'Test payment' \
  network-config mainnet sign-with-access-key-file ~/.near-credentials/implicit/<account-id>.json send
```

On success the CLI prints `Delegate action relayed successfully!` and a `View proposal: <url>` link to the new request.

Vote:

```sh
near-business requests <treasury> approve <request-id> network-config mainnet sign-with-access-key-file <key-file> send
near-business requests <treasury> reject <request-id> network-config mainnet sign-with-access-key-file <key-file> send
```

### Troubleshooting

- `The input device is not a TTY`: an argument is missing. Compare the command with the forms above, or run `near-business <command> --help`.
- `Not logged in`: run `near-business auth login` again.
- Add `--teach-me` before the command (`near-business --teach-me assets <treasury>`) for verbose output.

## Releasing

Releases are automated with [release-plz](https://release-plz.dev) and [dist](https://opensource.axo.dev/cargo-dist/):

1. Every push to `main` opens or updates a release PR with the next version and changelog.
2. Merging that PR publishes the crate to crates.io and tags `vX.Y.Z`.
3. The tag's release builds prebuilt binaries for macOS, Linux and Windows, creates the GitHub Release with the shell, PowerShell and MSI installers, and publishes the npm package.

The repository needs these secrets:

| Secret | Used for |
|---|---|
| `CARGO_REGISTRY_TOKEN` | Publishing to crates.io |
| `NPM_TOKEN` | The first npm publish only (see below) |
| `RELEASE_PLZ_TOKEN` (optional) | Opening the release PR so that CI runs on it |

After the first release, set up [npm trusted publishing](https://docs.npmjs.com/trusted-publishers) for `near-business-cli` on npmjs.com, with GitHub Actions, repository `NEAR-DevHub/near-business-cli` and these workflow filenames:

- `release-plz.yml`, for automatic releases (npm checks the calling workflow).
- `release.yml`, for releases started by hand from the Release workflow.

Then delete the `NPM_TOKEN` secret.

## License

Licensed under either of [Apache License, Version 2.0](LICENSE-APACHE) or [MIT license](LICENSE-MIT) at your option.
