---
title: Changelog
description: Changes to the DevBridge documentation site.
---

# Changelog

This file records changes to the DevBridge documentation site.

## 2026-09-21

### CLI changes

#### Changed

##### Cluster and gateway

- The production build's gateway address, SNI domain, and cluster identifier switched from `cn-north-4-bridge.myhuaweicloud.com` to `devbridge-s2.hwtunnel.com`.
- Added an injectable cluster identifier `ClusterID` (default `devbridge-s2`), sent with the request when creating a tunnel and configurable via ldflags.

## 2026-08-31

### CLI changes

#### Changed

##### Authentication

- Login migrated from AK/SK to API keys.
- `devbridge auth login` gained a `--api-key` flag to pass an API key directly.
- Removed the `--access-key`, `--secret-key`, and `--security-token` flags.
- Added support for the `HW_API_KEY` environment variable, read automatically by the CLI.
- Adapted the browser login callback to the `{error_code, error_msg, result}` wrapper format, so failure errors match the API client's format.
- On login failure, include the API key management page URL, prompting the user to delete and retry.
- Credential storage now uses the keyring and falls back to a config file; removed the expiration logic.

##### Host / Connect

- `host` and `connect` gained `--api-key` / `-k` flags to authenticate with an API key and skip token issuance.
- `host` and `connect` support the default tunnel configured with `set`; when no tunnel ID is given, the default is used.
- `connect --token` mode requires an explicit tunnel ID and does not use the default tunnel.

##### Tunnel details

- `devbridge show` now includes Host connection count, Connect connection count, and upload and download traffic statistics.

#### Removed

- Removed the `-j` / JSON output flag from `list` and `port list`.
- Removed the `--huaweicloud` flag and the `loginType` parameter.
- Removed the development self-signed certificate.

### Server changes

#### Added

##### API key management

- Added creating, viewing, and deleting API keys.
- API keys are distinguished by use case (DevBridge, DevBox), with up to 20 per case.
- The full API key value is shown only at creation; the list shows a masked value and last-used time, and a key is invalidated immediately upon deletion.

### Build and release

#### Changed

##### CI pipeline

- Added a `build-cli.yml` workflow supporting manual builds and automatic PR build validation.
- PR-triggered runs only compile-validate (`goreleaser build --snapshot`) and don't publish a release.
- A release is published to GitHub Release and GitCode Release only when the version includes `release`.

##### GoReleaser

- The build tooling migrated from hand-written cross-platform scripts to GoReleaser, unifying cross-compilation for 6 platforms, version injection, SHA256 checksums, and tar.gz packaging.
- The baked install script (`bake-install.sh`) runs as a GoReleaser `before.hook` to inject the version and release download URL automatically.
- Removed the deprecated hand-written `scripts/build.sh` build script.

##### GitCode Release

- Build artifacts are mirrored to GitCode Release with a rolling `latest` tag.
- Uploads use a two-step API (`upload_url` + `PUT`) with added timeout and retry logic.

##### Install script

- Simplified to a remote-only mode with multi-source auto-detection (GitHub / GitCode / OBS).
- Each channel bakes its own download URL, with no cross-source fallback.

## 2026-08-11

### CLI changes

#### Added

##### Quota query

- Added the `devbridge limits` command to view account quota and current usage.
- The output includes: reset time, traffic quota and traffic used, the number of active tunnels, limits on tunnels/ports/Hosts, tunnel bandwidth limit, per-port HTTP request-rate limit, and per-port connection limit.

##### Debug tools

- Added the `devbridge echo` command to start an HTTP echo service for verifying the tunnel link.
  - Random port by default, listening on `127.0.0.1`.
  - `-p` specifies the port and `-i` the listening address.
- Added the `devbridge ping` command to send HTTP ping probes to a URI.
  - Default interval 1000 ms; `-i` sets the interval in milliseconds.
  - Outputs the HTTP status code and latency, e.g. `HTTP 200 OK -- 4 ms`.

##### Debug logging

- Added a global `-v` / `--verbose` flag to enable debug-level logging.
- Applies to all commands, e.g. `devbridge -v host <tunnelId>`, `devbridge connect <tunnelId> --verbose`.

## 2026-07-31

### CLI changes

#### Added

#### Installation

- Supports installation via Bash and PowerShell 5.1 or later.
- The CLI installs to `~/.huawei/bin` by default, with configuration stored in `~/.huawei/devbridge`.
- Supports x86-64 and ARM64 architectures.

#### Authentication

- Supports interactive login via `devbridge auth login`.
- Supports AK/SK login via `--access-key` and `--secret-key`.
- Supports temporary-credential login via the additional `--security-token`.
- Provides `devbridge auth status` to view login status and `devbridge auth logout` to log out.

#### Tunnels

- The tunnel ID is an 8-character lowercase Base32 string.
- When creating a tunnel you can specify name, description, and expiration (in hours), default 72, maximum 720.
- A workspace holds up to 10 valid tunnels by default.
- Supports `create`, `list`, `show`, `update`, `delete`, and `delete-all` commands.
- Supports managing the local default tunnel via `set` and `unset`.
- Supports issuing Host or Connect tokens via the `token` command; tokens have a fixed short validity period.

#### Ports

- Ports range from `1` to `65535` and must not repeat within the same tunnel.
- Protocols: `http`, `https`, and `auto`, default `auto`.
- Anonymous-access policy is controlled via `--allow-anonymous` and `--deny-anonymous`.
- Supports `port create`, `port list`, `port show`, `port update`, and `port delete` commands.
- The current CLI doesn't support changing an existing port's protocol directly; delete and recreate instead.

#### Host

- Supports hosting an existing tunnel's ports, or creating a temporary tunnel and hosting immediately.
- Supports hosting multiple local ports at once.
- Automatically reconnects after brief network interruptions.
- Persistent tunnels and port configuration are retained after Host stops.

#### Connect

- Sets up local port mappings via `devbridge connect <tunnelId>`.
- Automatically reads the tunnel's port configuration and sets up the corresponding mappings.
- Protected ports require a valid identity and Connect token, requested automatically by the CLI.
- Automatically reconnects after brief network interruptions.

#### Configuration and directories

- The executable and user state are kept separately in `~/.huawei/bin` and `~/.huawei/devbridge`.
- Credentials are stored in protected local storage and should not be copied, committed, or printed.
- When migrating devices, log in again on the new device rather than copying the configuration directory.

#### Troubleshooting

- Covers command-not-found, install-script download failure, login failure, tunnel not found, quota limit, Host can't connect, Connect can't access, and incorrect HTTP/HTTPS behavior.
- Provides a non-sensitive information checklist for reporting issues.

### Server changes

#### REST API

- The service base URL is `https://hdspace-partner.cn-north-4.myhuaweicloud.com/open-api-public/v1/relay`.
- Responses are wrapped in `error_code`, `error_msg`, and `result`, with success code `0000`.
- Provides full CRUD endpoints for tunnels and ports, plus a tunnel-token issuance endpoint.
- Defines error codes such as `HD.98310001`, `HD.98300005`, and `HD.98320078`, with handling suggestions.
