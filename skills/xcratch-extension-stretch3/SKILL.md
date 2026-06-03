---
name: xcratch-extension-stretch3
description: 'Use when adding an Xcratch extension to stretch3 (https://stretch3.github.io). Creates stretch3-install.sh, webpack-mjs-support.patch, and index-stretch.jsx. Trigger phrases: add to stretch3, stretch3 install, stretch3-install, stretch3 スクリプト.'
license: MIT
argument-hint: 'Run from the root of the Xcratch extension repo (xcx-*). Optionally provide en/ja descriptions.'
user-invocable: true
---

# Xcratch Extension — stretch3 Integration

Add an Xcratch extension to [stretch3](https://stretch3.github.io) by creating the three required files and informing the user what to add to the stretch3 `deploy.yml`.

## When to Use

- Adding an Xcratch extension to stretch3 for the first time
- Re-generating stretch3 files after updating the extension

## Step 1 — Gather Extension Metadata

Read these files from the current working directory (the extension repo root):

| Source | Fields to extract |
|---|---|
| `package.json` | `name` (= EXTENSION_REP), `extensionId` (= EXTENSION_ID), `version` |
| `src/gui/lib/libraries/extensions/entry/index.jsx` | `collaborator`, `helpLink`, `tags` |
| `src/gui/lib/libraries/extensions/entry/translations.json` | existing `en`/`ja`/`ja-Hira` descriptions |

If `extensionId` is missing from `package.json`, derive it from `name` (strip `xcx-` prefix).

## Step 2 — Confirm Descriptions

If the existing descriptions in `translations.json` look like placeholders (e.g. "Description of this extension" / "拡張機能の説明"), **ask the user** for better en/ja/ja-Hira descriptions before writing files. Otherwise use the existing values.

## Step 3 — Create Files

Create the following three files. See [./references/file-templates.md](./references/file-templates.md) for exact content patterns.

### `scripts/stretch3-install.sh`

Shell script that (a) copies `dist/${EXTENSION_ID}.mjs` into scratch-vm and patches `extension-manager.js`, (b) copies entry files to scratch-gui, (c) injects the entry import into `src/lib/libraries/extensions/index.jsx`, (d) applies `webpack-mjs-support.patch` with `|| true` so a duplicate-apply doesn't break the build.

### `src/gui/lib/libraries/extensions/entry/index-stretch.jsx`

stretch3-specific entry file. Key differences from the regular `index.jsx`:
- Uses React JSX (`<FormattedMessage>`) instead of getter functions
- Embeds translations directly (no import from `translations.json`)
- Sets `extensionURL: null` (extension is registered as builtin)

### `scripts/webpack-mjs-support.patch`

Identical for every Xcratch extension — changes `test: /\.jsx?$/` to `test: /\.(jsx?|mjs)$/` in `webpack.config.js`.

## Step 4 — Commit

Stage and commit only the three new files:

```sh
git add scripts/stretch3-install.sh scripts/webpack-mjs-support.patch \
        src/gui/lib/libraries/extensions/entry/index-stretch.jsx
git commit -m "feat: add stretch3 install script and entry files"
```

## Step 5 — Inform the User

After committing, tell the user what to add to stretch3's `deploy.yml` (they need to send a PR to `stretch3/stretch3.github.io`):

```yaml
# checkout step — add near the other extension checkouts
- uses: actions/checkout@v4
  with:
    repository: <GITHUB_ACCOUNT>/<EXTENSION_REP>
    path: ./<EXTENSION_REP>

# run step — add near the other install script runs
- run: sh ./<EXTENSION_REP>/scripts/stretch3-install.sh
```

Replace `<GITHUB_ACCOUNT>` with the actual GitHub account (from `collaborator` in `index.jsx` or the repo URL).
