---
description: Create a demo environment for a Pi extension
---

Set up a demo directory for a Pi extension. Load the `demo-setup` skill first, then follow these steps:

## 1. Find the extension

If the current directory contains a `package.json` with a `pi` key, use it. Otherwise, ask the user for the path to the extension.

Read the extension's `package.json`, `README.md`, and source files to understand:
- What tools it registers
- What commands it provides
- What hooks it uses
- Whether it's a provider, theme, or standard extension

## 2. Create the demo directory

```bash
demo_dir=$(mktemp -d -t pi-demo-XXXXXX)
mkdir -p "$demo_dir/.pi/prompts"
```

## 3. Set up the demo

Follow the `demo-setup` skill to create:
- `.pi/settings.json` registering the extension by absolute path
- `.pi/prompts/demo.md` with steps covering each feature
- Fixture files appropriate for the extension type
- `AGENTS.md` if the agent needs behavioral instructions
- `.pi/extensions/<name>.json` if config overrides are needed

## 4. Print the result

Print the demo directory path and instructions for the user.
