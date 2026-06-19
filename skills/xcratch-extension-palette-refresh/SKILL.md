---
name: xcratch-extension-palette-refresh
description: 'Use when a scratch-vm / Xcratch extension creates, deletes, or renames a variable or list programmatically (e.g. target.lookupOrCreateList / createVariable / deleteVariable) and the change does NOT appear in the editor block palette until the user re-opens the Code tab or switches sprites. Make sure to use this whenever programmatic changes to the VM variable/list model (or custom My Blocks procedures) are not reflected in the Scratch editor palette right away, or whenever someone asks how to force the Blockly toolbox / variable-list flyout to refresh from extension code. Trigger phrases: list created but not in palette, variable not showing until tab switch, requestBlocksUpdate / emitWorkspaceUpdate did not update the palette, refresh toolbox flyout, force variable category refresh, lookupOrCreateList not visible, deleted variable still in palette.'
license: MIT
argument-hint: 'Describe what you created (variable/list/blocks), how (which VM API), and where it fails to show up.'
user-invocable: true
---

# Xcratch Extension: Refresh the Palette After Programmatic Variable/List Creation

When an Xcratch / scratch-vm extension creates a **variable or list** programmatically — typically `target.lookupOrCreateList(id, name)` or `target.createVariable(...)` — the new entry often does **not** appear in the editor's block palette (the "Variables" category flyout) until the user re-opens the Code tab or switches sprites. This skill explains why that happens and gives a verified, copy-pasteable fix.

The **mirror cases — deleting or renaming** a variable/list (e.g. `target.deleteVariable(id)`) — have the exact same cause and the exact same fix: the stale entry lingers, or the old name keeps showing, in the flyout until a refresh. The same symptom also appears for **custom procedures ("My Blocks")** and any other *dynamic* toolbox category populated from the workspace at flyout-show time. Throughout this skill, "create" stands in for any of create / delete / rename.

## Symptom checklist

Use this skill when all of these are true:

- Your extension changed the VM model directly (not via `vm.loadProject`).
- The change adds, removes, or renames a variable, list, or procedure that should be reflected in the palette.
- The palette only updates after a Code-tab switch / sprite switch, not immediately.
- You may have already tried `runtime.emitProjectChanged()` or `runtime.requestBlocksUpdate()` and it "did nothing" visible.

## TL;DR — the fix

Two parts are **both** required:

1. **Sync the new variable into the Blockly workspace** so the palette has data to show:
   `runtime.requestBlocksUpdate()` (the VM turns this into a workspace XML reload).
2. **Force the dynamic flyout to actually re-render** — the GUI will *not* do this on its own for a variable-only change (see "Why" below). Reach the scratch-blocks workspace from the main thread and refresh its toolbox.

Drop this helper into your extension and call it right after creating the variable/list:

```js
/**
 * Force the Blockly variable/list flyout to re-render so a just-created list or
 * variable shows up in the palette immediately, without re-opening the Code tab.
 *
 * Best-effort: Xcratch extensions run on the main thread with DOM access, so we
 * reach the scratch-blocks workspace through the rendered DOM + React fiber. If
 * the editor internals differ, the entry still appears on the next natural
 * toolbox refresh (tab switch), so failure here is non-fatal.
 */
const refreshVariablePalette = function () {
    if (typeof document === 'undefined') return;
    try {
        const svg = document.querySelector('svg.blocklyWorkspace') ||
            document.querySelector('.blocklyWorkspace');
        let el = svg && svg.parentElement;
        for (let depth = 0; depth < 20 && el; depth++) {
            const fiberKey = Object.keys(el).find(k =>
                k.startsWith('__reactFiber$') || k.startsWith('__reactInternalInstance$'));
            if (fiberKey) {
                let fiber = el[fiberKey];
                while (fiber) {
                    const ws = fiber.stateNode && fiber.stateNode.workspace;
                    if (ws && typeof ws.refreshToolboxSelection_ === 'function') {
                        // The workspace XML reload (from requestBlocksUpdate) leaves
                        // toolboxRefreshEnabled_ === false, which makes
                        // refreshToolboxSelection_ a no-op. Re-enable it first — the
                        // same thing the GUI's own updateToolbox() does.
                        ws.toolboxRefreshEnabled_ = true;
                        ws.refreshToolboxSelection_();
                        return;
                    }
                    fiber = fiber.return;
                }
            }
            el = el.parentElement;
        }
    } catch (e) {
        // ignore: best-effort only
    }
};
```

Call site (only when something was actually created — see "Do it once"):

```js
const list = target.lookupVariableByNameAndType(name, 'list') ||
    target.lookupOrCreateList(id, name);

if (/* it was newly created */ target.runtime) {
    target.runtime.emitProjectChanged();          // mark dirty (save prompt)
    if (typeof target.runtime.requestBlocksUpdate === 'function') {
        target.runtime.requestBlocksUpdate();     // (1) sync into Blockly var map
    }
    refreshVariablePalette();                      // (2) re-render the flyout
}
```

`requestBlocksUpdate()` runs synchronously through the GUI's `onWorkspaceUpdate` handler, so by the time it returns the workspace variable map already contains the new entry. That is why calling `refreshVariablePalette()` immediately after works.

The same two-step call applies unchanged to **deletes and renames** — run `runtime.requestBlocksUpdate()` then `refreshVariablePalette()` right after `target.deleteVariable(id)` (or the rename) so the removed/renamed entry stops showing the stale value in the flyout.

## Why it happens (root cause)

Knowing the mechanism helps you adapt the fix and avoid dead ends:

1. **`emitProjectChanged()` is not visual.** It only flags the project dirty for the "unsaved changes" prompt. It refreshes nothing.

2. **`requestBlocksUpdate()` → `emitWorkspaceUpdate()` syncs data but not the flyout.** It emits `BLOCKS_NEED_UPDATE`; the VM turns that into `emitWorkspaceUpdate()`; the GUI's `onWorkspaceUpdate` reloads the workspace XML (`clearWorkspaceAndLoadFromXml`). That *does* sync the new variable into the Blockly workspace variable map and redraws the script canvas — but it does **not** redraw the palette's variable/list flyout.

3. **The Variables/Lists category is a *dynamic* toolbox category** (`<category custom="VARIABLE">`). Its contents are generated by a callback that reads the workspace variable map **at flyout-show time**. Therefore the *static* toolbox XML string is identical before and after you add a variable.

4. **The GUI's toolbox refresh is diff-gated.** In scratch-gui's `blocks.jsx`, `componentDidUpdate` only calls `updateToolbox()` when `this.props.toolboxXML !== this._renderedToolboxXML`. Since the XML didn't change (step 3), `updateToolbox()` is skipped and the dynamic flyout is never re-read.

5. **Tab switching works** because the visibility-change branch of `componentDidUpdate` calls `refreshWorkspace()` + `updateToolbox()` *unconditionally* — which is the manual refresh we reproduce in the helper.

So the helper does what the GUI's own `updateToolbox()` does for this case: set `toolboxRefreshEnabled_ = true` and call `refreshToolboxSelection_()` on the live workspace.

## Gotchas

- **`toolboxRefreshEnabled_` must be true.** The XML reload from step (1) sets it to `false`; if you skip the flag, `refreshToolboxSelection_()` silently no-ops. This is the single most common reason a "refresh" attempt appears to do nothing.

- **Do it once, on creation only.** Look the variable up first and create only if missing; refresh only when you actually created it. Refreshing on every block run triggers needless workspace reloads and can disrupt the user mid-interaction.

- **Sprite-local variables only sync for the editing target.** `emitWorkspaceUpdate()` builds the workspace XML from `editingTarget.variables`. A sprite-local list created on a *non-editing* target will not show until that sprite is selected — which is the correct behavior anyway (sprite-local lists belong to that sprite's palette).

- **Keep it best-effort.** The helper touches scratch-blocks internals (`toolboxRefreshEnabled_`, `refreshToolboxSelection_`) and walks React fiber — inherently version-fragile. Guard `typeof document` and wrap in `try/catch` so a future editor change degrades to "appears on next tab switch" rather than throwing.

- **Pure block-model edits (scripts) don't need part (2).** Injecting blocks with `target.blocks.createBlock(...)` and then calling `emitWorkspaceUpdate()` / `requestBlocksUpdate()` re-renders the script canvas on its own. The extra flyout refresh is only needed for **dynamic categories** — variables, lists, and custom procedures.

## How to verify

In the running editor (e.g. via the `xcratch-extension-debug-auto` skill or `playwright-cli`), reproduce the worst case and assert the flyout updated **without a tab switch**:

```js
// after grabbing the scratch-blocks workspace `ws` (same fiber walk as the helper)
const flyoutLists = () => ws.getFlyout().getWorkspace().getAllBlocks()
    .filter(b => b.type === 'data_listcontents')
    .map(b => b.getField('LIST') && b.getField('LIST').getText());

ws.toolbox_.selectCategoryById('variables');   // open Variables (stale flyout)
// ... run the extension block that creates the list ...
flyoutLists();                                  // should now include your list name
```

Pass condition: the new list/variable name appears in `flyoutLists()` (or in the `.blocklyFlyout` DOM text) immediately after the creating block runs, with no Code-tab switch and no manual refresh.

## Reference: where this lives in the editor source

If you have a `scratch-editor` checkout and want to read the exact code paths:

- `scratch-gui/src/containers/blocks.jsx` — `onWorkspaceUpdate`, `componentDidUpdate` (the diff gate), `updateToolbox()` (sets `toolboxRefreshEnabled_ = true`).
- `scratch-gui/src/lib/make-toolbox-xml.js` — the `custom="VARIABLE"` dynamic category.
- `scratch-vm/src/engine/runtime.js` — `requestBlocksUpdate()` emits `BLOCKS_NEED_UPDATE`.
- `scratch-vm/src/virtual-machine.js` — listens for `BLOCKS_NEED_UPDATE` and calls `emitWorkspaceUpdate()`.
