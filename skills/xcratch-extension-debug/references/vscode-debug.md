# VS Code Debug Setup

Using VS Code's Chrome debugger together with a local HTTPS live server, you can set breakpoints
directly in extension source code while the extension runs inside the public editor at
`https://xcratch.github.io/editor/`. No local `scratch-editor` checkout or multi-root workspace is needed.

## Prerequisites

- HTTPS certificates are configured in the extension's `.vscode` directory (see below)
- `npx live-server` is available (installed on demand via `npx --yes live-server`)

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

Create certificates in the extension's own `.vscode` directory:

```sh
cd xcx-my-extension/.vscode
mkcert -cert-file localhost.pem -key-file localhost-key.pem localhost 127.0.0.1 0.0.0.0 ::1
```

This generates the following files:
- `localhost.pem` - certificate file
- `localhost-key.pem` - private key file

These files are matched by the `*.pem` rule in `.gitignore`, so they are not committed.

### 4. live-server HTTPS config

`.vscode/live-server-https.cjs` loads the cert/key for `live-server`:

```js
const fs = require('node:fs');
const path = require('node:path');

module.exports = {
  cert: fs.readFileSync(path.join(__dirname, 'localhost.pem')),
  key: fs.readFileSync(path.join(__dirname, 'localhost-key.pem'))
};
```

## Workspace Setup

No multi-root workspace or `scratch-editor` checkout is required — only the extension repo itself.

### 1. tasks.json (in the extension repo)

`.vscode/tasks.json` defines the task that serves the workspace root over HTTPS on port `5500`:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "shell",
      "command": "npx --yes live-server . --host=0.0.0.0 --port=5500 --https=./.vscode/live-server-https.cjs --cors --no-browser",
      "isBackground": true,
      "problemMatcher": {
        "owner": "custom",
        "pattern": { "regexp": "^$" },
        "background": {
          "activeOnStart": true,
          "beginsPattern": ".*",
          "endsPattern": ".*Serving.*"
        }
      },
      "label": "start live server"
    }
  ]
}
```

This serves `dist/xcratchExample.mjs` at `https://0.0.0.0:5500/dist/xcratchExample.mjs`.

### 2. launch.json (in the extension repo)

`.vscode/launch.json` opens Chrome at the public editor with the extension URL pre-filled via the `?extension=` query parameter:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "preLaunchTask": "start live server",
      "name": "debug extension on xcratch.github.io editor",
      "url": "https://xcratch.github.io/editor/?extension=${input:extensionUrl}",
      "webRoot": "${workspaceFolder}",
      "sourceMaps": true,
      "sourceMapRenames": true,
      "sourceMapPathOverrides": {
        "https://0.0.0.0:5500/*": "${webRoot}/*"
      }
    },
    {
      "type": "chrome",
      "request": "launch",
      "name": "attach on xcratch.github.io editor",
      "url": "https://xcratch.github.io/editor/?extension=${input:extensionUrl}",
      "webRoot": "${workspaceFolder}",
      "sourceMaps": true,
      "sourceMapRenames": true,
      "sourceMapPathOverrides": {
        "https://0.0.0.0:5500/*": "${webRoot}/*"
      }
    }
  ],
  "inputs": [
    {
      "id": "extensionUrl",
      "type": "promptString",
      "description": "Extension URL to load, for example: https://0.0.0.0:5500/dist/xcratchExample.mjs",
      "default": "https://0.0.0.0:5500/dist/xcratchExample.mjs"
    }
  ]
}
```

Related launch configurations:

- `debug extension on xcratch.github.io editor`: starts `start live server` and launches Chrome at the public editor with the extension preloaded
- `attach on xcratch.github.io editor`: launches Chrome against the public editor without starting tasks (use when the live server is already running)

### 3. tasks.json summary

- `start live server`: serves the workspace root over HTTPS on port `5500` (no `scratch-gui` dev server needed, since the public editor is used)

## Debug Steps

### 1. Start extension watch mode

```sh
cd xcx-my-extension
npm run watch
```

### 2. Start debugger

1. Open the extension folder in VS Code.
2. Press F5 and choose `debug extension on xcratch.github.io editor`.
3. VS Code runs task `start live server`.
4. Enter (or accept the default) extension URL, e.g. `https://0.0.0.0:5500/dist/xcratchExample.mjs`.
5. Chrome opens at `https://xcratch.github.io/editor/?extension=https://0.0.0.0:5500/dist/xcratchExample.mjs` and the extension loads automatically.

If the live server is already running, use `attach on xcratch.github.io editor` instead.

> The HTTPS certificate for `https://0.0.0.0:5500` is self-signed. On first load, Chrome may show a
> certificate warning — click through "Advanced" → "Proceed" to `0.0.0.0:5500` (visiting the URL
> directly once is enough to trust it for the session).

### 3. Set breakpoints

1. Open `src/vm/extensions/block/index.js` in VS Code.
2. Set breakpoints on lines you want to debug.
3. Run blocks in Scratch.
4. Execution stops at breakpoints.

## Troubleshooting

### Source maps are not resolved

- Check `sourceMapPathOverrides` in launch.json.
- Verify that `webRoot` points to the extension's workspace folder (`${workspaceFolder}`).
- If you launched manually, verify the same hostnames are used as the launch config (`https://xcratch.github.io/` and `https://0.0.0.0:5500/`).

### CORS errors

- Check whether `live-server` is running on the correct host/port (`0.0.0.0:5500`).
- Verify that HTTPS certificates are valid and the `--cors` flag is present in the task command.

### Extension is not loading

- Check whether `npm run watch` is running in the extension repository.
- Check whether task `start live server` is serving the extension URL on port `5500`.
- Verify that an `.mjs` file is generated in `dist/`.
- Check for network errors (certificate rejection, 404, CORS) in browser developer tools.
