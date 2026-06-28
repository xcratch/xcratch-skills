# xcratch-skills

Xcratch extension development skills for AI agents. Each skill is designed to handle a specific aspect of the extension development workflow, from initial setup to debugging and deployment. By modularizing the skills, we can provide targeted guidance and actions based on the user's current needs and context.

## Skill Structure

This repository uses a modular skill structure, with each skill focused on a specific aspect of Xcratch extension development:

- `xcratch-extension-create`: setup and scaffold workflow
- `xcratch-extension-debug`: debug workflow against the public `xcratch.github.io` editor
- `xcratch-extension-debug-auto`: autonomous agent debug via `?extension=` query parameter on the public `xcratch.github.io` editor (includes a `playwright-cli` config that auto-bypasses the self-signed cert and Chrome's Local Network Access prompt so the local extension loads without manual clicks)
- `xcratch-extension-stretch3`: stretch3 integration (creates install script and entry files)
- `xcratch-extension-palette-refresh`: force the editor block palette (variable/list/My Blocks flyout) to refresh after programmatic VM model changes

## Which Skill To Use

- If the user needs a new extension repository, `xcratch-create`, `setup-dev`, or initial build, use `xcratch-extension-create`.
- If the user has breakpoint/source map/loading issues debugging against the public `xcratch.github.io` editor, use `xcratch-extension-debug`.
- If the agent needs to autonomously navigate to `https://xcratch.github.io/editor/?extension=` and verify the extension loads, use `xcratch-extension-debug-auto`.
- If the user wants to add an extension to stretch3, use `xcratch-extension-stretch3`.
- If a programmatically created, deleted, or renamed variable/list/custom procedure does not show up in the block palette until a Code-tab or sprite switch, use `xcratch-extension-palette-refresh`.

## Routing Policy

- Detailed steps belong to specialized skills:
  - setup details -> `xcratch-extension-create`
  - debug details -> `xcratch-extension-debug`
  - autonomous browser debug -> `xcratch-extension-debug-auto`
  - stretch3 integration -> `xcratch-extension-stretch3`
  - palette/flyout refresh after programmatic variable/list/procedure changes -> `xcratch-extension-palette-refresh`

## Related Files

- Create skill: `skills/xcratch-extension-create/SKILL.md`
- Debug skill: `skills/xcratch-extension-debug/SKILL.md`
- Auto debug skill: `skills/xcratch-extension-debug-auto/SKILL.md`
- stretch3 skill: `skills/xcratch-extension-stretch3/SKILL.md`
- Palette refresh skill: `skills/xcratch-extension-palette-refresh/SKILL.md`
