# Xcratch Extension Project Setup

## Prerequisites

- Node.js (v18 or later recommended)
- Git
- mkcert (for HTTPS in local integration)

For scratch-editor debug prerequisites, see [../../xcratch-extension-debug/references/vscode-debug.md](../../xcratch-extension-debug/references/vscode-debug.md).

## Directory Layout

```text
workspace/
|- scratch-editor/
|  |- packages/
|     |- scratch-gui/
|     |- scratch-vm/
|     `- scratch-render/
`- xcx-my-extension/
```

## Setup Steps

### 1. Clone and install scratch-editor

```sh
git clone https://github.com/xcratch/scratch-editor.git
cd scratch-editor
npm install
```

### 2. Prepare HTTPS certificates

Certificate generation details and the related scratch-editor debug workflow are documented in [../../xcratch-extension-debug/references/vscode-debug.md](../../xcratch-extension-debug/references/vscode-debug.md).

```sh
# macOS
brew install mkcert
mkcert -install

cd scratch-editor
mkdir -p .vscode
mkcert -key-file .vscode/localhost-key.pem -cert-file .vscode/localhost.pem localhost 127.0.0.1 0.0.0.0 ::1
```

### 3. Scaffold extension

```sh
cd ..
npx xcratch-create \
  --repo=xcx-my-extension \
  --account=<your-github-account> \
  --extensionID=myExtension \
  --extensionName='My Extension'
```

Arguments:
- `--repo`: GitHub repository name (recommended prefix: `xcx-`)
- `--account`: GitHub account name
- `--extensionID`: Internal extension ID (camelCase recommended)
- `--extensionName`: Display name shown in editor

### 4. Initialize and push repository

```sh
cd xcx-my-extension
git init -b main
git remote add origin <REPO_URL>
git add .
git commit -m "Scaffold code"
git push -u origin main
```

### 5. Install and link dependencies

```sh
npm install
npm run setup-dev
```

If you plan to debug with local scratch-editor, use these existing VS Code entries in scratch-editor:

- launch: `debug on dev-server`
- tasks: `start debug servers`, `start live server`, `start https`

See [../../xcratch-extension-debug/references/vscode-debug.md](../../xcratch-extension-debug/references/vscode-debug.md) for the full task and certificate setup.

## Build

```sh
npm run build
npm run watch
```

Output: `dist/<extensionID>.mjs`

## GitHub Pages Deployment

1. Repository Settings -> Pages -> Source: `main` branch, `/(root)`
2. Extension URL: `https://<account>.github.io/<repo>/dist/<extensionID>.mjs`
