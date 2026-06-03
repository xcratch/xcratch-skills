# xcratch-skills

Xcratch extension development skills for Copilot.

## Skill Structure

This repository uses a modular skill structure, with each skill focused on a specific aspect of Xcratch extension development:

- `xcratch-extension-create`: setup and scaffold workflow
- `xcratch-extension-debug`: local debug workflow
- `xcratch-extension-debug-auto`: autonomous agent debug via `?extension=` query parameter
- `xcratch-extension-stretch3`: stretch3 integration (creates install script and entry files)

## Which Skill To Use

- If the user needs a new extension repository, `xcratch-create`, `setup-dev`, or initial build, use `xcratch-extension-create`.
- If the user has breakpoint/source map/loading issues in local development, use `xcratch-extension-debug`.
- If the agent needs to autonomously navigate to `https://localhost:8601/?extension=` and verify the extension loads, use `xcratch-extension-debug-auto`.
- If the user wants to add an extension to stretch3, use `xcratch-extension-stretch3`.

## Routing Policy

- Detailed steps belong to specialized skills:
  - setup details -> `xcratch-extension-create`
  - debug details -> `xcratch-extension-debug`
  - autonomous browser debug -> `xcratch-extension-debug-auto`
  - stretch3 integration -> `xcratch-extension-stretch3`

## Related Files

- Create skill: `skills/xcratch-extension-create/SKILL.md`
- Debug skill: `skills/xcratch-extension-debug/SKILL.md`
- Auto debug skill: `skills/xcratch-extension-debug-auto/SKILL.md`
- stretch3 skill: `skills/xcratch-extension-stretch3/SKILL.md`
