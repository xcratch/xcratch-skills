---
name: xcratch-extension-debug-auto
description: 'Autonomously debug an Xcratch extension in the public xcratch.github.io editor by navigating to https://xcratch.github.io/editor/?extension=<extensionURL> and loading the extension from https://0.0.0.0:5500/*. Use when: verifying extension loads, checking console errors, inspecting extension blocks in the Scratch editor UI, reproducing runtime failures with a specific extension URL.'
license: MIT
argument-hint: 'Provide the extension dist file path relative to the workspace root, e.g. dist/xcratchExample.mjs'
user-invocable: true
---

# Debug Extension with Query Parameter

Autonomously load and inspect an Xcratch extension in the public editor at `https://xcratch.github.io/editor/` by appending it as the `?extension=` query parameter. No local `scratch-editor` checkout is needed — only a local HTTPS live server that serves the extension's build output.

## When to Use

- Verify that a built extension loads without errors
- Inspect browser console output after extension load
- Check that extension blocks appear in the Scratch editor toolbox
- Reproduce a runtime failure triggered by loading via URL

## Prerequisites

| Requirement | Detail |
|-------------|--------|
| Live server running | port 5500, serving the extension repo over HTTPS |
| Extension built | `npm run build` or `npm run watch` ran in the extension repo |
| HTTPS certificates | mkcert-generated certs installed in `<extension-repo>/.vscode/` |

If the live server is not running, start it using the `start live server` task in the extension's
VS Code workspace (or run the launch config `debug extension on xcratch.github.io editor`, which
starts it via `preLaunchTask`) before proceeding.

## Procedure

### Step 1 — Resolve the extension URL

The extension is served by the live server at:

```
https://0.0.0.0:5500/<workspace-relative-path-to-dist-file>
```

Example: if the dist file is at `dist/xcratchExample.mjs` (relative to the extension repo root that
live-server serves), the extension URL is:

```
https://0.0.0.0:5500/dist/xcratchExample.mjs
```

Confirm the file exists before navigating.

### Step 2 — Compose the editor URL

Combine the public editor base URL with the extension query parameter:

```
https://xcratch.github.io/editor/?extension=https://0.0.0.0:5500/<path-to-dist-file>
```

### Step 3 — Configure playwright-cli to allow the local HTTPS extension

The public editor (`https://xcratch.github.io`, a public origin) fetches the extension module from
`https://0.0.0.0:5500` (a loopback address). Under playwright-cli this hits **two** browser barriers
that block the fetch and **cannot be clicked away from playwright** (they are native browser UI):

1. **Self-signed certificate** rejection for `https://0.0.0.0:5500`.
2. **Local Network Access (LNA) permission prompt** — Chrome shows "xcratch.github.io が次の許可を
   求めています — このデバイス上の他のアプリやサービスにアクセスする". If not granted, the fetch fails with
   `Permission was denied for this request to access the 'loopback' address space` and the extension
   never loads.

Both are bypassed automatically by launching the browser with a config file. Create
`playwright-cli.json` (auto-loaded from the current directory, or pass it via `--config`):

```json
{
  "browser": {
    "launchOptions": {
      "args": [
        "--disable-features=LocalNetworkAccessChecks,BlockInsecurePrivateNetworkRequests,PrivateNetworkAccessSendPreflights,PrivateNetworkAccessRespectPreflightResults"
      ]
    },
    "contextOptions": {
      "ignoreHTTPSErrors": true
    }
  }
}
```

- `contextOptions.ignoreHTTPSErrors: true` → accept the self-signed cert (no "Advanced → Proceed").
- `launchOptions.args` `--disable-features=...` → disable the Local Network Access / Private Network
  Access prompt so the public→loopback fetch is allowed without the (unclickable) permission dialog.
  Multiple feature names are listed for cross-version safety; harmless extras are ignored.

### Step 4 — Open the browser and navigate

The config must be applied at **browser launch time**, so kill any stale session first, then `open`:

```bash
playwright-cli kill-all
playwright-cli open --headed --config ./playwright-cli.json
playwright-cli goto "https://xcratch.github.io/editor/?extension=https://0.0.0.0:5500/<path-to-dist-file>"
```

If `playwright-cli.json` sits in the current directory it is auto-loaded and `--config` can be
omitted. With this config the extension loads with **no manual clicks** — no cert warning, no LNA
prompt, no reload.

### Step 5 — Wait for the editor to load

Take a snapshot and confirm the Scratch editor UI is visible (stage, toolbox, sprite list):

```bash
playwright-cli snapshot
```

If the editor is still loading, wait and snapshot again.

### Step 6 — Check the browser console for errors

```bash
playwright-cli console
```

Look for:
- `Failed to load extension` messages
- Network errors (CORS, 404, certificate rejection)
- JavaScript exceptions thrown by the extension

### Step 7 — Verify extension blocks appear in the toolbox

Take a screenshot of the full editor UI:

```bash
playwright-cli screenshot
```

Inspect the snapshot for a custom category block corresponding to the extension. If not found, the extension likely failed to register — check the console output from Step 5.

### Step 8 — Collect network diagnostics (if needed)

```bash
playwright-cli network
```

Confirm the `.mjs` dist file was fetched with HTTP 200 from `https://0.0.0.0:5500`.

### Step 9 — Report findings

Summarize:
1. Whether the extension URL loaded successfully (HTTP status)
2. Any console errors and their messages
3. Whether the extension's blocks appeared in the toolbox (with screenshot)
4. Suggested fix if an error was identified

## Common Errors and Fixes

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Certificate warning on `0.0.0.0:5500` | Live server cert not trusted by browser | Launch with the Step 3 config (`contextOptions.ignoreHTTPSErrors: true`); restart the session so it applies |
| Extension never loads; console shows `Permission was denied for this request to access the 'loopback' address space` (no fetch to `0.0.0.0:5500`) | Chrome **Local Network Access** prompt blocking public→loopback; the native dialog can't be clicked from playwright | Launch with the Step 3 config (`launchOptions.args` `--disable-features=LocalNetworkAccessChecks,...`); run `playwright-cli kill-all` then re-`open` so the flag applies |
| 404 on ext URL | Build output missing | Run `npm run build` in extension repo |
| CORS error | Live server not started with `--cors` | Check that `start live server` task is running |
| Extension blocks not shown | Extension threw at registration | Check console for JS errors in extension source |

## Related Skills

- [xcratch-extension-debug](../xcratch-extension-debug/SKILL.md) — full VS Code launch config and source map setup
- [playwright-cli](../../../../../../../../.claude/skills/playwright-cli/SKILL.md) — browser automation commands reference
