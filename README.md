# Appaflytech Agent Skills

Agent skills for building projects with the **Wappa CMS** platform.
Follows the [Agent Skills](https://agentskills.io/) specification — works with GitHub Copilot, Claude Code, Cursor, Windsurf, and 40+ other AI agents.

## Install

```bash
npx skills add appaflytech/skills
```

Install globally (available across all projects):

```bash
npx skills add appaflytech/skills -g
```

Install to a specific agent:

```bash
npx skills add appaflytech/skills -a github-copilot
npx skills add appaflytech/skills -a claude-code
npx skills add appaflytech/skills -a cursor
```

## Available Skills

### `wappa-skills`

Complete guide for building **Next.js** (web) or **Expo React Native** (mobile) projects with Wappa CMS.

The component **props contract is framework-agnostic** — it is derived from the admin page-builder schema (`elements.ts` / `constants.ts`); you implement it with any UI library. Reference stacks: **wappa-web** uses plain **Tailwind CSS v3** (hand-written classes, Next.js 15 / React 19); **wappa-mobile** uses **gluestack-ui v3 primitives + NativeWind v4** (Expo SDK 57 / RN 0.86). *(Not gluestack-ui v4.)*

**Use when:**

- Creating a new Wappa CMS project (web or mobile)
- Implementing a PageComponent render system
- Building a component registry from the Wappa schema
- Integrating `@appaflytech/wappa-client` SDK
- Setting up the AppContext, theme, or navigation

**Topics covered:**

| Sub-Skill                 | File                             | Coverage                                                           |
| ------------------------- | -------------------------------- | ------------------------------------------------------------------ |
| `wappa-skills`            | [SKILL.md](./SKILL.md)           | Main overview, mandatory rules, component groups + full 127-name catalog |
| `wappa-skills:components` | [components.md](./components.md) | Props interfaces for all **127 components** (10 groups, from admin schema) |
| `wappa-skills:web`        | [web.md](./web.md)               | Next.js 15 setup, App Router, plain-Tailwind registry (wappa-web reference) |
| `wappa-skills:mobile`     | [mobile.md](./mobile.md)         | Expo SDK 57 setup, app + shared `@appaflytech/wappa-mobile-ui` package, WapScreen, Zustand store, registry |
| `wappa-skills:theme`      | [theme.md](./theme.md)           | Wappa theme system (web CSS vars + mobile ThemeProvider) for web and mobile |

## Usage

Once installed, skills are automatically loaded by your AI agent when relevant tasks are detected.

**Trigger examples:**

```
Create a Wappa web project
Build an Expo app with Wappa CMS
Add a Button component using the Wappa schema
Set up the PageComponent render system for mobile
```

**Load a specific sub-skill** (tell your agent):

```
Load wappa-skills:components and implement the Card component
Load wappa-skills:mobile and set up the WapScreen for Expo
```

## Prerequisites

- `@appaflytech/wappa-client` SDK (v0.0.11+)
- **Web:** Next.js 15 (App Router), React 19, Tailwind CSS v3 (reference uses plain Tailwind)
- **Mobile:** Expo SDK 57, React Native 0.86, `@appaflytech/wappa-mobile-ui`, gluestack-ui v3, NativeWind v4, Zustand v5, expo-router

## Supported Agents

Works with any agent that supports the Agent Skills specification, including:
GitHub Copilot, Claude Code, Cursor, Windsurf, Cline, OpenCode, Codex, and [40+ more](https://www.npmjs.com/package/skills#supported-agents).

## License

MIT
