---
name: xcratch-extension-debug-auto
description: 'Autonomously debug an Xcratch extension in the scratch-editor dev-server by navigating to https://localhost:8601/?extension=<extensionURL> and loading the extension from https://0.0.0.0:5500/*. Use when: verifying extension loads, checking console errors, inspecting extension blocks in the Scratch editor UI, reproducing runtime failures with a specific extension URL.'
argument-hint: 'Provide the extension dist file path relative to the workspace root, e.g. xcx-my-extension/dist/myExtension.mjs'
user-invocable: true
---

# Debug Extension with Query Parameter

Autonomously load and inspect an Xcratch extension in the scratch-editor by appending it as the `?extension=` query parameter.

## When to Use

- Verify that a built extension loads without errors
- Inspect browser console output after extension load
- Check that extension blocks appear in the Scratch editor toolbox
- Reproduce a runtime failure triggered by loading via URL

## Prerequisites

| Requirement | Detail |
|-------------|--------|
| Dev servers running | port 8601 (scratch-gui) and port 5500 (live server) |
| Extension built | `npm run build` or `npm run watch` ran in the extension repo |
| HTTPS certificates | mkcert-generated certs installed in `scratch-editor/.vscode/` |

If the dev servers are not running, start them using the `start debug servers` task in the scratch-editor workspace before proceeding.

## Procedure

### Step 1 — Resolve the extension URL

The extension is served by the live server at:

```
https://0.0.0.0:5500/<workspace-relative-path-to-dist-file>
```

Example: if the dist file is at `xcx-my-extension/dist/myExtension.mjs` (relative to the parent that live-server serves), the extension URL is:

```
https://0.0.0.0:5500/xcx-my-extension/dist/myExtension.mjs
```

Confirm the file exists before navigating.

### Step 2 — Compose the editor URL

Combine the dev server base URL with the extension query parameter:

```
https://localhost:8601/?extension=https://0.0.0.0:5500/<path-to-dist-file>
```

### Step 3 — Open the browser and navigate

Use `playwright-cli` to open and navigate to the composed URL:

```bash
playwright-cli open
playwright-cli goto "https://localhost:8601/?extension=https://0.0.0.0:5500/<path-to-dist-file>"
```

> The HTTPS certificates are self-signed. If the browser blocks the page with a certificate warning, handle it:
> ```bash
> playwright-cli snapshot        # check if a warning page is shown
> playwright-cli click <proceed-link-ref>   # click "Advanced" → "Proceed"
> ```
> You may need to do this for both `localhost:8601` and `0.0.0.0:5500`.

### Step 4 — Wait for the editor to load

Take a snapshot and confirm the Scratch editor UI is visible (stage, toolbox, sprite list):

```bash
playwright-cli snapshot
```

If the editor is still loading, wait and snapshot again.

### Step 5 — Check the browser console for errors

```bash
playwright-cli console
```

Look for:
- `Failed to load extension` messages
- Network errors (CORS, 404, certificate rejection)
- JavaScript exceptions thrown by the extension

### Step 6 — Verify extension blocks appear in the toolbox

Take a screenshot of the full editor UI:

```bash
playwright-cli screenshot
```

Inspect the snapshot for a custom category block corresponding to the extension. If not found, the extension likely failed to register — check the console output from Step 5.

### Step 7 — Collect network diagnostics (if needed)

```bash
playwright-cli network
```

Confirm the `.mjs` dist file was fetched with HTTP 200 from `https://0.0.0.0:5500`.

### Step 8 — Report findings

Summarize:
1. Whether the extension URL loaded successfully (HTTP status)
2. Any console errors and their messages
3. Whether the extension's blocks appeared in the toolbox (with screenshot)
4. Suggested fix if an error was identified

## Common Errors and Fixes

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Certificate warning on `localhost:8601` | mkcert CA not trusted | Run `mkcert -install` and regenerate certs |
| Certificate warning on `0.0.0.0:5500` | Live server cert not trusted by browser | Accept the cert by visiting `https://0.0.0.0:5500` directly first |
| 404 on ext URL | Build output missing | Run `npm run build` in extension repo |
| CORS error | Live server not started with `--cors` | Check that `start live server` task is running |
| Extension blocks not shown | Extension threw at registration | Check console for JS errors in extension source |

## Related Skills

- [xcratch-extension-debug](../xcratch-extension-debug/SKILL.md) — full VS Code launch config and source map setup
- [playwright-cli](../../../../../../../../.claude/skills/playwright-cli/SKILL.md) — browser automation commands reference
