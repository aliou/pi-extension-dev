---
name: demo-setup
description: Set up demo environments for Pi extensions to record videos or screenshots for the Pi package browser. Use when preparing a demo, recording a video, or creating preview assets for an extension or theme.
---

# Demo Setup

Set up self-contained demo directories for Pi extensions. The demo directory contains everything needed to showcase an extension: a prompt that runs through the features, fixture files, and pi configuration.

## Pi Package Browser

The Pi website has a package browser at `buildwithpi.ai/packages`. Packages can display preview media.

### Adding Preview Media to package.json

```json
{
  "pi": {
    "extensions": ["./index.ts"],
    "video": "https://example.com/demo.mp4",
    "image": "https://example.com/preview.png"
  }
}
```

Both `video` and `image` are optional. Video takes precedence when present.

### Video/Image Specs

- **Aspect ratio**: 16:9 (enforced by `aspect-ratio: 16/9` CSS)
- **Recommended resolution**: 1920x1080
- **Object fit**: `contain` on black background
- **Modal max width**: 900px
- **Format**: `.mp4` for video, `.png`/`.jpg`/`.jpeg`/`.webp`/`.gif` for image
- **Caching**: Client-side localStorage, 15-minute TTL. New visitors or cleared cache see updates immediately.

### For Themes

Themes show better as images (screenshots of dark and light variants side by side). A single 16:9 image with both variants is ideal.

## Demo Directory Structure

```
<demo-dir>/
├── .pi/
│   ├── settings.json              # Registers the extension as a local path
│   ├── prompts/
│   │   └── demo.md                # The /demo prompt
│   └── extensions/
│       └── <name>.json            # Extension config overrides (if needed)
├── AGENTS.md                      # Agent instructions (if needed)
└── <fixture files>                # Supporting files the demo needs
```

## Setup Workflow

### 1. Detect the Extension

Check `package.json` in the target directory for a `pi` key with `extensions`, `themes`, or `skills`. Read the README and source to understand what the extension provides: tools, commands, hooks, providers, themes.

### 2. Create Demo Directory

```bash
demo_dir=$(mktemp -d)
mkdir -p "$demo_dir/.pi/prompts"
```

### 3. Register the Extension

Create `.pi/settings.json` pointing to the extension's absolute path:

```json
{
  "packages": [
    "/absolute/path/to/extension"
  ],
  "defaultThinkingLevel": "off"
}
```

Use `defaultThinkingLevel: "off"` to keep responses fast and visible during demos.

### 4. Write the Demo Prompt

Create `.pi/prompts/demo.md`. Structure it as numbered steps, one feature per step. Each step should produce visible output.

```markdown
---
description: Showcase the <extension-name> extension
---

Demo the <extension-name> extension. Do each step one at a time.

## 1. <Feature Name>

<What to do and what it demonstrates.>

## 2. <Next Feature>

...
```

### 5. Add Fixture Files

Create any files the demo needs to function. Examples by extension type:

**Hook-based extensions** (e.g., guardrails):
- `.env` with fake secrets (to trigger env file protection)
- `.env.example` (to show allowed patterns)
- `.pi/extensions/<name>.json` with feature toggles enabled

**Tool-based extensions** (e.g., processes, linkup):
- Fake project files (package.json, scripts/) that the tools operate on
- `AGENTS.md` instructing the agent to use the extension's tools

**Provider extensions** (e.g., synthetic):
- No fixtures needed; demo is interactive (`/model`, send message, `/quotas`)

**Theme packages**:
- No fixtures needed; demo switches themes and writes a code file for syntax highlighting

### 6. Add AGENTS.md (if needed)

When the agent needs specific behavior during the demo (e.g., always use background processes, never run servers in foreground), add an `AGENTS.md`.

## Demo Prompt Patterns by Extension Type

### Extensions with Hooks

Trigger each hook by running commands that match hook patterns. Include both blocked and allowed cases.

### Extensions with Tools

Call each tool with representative inputs. Show the tool's output rendering.

### Extensions with Commands

Run each command (e.g., `/extension:command`) to show the interactive UI.

### Extensions with Providers

1. Switch to the provider: `/model <provider>`
2. Send a test message
3. Show provider-specific commands (quotas, usage, balance)
4. If the provider has tools (web search), use them
5. Switch back to default provider

### Themes

1. Switch to the theme: `/theme <name>`
2. Write a code file to show syntax highlighting
3. If there are variants (light/dark), switch between them

## Output

After creating the demo directory, print the path and instructions:

```
Demo ready at: <path>

  cd <path>
  pi

Then type /demo to start.
```
