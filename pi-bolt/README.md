# Pi-Bolt

A standalone sandbox kit for
[Pi-Bolt](https://github.com/opensec-git/Pi-Bolt), the Pi coding agent
compiled ahead of time to a native executable.

> [!IMPORTANT]
> On Linux, Pi-Bolt currently supports x86-64 only. This kit does not work in
> Linux ARM64 sandboxes, including native ARM64 sandboxes on Apple Silicon.

## Prerequisites

Store an Anthropic API key before creating the sandbox:

```console
sbx secret set anthropic
```

An existing Anthropic OAuth login from a Claude sandbox can be reused instead.

## Usage

Use the published kit artifact:

```console
sbx run --kit "docker.io/sbx/pi-bolt-kit:latest" pi-bolt
```

Or run it from this repository:

```console
sbx run --kit "git+https://github.com/docker/sbx-kits-contrib.git#dir=pi-bolt" pi-bolt
```

Or from a local clone:

```console
sbx run --kit ./pi-bolt/ pi-bolt
```

The first creation downloads and verifies the latest Pi-Bolt release, then
starts its TUI. Subsequent starts reuse that sandbox. Recreate the sandbox to
install a newer release.

## How installation works

The kit runs Pi-Bolt's official installer as the unprivileged `agent` user.
Installation is non-interactive, skips the optional OpenSec extensions, and
places the executable at `/home/agent/.local/bin/pi-bolt`. The setup step
requires OpenSSL 3 so the upstream release signature is checked, and verifies
the installed executable with `pi-bolt --version` before starting the agent.

The shared shell image is used directly rather than building a Pi-Bolt image.
This repository publishes custom images for both amd64 and arm64, while
Pi-Bolt currently has no Linux ARM64 executable.

## How authentication works

The credential contract matches the existing Pi kit because Pi-Bolt uses the
same providers and `~/.pi/agent` configuration:

- Anthropic API keys are represented in the sandbox by a proxy-managed
  sentinel and injected as `x-api-key` only on requests to
  `api.anthropic.com`.
- A Claude subscription OAuth credential is materialized as sentinels in
  Pi's native `~/.pi/agent/auth.json` format. The sandbox proxy substitutes
  the real tokens on inference and refresh requests.

The initial kit supports Anthropic. Other Pi providers need their own narrowly
scoped credential and network declarations before they can be supported under
the sandbox's deny-all policy.

## Network access

The allowlist covers the official installer, signed release downloads, npm
packages and extensions, Anthropic inference, and Anthropic OAuth refresh.
Everything else remains blocked by the sandbox network policy.

## Cleanup

Removing the sandbox removes Pi-Bolt and its sandbox-local state:

```console
sbx rm <sandbox-name>
```
