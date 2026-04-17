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

All components use **gluestack-ui v4** for styling on both platforms.

**Use when:**

- Creating a new Wappa CMS project (web or mobile)
- Implementing a PageComponent render system
- Building a component registry from the Wappa schema
- Integrating `@appaflytech/wappa-client` SDK
- Setting up the AppContext, theme, or navigation

**Topics covered:**

| Sub-Skill                 | File                             | Coverage                                                           |
| ------------------------- | -------------------------------- | ------------------------------------------------------------------ |
| `wappa-skills`            | [SKILL.md](./SKILL.md)           | Main overview, mandatory rules, component table                    |
| `wappa-skills:components` | [components.md](./components.md) | Props interfaces + gluestack implementations for all 27 components |
| `wappa-skills:web`        | [web.md](./web.md)               | Next.js setup, GluestackUIProvider, routing, component registry    |
| `wappa-skills:mobile`     | [mobile.md](./mobile.md)         | Expo setup, WapScreen, contextService, Zustand store, registry     |
| `wappa-skills:theme`      | [theme.md](./theme.md)           | Wappa theme system + gluestack-ui v4 theming for web and mobile    |

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

- `@appaflytech/wappa-client` SDK
- **Web:** Next.js 14+ (App Router), `@gluestack-ui/nativewind-utils`
- **Mobile:** Expo SDK 51+, `@gluestack-ui/nativewind-utils`, NativeWind v4

## Supported Agents

Works with any agent that supports the Agent Skills specification, including:
GitHub Copilot, Claude Code, Cursor, Windsurf, Cline, OpenCode, Codex, and [40+ more](https://www.npmjs.com/package/skills#supported-agents).

## License

MIT
