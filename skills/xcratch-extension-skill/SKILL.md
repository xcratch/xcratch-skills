---
name: xcratch-extension
description: Xcratch拡張機能の開発ワークフローガイド。新規拡張機能の作成、ローカル環境でのデバッグ、GitHub Pagesへのデプロイをサポート。以下の場合に使用：(1) 新しいXcratch拡張機能を作成する、(2) 拡張機能をローカルでデバッグする、(3) 拡張機能をGitHub Pagesにデプロイする、(4) Xcratch開発環境をセットアップする
---

# Xcratch Extension Development

Xcratch拡張機能の開発ワークフロー。

## Quick Reference

### 新規拡張機能の作成

```sh
# scratch-editorの親ディレクトリで実行
npx xcratch-create --repo=xcx-<name> --account=<github-account> --extensionID=<id> --extensionName='<Name>'

# 初期化
cd xcx-<name>
git init -b main
npm install
npm run setup-dev  # scratch-vmへのシンボリックリンク作成
```

### ビルドとデバッグ

```sh
npm run build   # dist/<extensionID>.mjs を生成
npm run watch   # 変更監視モード
```

### デプロイ

GitHub Pages設定: `main`ブランチの`/(root)`を配信元に設定

公開URL: `https://<account>.github.io/<repo>/dist/<extensionID>.mjs`

## Workflow Details

- **開発環境セットアップ**: See [references/workflow.md](references/workflow.md)
- **VSCodeデバッグ設定**: See [references/vscode-debug.md](references/vscode-debug.md)

## Directory Structure

```
xcx-<extension>/
├── src/
│   ├── vm/extensions/block/
│   │   ├── index.js          # ブロック定義（メインロジック）
│   │   ├── translations.json # 翻訳
│   │   └── block-icon.png    # ブロックアイコン
│   └── gui/lib/libraries/extensions/entry/
│       ├── index.jsx         # 拡張機能エントリー
│       ├── entry-icon.png    # カタログアイコン
│       ├── inset-icon.svg    # インセットアイコン
│       └── translations.json # 翻訳
├── dist/                      # ビルド出力
├── projects/                  # サンプルプロジェクト
└── scripts/                   # ビルドスクリプト
```

## Key Files

### src/vm/extensions/block/index.js

ブロックの定義とロジックを実装するメインファイル。

```js
import BlockType from '../../extension-support/block-type';
import ArgumentType from '../../extension-support/argument-type';
import Cast from '../../util/cast';

class ExtensionBlocks {
  getInfo() {
    return {
      id: 'extensionID',
      name: 'Extension Name',
      blocks: [
        {
          opcode: 'blockName',
          blockType: BlockType.COMMAND,  // or REPORTER, BOOLEAN, HAT
          text: 'block text [ARG]',
          arguments: {
            ARG: {
              type: ArgumentType.STRING,
              defaultValue: 'default'
            }
          }
        }
      ]
    };
  }

  blockName(args) {
    const value = Cast.toString(args.ARG);
    // implement block logic
  }
}
```

### BlockType Options

- `COMMAND` - 戻り値なしのブロック
- `REPORTER` - 値を返すブロック
- `BOOLEAN` - true/falseを返すブロック
- `HAT` - イベントトリガーブロック
- `CONDITIONAL` - 条件分岐ブロック
- `LOOP` - ループブロック

### ArgumentType Options

- `STRING` - 文字列入力
- `NUMBER` - 数値入力
- `BOOLEAN` - ブール値
- `COLOR` - カラーピッカー
- `ANGLE` - 角度入力
- `NOTE` - 音符入力
- `MATRIX` - 2次元マトリックス入力
- `IMAGE` - ラベルのインライン画像
