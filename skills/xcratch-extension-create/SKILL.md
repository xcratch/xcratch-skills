---
name: xcratch-extension-create
description: 'Use when setting up a new Xcratch extension project and local development environment. Trigger phrases: xcratch-create, scaffold extension, setup-dev, initialize extension repo, local extension setup.'
license: MIT
argument-hint: 'Describe the extension name/ID, GitHub account/repo, and current setup status.'
user-invocable: true
---

# Xcratch Extension Create

Set up a new Xcratch extension repository and local development environment.

## When to Use

- Creating a new extension with `xcratch-create`
- Initializing a repository after scaffold
- Installing dependencies and linking to local `scratch-vm`
- Preparing build and deployment basics

## Required Parameters

Before running any commands, **always ask the user for the following parameters** using `vscode_askQuestions`:

| Parameter | Description | Example |
|---|---|---|
| `repo` | GitHub repository name (recommended prefix: `xcx-`) | `xcx-my-extension` |
| `account` | GitHub account name | `myname` |
| `extensionID` | Internal extension ID (camelCase) | `myExtension` |
| `extensionName` | Display name shown in the editor | `My Extension` |

Do not assume or guess any of these values. If any parameter is missing or unclear, ask for it explicitly before proceeding.

## Quick Start

```sh
# Run in the parent directory of scratch-editor
npx xcratch-create --repo=xcx-<name> --account=<github-account> --extensionID=<id> --extensionName='<Name>'

cd xcx-<name>
git init -b main
npm install
npm run setup-dev
```

## Validate Setup

```sh
npm run build
```

Expected output: `dist/<extensionID>.mjs`

## Deployment URL Pattern

`https://<account>.github.io/<repo>/dist/<extensionID>.mjs`

## Workflow Details

- Full setup workflow: See [./references/workflow.md](./references/workflow.md)
- scratch-editor task and certificate setup for local debug: See [../xcratch-extension-debug/references/vscode-debug.md](../xcratch-extension-debug/references/vscode-debug.md)
