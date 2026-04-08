---
name: xcratch-extension
description: 'Use when creating, debugging, building, or deploying Xcratch extensions. Keywords: Xcratch extension, xcratch-create, scratch-editor, GitHub Pages, VS Code debug, local development setup.'
argument-hint: 'Describe the Xcratch extension task, current setup, and expected outcome.'
user-invocable: true
---

# Xcratch Extension Development

Development workflow for Xcratch extensions.

## When to Use

- Creating a new Xcratch extension
- Setting up a local development environment for an existing extension
- Debugging an extension in VS Code
- Deploying an extension to GitHub Pages

## Quick Reference

### Create a New Extension

```sh
# Run in the parent directory of scratch-editor
npx xcratch-create --repo=xcx-<name> --account=<github-account> --extensionID=<id> --extensionName='<Name>'

# Initialize
cd xcx-<name>
git init -b main
npm install
npm run setup-dev  # Create a symbolic link to scratch-vm
```

### Build and Debug

```sh
npm run build   # Generates dist/<extensionID>.mjs
npm run watch   # Watch mode
```

### Deploy

GitHub Pages settings: set the publishing source to `main` branch `/(root)`.

Public URL: `https://<account>.github.io/<repo>/dist/<extensionID>.mjs`

## Workflow Details

- Development environment setup: See [./references/workflow.md](./references/workflow.md)
- VS Code debug settings: See [./references/vscode-debug.md](./references/vscode-debug.md)

## Directory Structure

```text
xcx-<extension>/
├── src/
│   ├── vm/extensions/block/
│   │   ├── index.js          # Block definitions (main logic)
│   │   ├── translations.json # Translations
│   │   └── block-icon.png    # Block icon
│   └── gui/lib/libraries/extensions/entry/
│       ├── index.jsx         # Extension entry
│       ├── entry-icon.png    # Catalog icon
│       ├── inset-icon.svg    # Inset icon
│       └── translations.json # Translations
├── dist/                      # Build output
├── projects/                  # Sample projects
└── scripts/                   # Build scripts
```

## Key Files

### src/vm/extensions/block/index.js

Main file for implementing block definitions and logic.

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
          blockType: BlockType.COMMAND,
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
  }
}
```

### BlockType Options

- `COMMAND` - Block with no return value
- `REPORTER` - Block that returns a value
- `BOOLEAN` - Block that returns true/false
- `HAT` - Event trigger block
- `CONDITIONAL` - Conditional branch block
- `LOOP` - Loop block

### ArgumentType Options

- `STRING` - String input
- `NUMBER` - Numeric input
- `BOOLEAN` - Boolean value
- `COLOR` - Color picker
- `ANGLE` - Angle input
- `NOTE` - Musical note input
- `MATRIX` - 2D matrix input
- `IMAGE` - Inline image in labels
