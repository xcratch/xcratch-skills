# Xcratch開発環境セットアップ

## 前提条件

- Node.js (v18以上推奨)
- Git
- mkcert (HTTPS証明書生成用)

## ディレクトリ構成

```
workspace/
├── scratch-editor/          # monorepo (xcratch/scratch-editor)
│   └── packages/
│       ├── scratch-gui/     # フロントエンド
│       ├── scratch-vm/      # 仮想マシン
│       └── scratch-render/  # レンダリング
└── xcx-my-extension/        # 拡張機能
```

## セットアップ手順

### 1. scratch-editorのクローン

```sh
git clone https://github.com/xcratch/scratch-editor.git
cd scratch-editor
npm install
```

### 2. HTTPS証明書の準備

```sh
# mkcertのインストール (macOS)
brew install mkcert
mkcert -install

# 証明書の生成
cd scratch-editor
mkdir -p .vscode
mkcert -key-file .vscode/localhost-key.pem -cert-file .vscode/localhost.pem localhost 127.0.0.1 0.0.0.0 ::1
```

### 3. 拡張機能のスキャフォールド

```sh
cd ..  # scratch-editorの親へ
npx xcratch-create \
  --repo=xcx-my-extension \
  --account=<your-github-account> \
  --extensionID=myExtension \
  --extensionName='My Extension'
```

引数:
- `--repo` - GitHubリポジトリ名 (xcx-で始める慣例)
- `--account` - GitHubアカウント名
- `--extensionID` - 拡張機能の内部ID (キャメルケース推奨)
- `--extensionName` - 表示名

### 4. リポジトリ初期化

```sh
cd xcx-my-extension
git init -b main
git remote add origin <REPO_URL>
git add .
git commit -m "Scaffold code"
git push -u origin main
```

### 5. 依存関係のインストール

```sh
npm install
npm run setup-dev  # scratch-vmへのシンボリックリンク作成
```

## ビルド

```sh
npm run build   # 単発ビルド
npm run watch   # 監視モード
```

出力: `dist/<extensionID>.mjs`

## デバッグ方法

### 方法1: 公開Xcratch Editor経由

1. Live Serverで拡張機能をHTTPS配信
2. https://xcratch.github.io/editor/ を開く
3. 「拡張機能を読み込む」から URL を指定:
   ```
   https://0.0.0.0:5500/xcx-my-extension/dist/myExtension.mjs
   ```

### 方法2: ローカルscratch-editor (推奨)

VSCodeデバッグ設定を使用。詳細は [vscode-debug.md](vscode-debug.md) を参照。

## デプロイ

### GitHub Pages

1. リポジトリ設定 > Pages > Source: `main`ブランチ, `/(root)`
2. 公開URL: `https://<account>.github.io/<repo>/dist/<extensionID>.mjs`

### サンプルプロジェクト

`projects/example.sb3` を配置すると自動読み込み可能:

```
https://xcratch.github.io/editor/#https://<account>.github.io/<repo>/projects/example.sb3
```

### iframe埋め込み

```html
<iframe
  src="https://xcratch.github.io/editor/player#https://<account>.github.io/<repo>/projects/example.sb3"
  width="600" height="500">
</iframe>
```

カメラ/マイク使用時:
```html
<iframe
  src="https://xcratch.github.io/editor/player#..."
  allow="camera; microphone;">
</iframe>
```
