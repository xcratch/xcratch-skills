# Xcratch Development Environment Setup

## Prerequisites

- Node.js (v18 or later recommended)
- Git
- mkcert (for generating HTTPS certificates)

## Directory Layout

```text
workspace/
├── scratch-editor/          # monorepo (xcratch/scratch-editor)
│   └── packages/
│       ├── scratch-gui/     # Frontend
│       ├── scratch-vm/      # Virtual machine
│       └── scratch-render/  # Rendering
└── xcx-my-extension/        # Extension
```

## Setup Steps

### 1. Clone scratch-editor

```sh
git clone https://github.com/xcratch/scratch-editor.git
cd scratch-editor
npm install
```

### 2. Prepare HTTPS certificates

```sh
# Install mkcert (macOS)
brew install mkcert
mkcert -install

# Generate certificates
cd scratch-editor
mkdir -p .vscode
mkcert -key-file .vscode/localhost-key.pem -cert-file .vscode/localhost.pem localhost 127.0.0.1 0.0.0.0 ::1
```

### 3. Scaffold the extension

```sh
cd ..
npx xcratch-create \
  --repo=xcx-my-extension \
  --account=<your-github-account> \
  --extensionID=myExtension \
  --extensionName='My Extension'
```

Arguments:
- `--repo` - GitHub repository name (convention: starts with `xcx-`)
- `--account` - GitHub account name
- `--extensionID` - Internal extension ID (camelCase recommended)
- `--extensionName` - Display name

### 4. Initialize the repository

```sh
cd xcx-my-extension
git init -b main
git remote add origin <REPO_URL>
git add .
git commit -m "Scaffold code"
git push -u origin main
```

### 5. Install dependencies

```sh
npm install
npm run setup-dev
```

## Build

```sh
npm run build
npm run watch
```

Output: `dist/<extensionID>.mjs`

## Debug Methods

### Method 1: Use the public Xcratch Editor

1. Serve the extension over HTTPS with Live Server.
2. Open `https://xcratch.github.io/editor/`.
3. Use "Load Extension" and provide this URL:
   ```text
   https://0.0.0.0:5500/xcx-my-extension/dist/myExtension.mjs
   ```

### Method 2: Local scratch-editor (recommended)

Use VS Code debug configuration. For details, see [vscode-debug.md](vscode-debug.md).

## Deployment

### GitHub Pages

1. Repository settings > Pages > Source: `main` branch, `/(root)`
2. Public URL: `https://<account>.github.io/<repo>/dist/<extensionID>.mjs`

### Sample project

Place `projects/example.sb3` in your repository to enable auto-loading:

```text
https://xcratch.github.io/editor/#https://<account>.github.io/<repo>/projects/example.sb3
```

### Embed with an iframe

```html
<iframe
  src="https://xcratch.github.io/editor/player#https://<account>.github.io/<repo>/projects/example.sb3"
  width="600" height="500">
</iframe>
```

When using camera/microphone:

```html
<iframe
  src="https://xcratch.github.io/editor/player#..."
  allow="camera; microphone;">
</iframe>
```
