---
title: "Keeping Secrets Out of .mcp.json: A Credential Manager Wrapper Pattern for Windows"
date: 2026-09-16
summary: "Why environment-variable substitution doesn't work for MCP server configs on Windows, and how a small PowerShell wrapper backed by Windows Credential Manager solves it for both stdio and HTTP servers."
description: "A walkthrough of storing MCP server secrets in Windows Credential Manager instead of plaintext config files, using PowerShell wrapper scripts to inject them at launch time for both stdio and HTTP-based MCP servers."
slug: "securing-mcp-server-secrets-on-windows"
draft: false
aliases:
  - "/securing-mcp-server-secrets-on-windows"
categories:
  - "Security"
  - "Automation"
tags:
  - "MCP"
  - "PowerShell"
  - "Windows Credential Manager"
  - "Secrets Management"
  - "DPAPI"
---

# Introduction

I've been wiring up a handful of local MCP (Model Context Protocol) servers - things like a network controller integration, a NAS API, and a home automation bridge - and every one of them needs a secret: an API key, a password, or a bearer token. The config file that wires these servers up, `.mcp.json`, lives in a folder that syncs to the cloud. That's a problem, because the obvious way to keep secrets out of a synced file - `${VAR}` placeholders resolved from an environment variable - simply doesn't work the way you'd expect.

This post covers why that substitution fails, and the wrapper-script pattern I ended up building instead, backed by Windows Credential Manager rather than plaintext environment variables.

## Why `${VAR}` substitution doesn't work

The obvious fix for "don't put secrets in a synced JSON file" is to reference an environment variable and let something expand it at load time. That assumption breaks down here for a few reasons:

- There's no `.env` file loading in the picture.
- The `env` block in the tool's own settings only affects its own subprocesses - it isn't consulted when the MCP loader resolves a server config.
- For **stdio-type** servers (the ones defined with a `command`), placeholders like `${VAR}` or `${env:VAR}` inside the config's `env` block are not expanded at all.
- For **HTTP-type** servers (defined with a `url`), the `url` field is validated as a literal URL string *before* any substitution could happen. A placeholder fails validation outright with something like `'url' is not a valid URL`.

What *does* work for stdio servers: the spawned subprocess inherits the full OS environment of whatever launched it, merged with the `env` block in the config. So a secret set as a real Windows user environment variable does reach the subprocess - as long as the config's `env` block doesn't also define that same key, since an explicit entry there takes priority over the inherited one.

That's a real mechanism, but it comes with a real cost.

## Why a plaintext user environment variable isn't good enough

A permanent Windows user environment variable is:

- **Plaintext at rest**, sitting in `HKCU\Environment` in the registry.
- **Inherited by every process you launch afterward**, forever, until someone manually removes it - any script or app running as you can read it.

**Windows Credential Manager** is a meaningful step up: entries are DPAPI-encrypted at rest, and only readable by something that explicitly calls the Credential Manager API for that specific entry. It doesn't get silently inherited by every child process the way an environment variable does.

So the design I landed on is:

- Secrets live only in Windows Credential Manager, never in the config file and never as a standing environment variable.
- Each MCP server that needs a secret is launched through a small PowerShell **wrapper script** instead of being invoked directly.
- The wrapper retrieves the secret from Credential Manager at launch, sets it as an environment variable scoped to *just that one subprocess*, then execs the real MCP server - inheriting stdio so the protocol still flows through normally.
- HTTP-type servers get converted to stdio too, bridged locally via [`mcp-remote`](https://www.npmjs.com/package/mcp-remote), so the same wrapper pattern covers them. This is the only way to keep a URL-embedded token (or a header-based bearer token) out of the config for a remote HTTP MCP server, since there's no injection point for the `url` field otherwise.

## What actually changes: scope and lifetime, not "is it an env var"

The wrapper still sets a real environment variable before launching the real MCP process - that part is unavoidable, since the underlying tools only accept secrets via env vars or CLI arguments. What changes is *scope and lifetime*:

| | Old (permanent user env var) | New (wrapper-set env var) |
|---|---|---|
| Where stored at rest | Registry, plaintext | Credential Manager, DPAPI-encrypted |
| Who inherits it | Every process you launch afterward, forever, until manually removed | Only that one child process (and anything it spawns) |
| Lifetime | Persists across reboots until deleted | Exists only in that process's memory; gone when it exits |
| Readable via a simple registry/env dump from any script | Yes | No - never touches that store |

A permanent user environment variable is readable by *any* process running as you, at any time - a browser extension, a compromised npm postinstall script, anything. The wrapper approach narrows that down to: only readable by calling the Credential Manager API for that specific target, or by inspecting the live memory/environment block of that one subprocess while it happens to be running. Not perfect secrecy, but a much narrower blast radius than "permanent and global."

## The pieces

A small `CredHelper.ps1` wraps `advapi32.dll`'s Credential Manager API via P/Invoke, with no external dependencies - just three functions: `Set-GenericCredential`, `Get-GenericCredentialPassword`, and `Remove-GenericCredential`.

Each MCP server that needs a secret gets its own tiny wrapper script that:

1. Dot-sources the helper.
2. Pulls the secret out of Credential Manager.
3. Sets it as an environment variable for the current process only.
4. Execs the real command.

For example, a stdio server that needs an API key:

```powershell
$ErrorActionPreference = 'Stop'
. "$PSScriptRoot\CredHelper.ps1"
$env:THE_SECRET_ENV_VAR_NAME = Get-GenericCredentialPassword -Target 'MCP-ServiceName'
& <original command> <original args>
exit $LASTEXITCODE
```

And for an HTTP server, bridged to stdio via `mcp-remote`:

```powershell
$ErrorActionPreference = 'Stop'
. "$PSScriptRoot\CredHelper.ps1"
$url = Get-GenericCredentialPassword -Target 'MCP-ServiceName'
& npx -y mcp-remote@latest $url --allow-http   # drop --allow-http for HTTPS endpoints
exit $LASTEXITCODE
```

Credential Manager entries show up as generic credentials, visible (but not readable) via `cmdkey /list` or Control Panel → Credential Manager → Windows Credentials.

## Wiring it into the config

In `.mcp.json`, the server entry just points at the wrapper instead of the real command:

```json
"command": "powershell.exe",
"args": ["-NoProfile", "-File", "C:\\path\\to\\wrappers\\<name>-wrapper.ps1"]
```

Non-secret config - hostnames, flags, and the like - stays in the config file's `env` block as plaintext, since none of that is sensitive. It still gets merged in before the wrapper runs.

For an HTTP-type server, the `"type": "http"` / `"url"` entry gets replaced entirely with a stdio-style entry pointing at the wrapper, following the same pattern.

One thing worth flagging if you try this yourself: don't add `-ExecutionPolicy Bypass` to the wrapper invocation. It's unnecessary if your local machine policy is already `RemoteSigned` (check with `Get-ExecutionPolicy -List`), which permits local unsigned scripts to run fine, and `Bypass` tends to trip security tooling for no benefit.

## Access boundary: it's per-user-account, not per-privilege-level

It's worth being honest about what this setup does and doesn't protect against. No UAC prompt fires when a wrapper reads a credential, and that's by design, not a gap: Credential Manager's generic credentials are DPAPI-encrypted using a key derived from your Windows logon, not gated by admin rights. Any process running **as your user account** - elevated or not - can read them with no prompt. This is the same mechanism that lets browsers and other tools silently reuse saved logins.

So the real boundary is "something running as you" versus "something not," not "admin" versus "non-admin." An unelevated script, a compromised npm postinstall hook, a browser extension - anything running under your account has the same access to a stored credential as an elevated PowerShell window would. What this buys you over a plaintext environment variable is that extracting the secret requires code that specifically knows to call the Credential Manager API for that target, rather than a passive dump of the registry or a config file. That's a real but narrow improvement, not a hard wall against a targeted attacker already running as you.

That narrowness is fine, because it isn't the goal. The actual aim of this whole exercise is much simpler: get secrets out of a plaintext file that syncs to the cloud and into a store built for the job. Solving "what if my account is already compromised" is a different, harder problem, and no single control - this one included - solves it alone. That's what defense in depth is for. In practice, that means scoping and restricting each MCP server's credential to the minimum it actually needs: a read-only key instead of an admin one, a token limited to a single integration instead of a shared account, and rotation on a real schedule instead of "whenever I remember." Do that, and even a worst-case extraction is a narrow, low-value blast radius rather than a master key to everything.

## Possible future enhancement: gating access with Windows Hello

Everything above assumes the threat is malware running under your own account, and it doesn't fully solve even that. A different threat - someone else picking up your unlocked machine - calls for a different defense. If that's a scenario worth guarding against, there are two tiers of Windows Hello integration worth considering, increasing in strength and cost:

**Tier 1 - UI consent gate.** The wrapper calls `Windows.Security.Credentials.UI.UserConsentVerifier` before reading the credential, triggering a real fingerprint/face/PIN prompt. Cheap to add - roughly a dozen lines of WinRT projection from PowerShell. But it's a soft gate: the underlying Credential Manager entry is unchanged, so it only stops *that wrapper's* flow. A different process could still read the same target directly and skip the prompt, since the secret's encryption doesn't actually depend on Hello succeeding.

**Tier 2 - cryptographically bound to Hello.** Generate a TPM-backed key pair against the Passport Key Storage Provider, requiring a Hello gesture to unlock. Encrypt the real secret with that key's public half, so the ciphertext is safe to store anywhere; decrypting it calls into the private key handle, which the OS itself blocks until Hello succeeds - no code path can skip it, because the key material genuinely isn't usable without the gesture. This is the same mechanism behind Windows Hello for Business certificates. Real protection, but meaningfully more work.

Either tier adds a prompt on every wrapper launch - every editor start, every dropped-connection auto-reconnect, every window reload that respawns an MCP server. That's real friction against something that currently reconnects silently in the background, so it's only worth it if the threat model actually calls for it.

## Rotating or removing a credential

Rotation doesn't touch the config or the wrapper at all - the wrapper reads the current value at every launch:

```powershell
. C:\path\to\wrappers\CredHelper.ps1
Set-GenericCredential -Target 'MCP-ServiceName' -Username 'mcp' -Secret '<new secret>'   # overwrite
Remove-GenericCredential -Target 'MCP-ServiceName'                                        # delete
```

## Final thoughts

None of this is exotic cryptography - it's DPAPI, which has been part of Windows for over two decades, wired up through a small P/Invoke helper. But it closes a real gap: a config file that syncs to the cloud no longer needs to carry plaintext secrets, and a compromise of that file (or the sync service it flows through) doesn't hand over credentials for every service you've connected. If you're wiring up MCP servers on Windows and hit the same `${VAR}` substitution wall, this pattern is worth the twenty minutes it takes to set up.
