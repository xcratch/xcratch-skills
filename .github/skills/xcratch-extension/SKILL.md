---
name: xcratch-extension
description: 'Use when the task involves Xcratch extension work and it is not yet clear whether the user needs project setup or debugging. Trigger phrases: Xcratch extension, xcratch-create, scaffold extension, setup-dev, breakpoints not hit, debug on dev-server, extension not loading.'
argument-hint: 'Describe whether you are creating a new extension or debugging an existing one, and include the current setup/problem.'
user-invocable: true
---

# Xcratch Extension Router

Route Xcratch extension requests to the right specialized skill.

## Use This Skill When

- The user says "Xcratch extension" but the task type is still ambiguous
- The request mixes setup and debug concerns
- You need a quick decision between scaffold/setup and debug workflows

## Routing

- For project setup, scaffolding, `setup-dev`, initial build, and GitHub Pages basics: use [../xcratch-extension-create/SKILL.md](../xcratch-extension-create/SKILL.md)
- For local VS Code debugging, certificates, scratch-editor tasks, launch configs, source maps, and extension load failures: use [../xcratch-extension-debug/SKILL.md](../xcratch-extension-debug/SKILL.md)

## Quick Decision Guide

- If the user needs to create or initialize an extension repository, choose `xcratch-extension-create`
- If the user already has an extension and needs to run, inspect, or troubleshoot it locally, choose `xcratch-extension-debug`
- If the user needs both, start with `xcratch-extension-create` and then continue with `xcratch-extension-debug`