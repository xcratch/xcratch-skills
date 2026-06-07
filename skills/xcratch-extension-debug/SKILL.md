---
name: xcratch-extension-debug
description: 'Use when debugging an in-development Xcratch extension in VS Code. Trigger phrases: breakpoints not hit, source maps, debug on xcratch.github.io editor, live server HTTPS, extension not loading in editor.'
license: MIT
argument-hint: 'Describe current debug setup, failing behavior, and expected result.'
user-invocable: true
---

# Xcratch Extension Debug

Debug an extension under development with the public `xcratch.github.io` editor and VS Code, by serving the local build over HTTPS and loading it via the `?extension=` query parameter. No local `scratch-editor` checkout is required.

## When to Use

- Breakpoints in extension source are not hit
- Source maps are not resolved
- Extension fails to load from local HTTPS URL
- You need a reproducible VS Code debug workflow

## Quick Start

```sh
# Extension repo
npm run watch
```

Then in VS Code, run launch config `debug extension on xcratch.github.io editor`.
This launch uses `preLaunchTask: start live server`, which serves the workspace root over HTTPS on `https://0.0.0.0:5500`, then opens Chrome at:

```text
https://xcratch.github.io/editor/?extension=https://0.0.0.0:5500/dist/xcratchExample.mjs
```

You will be prompted for the extension URL; the default points at `dist/xcratchExample.mjs`.

## Load Extension URL

`https://0.0.0.0:5500/dist/xcratchExample.mjs`

## Workflow Details

- VS Code debug configuration and troubleshooting: See [./references/vscode-debug.md](./references/vscode-debug.md)
