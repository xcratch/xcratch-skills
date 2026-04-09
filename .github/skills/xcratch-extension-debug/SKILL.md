---
name: xcratch-extension-debug
description: 'Use when debugging an in-development Xcratch extension in VS Code. Trigger phrases: breakpoints not hit, source maps, debug on dev-server, live server HTTPS, extension not loading in editor.'
argument-hint: 'Describe current debug setup, failing behavior, and expected result.'
user-invocable: true
---

# Xcratch Extension Debug

Debug an extension under development with local `scratch-editor` and VS Code.

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

Then in the `scratch-editor` workspace, run VS Code launch config `debug on dev-server`.
This launch uses preLaunchTask `start debug servers`, which starts these existing tasks automatically:

- `start live server`
- `start https`

## Load Extension URL

`https://0.0.0.0:5500/xcx-my-extension/dist/myExtension.mjs`

## Workflow Details

- VS Code debug configuration and troubleshooting: See [./references/vscode-debug.md](./references/vscode-debug.md)
