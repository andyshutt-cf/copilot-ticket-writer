# docs/design

This directory holds Figma exports: PNG screen captures and Markdown component summaries.

## Purpose

Provides design artefacts that inform ticket writing — including screen designs, component specs, and annotated UI flows.

## Naming Convention

File names should **match the Figma layer path**, using kebab-case with `/` replaced by `--`:

```
[page]--[frame]--[component].[ext]
```

Examples:
- `checkout--payment-step--card-input.png`
- `checkout--payment-step--card-input.md`
- `onboarding--welcome-screen.png`

## File Types

| Extension | Contents |
|-----------|----------|
| `.png` | Exported screen or component from Figma |
| `.md` | Markdown summary of the component: purpose, states, props, accessibility notes |
