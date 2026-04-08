# VS Code Debug Setup

Using scratch-editor's VS Code debugging support, you can set breakpoints directly in extension source code.

## Prerequisites

- scratch-editor and the xcx extension are in the same parent directory
- HTTPS certificates are already configured (see below)
- VS Code Live Server extension is installed

## Configure HTTPS Certificates with mkcert

To run an HTTPS server in local development, use mkcert to create a local Certificate Authority (CA) and certificates.

### 1. Install mkcert

```sh
# macOS
brew install mkcert

# Windows (Chocolatey)
choco install mkcert

# Linux (apt)
sudo apt install mkcert
```

### 2. Install the local CA

```sh
mkcert -install
```

This installs the local CA into the system's trusted root certificate store.

### 3. Generate certificates

Create certificates in scratch-editor's `.vscode` directory:

```sh
cd scratch-editor/.vscode
mkcert -cert-file localhost.pem -key-file localhost-key.pem localhost 127.0.0.1 0.0.0.0 ::1
```

This generates the following files:
- `localhost.pem` - certificate file
- `localhost-key.pem` - private key file

### 4. HTTPS behavior of scratch-editor dev server

The scratch-editor dev server automatically detects certificate files in `.vscode` and starts with HTTPS.

If certificate files are not found, it starts with HTTP.

## Workspace Setup

### 1. Create a multi-root workspace

`xcx-my-extension.code-workspace`:

```json
{
  "folders": [
    { "path": "xcx-my-extension" },
    { "path": "scratch-editor" }
  ],
  "settings": {
    "liveServer.settings.root": "../",
    "liveServer.settings.multiRootWorkspaceName": "xcx-my-extension"
  }
}
```

### 2. Live Server configuration

`.vscode/settings.json` (in the extension project):

```json
{
  "liveServer.settings.port": 5500,
  "liveServer.settings.host": "0.0.0.0",
  "liveServer.settings.https": {
    "enable": true,
    "cert": "/path/to/scratch-editor/.vscode/localhost.pem",
    "key": "/path/to/scratch-editor/.vscode/localhost-key.pem"
  },
  "liveServer.settings.CustomBrowser": "chrome"
}
```

### 3. launch.json (in scratch-editor)

`scratch-editor/.vscode/launch.json` includes the following configuration:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "debug on dev-server",
      "url": "https://0.0.0.0:8601",
      "webRoot": "${workspaceFolder}/..",
      "sourceMapPathOverrides": {
        "webpack://GUI/*": "${webRoot}/scratch-editor/packages/scratch-gui/*",
        "webpack://GUI/scratch-vm/*": "${webRoot}/scratch-editor/packages/scratch-vm/*",
        "https://0.0.0.0:5500/*": "${webRoot}/*"
      }
    }
  ]
}
```

## Debug Steps

### 1. Start extension watch mode

```sh
cd xcx-my-extension
npm run watch
```

### 2. Start Live Server

Open the extension folder in VS Code and start Live Server.
It will be served at `https://0.0.0.0:5500/`.

### 3. Start scratch-editor dev server

```sh
cd scratch-editor/packages/scratch-gui
npm run start
```

### 4. Start debugger

1. Open the scratch-editor folder in VS Code.
2. Press F5 and launch `debug on dev-server`.
3. Chrome opens at `https://0.0.0.0:8601`.

### 5. Load extension

1. In Xcratch Editor, click "Load Extension".
2. Enter this URL:
   ```text
   https://0.0.0.0:5500/xcx-my-extension/dist/myExtension.mjs
   ```
3. The extension is loaded.

### 6. Set breakpoints

1. Open `src/vm/extensions/block/index.js` in VS Code.
2. Set breakpoints on lines you want to debug.
3. Run blocks in Scratch.
4. Execution stops at breakpoints.

## Troubleshooting

### Source maps are not resolved

- Check `sourceMapPathOverrides` in launch.json.
- Verify that `webRoot` points to the workspace parent directory.

### CORS errors

- Check whether Live Server is running on the correct host/port.
- Verify that HTTPS certificates are valid.

### Extension is not loading

- Check whether `npm run watch` is running.
- Verify that an `.mjs` file is generated in `dist/`.
- Check for network errors in browser developer tools.
