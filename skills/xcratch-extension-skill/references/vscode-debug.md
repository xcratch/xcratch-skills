# VSCode デバッグ設定

scratch-editorのVSCodeデバッグ機能を使用すると、拡張機能のソースコードに直接ブレークポイントを設定できる。

## 前提条件

- scratch-editorとxcx-拡張機能が同じ親ディレクトリにある
- HTTPS証明書が設定済み
- VSCode Live Server拡張機能がインストール済み

## ワークスペース設定

### 1. マルチルートワークスペースの作成

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

### 2. Live Server設定

`.vscode/settings.json` (拡張機能側):

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

### 3. launch.json (scratch-editor側)

scratch-editorの`.vscode/launch.json`には以下の設定が含まれている:

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

## デバッグ手順

### 1. 拡張機能のwatchモード起動

```sh
cd xcx-my-extension
npm run watch
```

### 2. Live Serverの起動

VSCodeで拡張機能フォルダを開き、Live Serverを起動。
`https://0.0.0.0:5500/` で配信される。

### 3. scratch-editorの開発サーバー起動

```sh
cd scratch-editor
npm run start
```

### 4. デバッガーの起動

1. VSCodeでscratch-editorフォルダを開く
2. F5キーで「debug on dev-server」を起動
3. Chromeが `https://0.0.0.0:8601` で開く

### 5. 拡張機能の読み込み

1. Xcratch Editorで「拡張機能を読み込む」をクリック
2. URLを入力:
   ```
   https://0.0.0.0:5500/xcx-my-extension/dist/myExtension.mjs
   ```
3. 拡張機能が読み込まれる

### 6. ブレークポイントの設定

1. VSCodeで`src/vm/extensions/block/index.js`を開く
2. デバッグしたい行にブレークポイントを設定
3. Scratch上でブロックを実行
4. ブレークポイントで停止

## トラブルシューティング

### ソースマップが解決されない

- launch.jsonの`sourceMapPathOverrides`を確認
- `webRoot`がワークスペースの親ディレクトリを指しているか確認

### CORS エラー

- Live Serverが正しいホスト/ポートで起動しているか確認
- HTTPS証明書が有効か確認

### 拡張機能が読み込まれない

- `npm run watch`が実行中か確認
- `dist/`フォルダに`.mjs`ファイルが生成されているか確認
- ブラウザの開発者ツールでネットワークエラーを確認
