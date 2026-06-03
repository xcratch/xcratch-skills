# File Templates for stretch3 Integration

## `scripts/stretch3-install.sh`

```sh
#!/bin/sh

## Install script for https://stretch3.github.io/
## execute `sh ./<EXTENSION_REP>/scripts/stretch3-install.sh`
## on the dir configuration:
##  scratch-gui
##      - <EXTENSION_REP>

LF=$(printf '\\\012_')
LF=${LF%_}
EXTENSION_REP=<EXTENSION_REP>
EXTENSION_ID=<EXTENSION_ID>

### register it as a builtin extension
mkdir -p node_modules/scratch-vm/src/extensions/${EXTENSION_ID}
cp ${EXTENSION_REP}/dist/${EXTENSION_ID}.mjs node_modules/scratch-vm/src/extensions/${EXTENSION_ID}/
mv node_modules/scratch-vm/src/extension-support/extension-manager.js node_modules/scratch-vm/src/extension-support/extension-manager.js_orig
sed -e "s|class ExtensionManager {|builtinExtensions['${EXTENSION_ID}'] = () => {${LF}    const formatMessage = require('format-message');${LF}    const ext = require('../extensions/${EXTENSION_ID}/${EXTENSION_ID}.mjs');${LF}    const blockClass = ext.blockClass;${LF}    blockClass.formatMessage = formatMessage;${LF}    return blockClass;${LF}};${LF}${LF}class ExtensionManager {|g" node_modules/scratch-vm/src/extension-support/extension-manager.js_orig > node_modules/scratch-vm/src/extension-support/extension-manager.js

### copy entry files
mkdir -p src/lib/libraries/extensions/${EXTENSION_ID}
cp ${EXTENSION_REP}/src/gui/lib/libraries/extensions/entry/index-stretch.jsx src/lib/libraries/extensions/${EXTENSION_ID}/index.jsx
cp ${EXTENSION_REP}/src/gui/lib/libraries/extensions/entry/translations.json src/lib/libraries/extensions/${EXTENSION_ID}/
cp ${EXTENSION_REP}/src/gui/lib/libraries/extensions/entry/entry-icon.png src/lib/libraries/extensions/${EXTENSION_ID}/
cp ${EXTENSION_REP}/src/gui/lib/libraries/extensions/entry/inset-icon.svg src/lib/libraries/extensions/${EXTENSION_ID}/

### insert it to the library
mv src/lib/libraries/extensions/index.jsx src/lib/libraries/extensions/index.jsx_orig
sed -e "s|^export default \[$|import ${EXTENSION_ID}Entry from './${EXTENSION_ID}/index.jsx';${LF}${LF}export default [${LF}    ${EXTENSION_ID}Entry,|g" src/lib/libraries/extensions/index.jsx_orig > src/lib/libraries/extensions/index.jsx

### apply webpack-mjs-support patch
patch -p1 < ${EXTENSION_REP}/scripts/webpack-mjs-support.patch || true
```

## `src/gui/lib/libraries/extensions/entry/index-stretch.jsx`

```jsx
/* eslint-disable import/no-unresolved */
import React from 'react';
import {FormattedMessage} from 'react-intl';

/**
 * This is an extension for Xcratch.
 */

import iconURL from './entry-icon.png';
import insetIconURL from './inset-icon.svg';

const translations =
{
    'en': {
        '<EXTENSION_ID>.entry.name': '<EXTENSION_NAME>',
        '<EXTENSION_ID>.entry.description': '<DESCRIPTION_EN>'
    },
    'ja': {
        '<EXTENSION_ID>.entry.name': '<EXTENSION_NAME>',
        '<EXTENSION_ID>.entry.description': '<DESCRIPTION_JA>'
    },
    'ja-Hira': {
        '<EXTENSION_ID>.entry.name': '<EXTENSION_NAME>',
        '<EXTENSION_ID>.entry.description': '<DESCRIPTION_JA_HIRA>'
    }
};

const entry = {
    name: (
        <FormattedMessage
            defaultMessage="<EXTENSION_NAME>"
            id="<EXTENSION_ID>.entry.name"
        />
    ),
    extensionId: '<EXTENSION_ID>',
    extensionURL: null,
    collaborator: '<COLLABORATOR>',
    iconURL: iconURL,
    insetIconURL: insetIconURL,
    description: (
        <FormattedMessage
            defaultMessage="<DESCRIPTION_EN>"
            description="Description for this extension"
            id="<EXTENSION_ID>.entry.description"
        />
    ),
    tags: <TAGS>,
    featured: true,
    disabled: false,
    bluetoothRequired: false,
    internetConnectionRequired: false,
    helpLink: '<HELP_LINK>',
    translationMap: translations
};

export default entry;
```

## `scripts/webpack-mjs-support.patch`

This file is identical for every extension:

```diff
diff --git a/webpack.config.js b/webpack.config.js
index 32e8246ce..ecb137e3f 100644
--- a/webpack.config.js
+++ b/webpack.config.js
@@ -32,7 +32,7 @@ const base = {
     },
     module: {
         rules: [{
-            test: /\.jsx?$/,
+            test: /\.(jsx?|mjs)$/,
             loader: 'babel-loader',
             include: [
                 path.resolve(__dirname, 'src'),
```
