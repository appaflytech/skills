---
name: wappa-skills
description: Complete guide for building Next.js (web) or Expo React Native (mobile) projects with Wappa CMS. Framework-agnostic — developer can use any UI library. Covers @appaflytech/wappa-client SDK, PageComponent render system, component registry, component schema contracts (derived from admin elements.ts / constants.ts), and default gluestack-ui v4 implementations. Ask the user which UI framework they want; default is gluestack-ui v4.
---

# Wappa Schema Infrastructure — Developer Guide

> **IMPORTANT**: This is a SKILL file, NOT a project. Never run `npm install` or `bun install` in this folder. When creating a new project, always ask the user for the project path or create it in a separate directory (e.g., `~/Projects/my-app`).

---

## MANDATORY REQUIREMENTS

Whenever the user says "create a wappa project" or "build an app with wappa", **always** do the following **before writing any code**:

### 1. Determine Project Type

**Ask:** Web (Next.js) or Mobile (Expo React Native)?

### 2. Choose UI Framework

**Ask:** Which UI framework do you want to use?

| Option                         | Web                       | Mobile                     |
| ------------------------------ | ------------------------- | -------------------------- |
| **gluestack-ui v4** ⭐ Default | ✅ NativeWind + RSC       | ✅ GluestackUIProvider     |
| shadcn/ui                      | ✅ Radix-based            | ❌ Not supported           |
| Tailwind CSS only              | ✅ Plain HTML + className | ❌ Not supported           |
| NativeWind only                | ❌                        | ✅ StyleSheet + NativeWind |
| Custom                         | Any                       | Any                        |

If the user does not specify → **use gluestack-ui v4**.

### 3. Required Files (Every Project)

**Web (Next.js):**

- [ ] `core/render.tsx` — Converts PageComponent tree to React
- [ ] `components/index.tsx` — Component registry (name → React component)
- [ ] `components/ui/` — Component implementations (gluestack-ui v4 primitives)
- [ ] `services/contextService.ts` — API data fetching (server)
- [ ] `app/[[...pathname]]/page.tsx` — SSR entry
- [ ] `app/[[...pathname]]/client.tsx` — AppContextProvider + render
- [ ] `.env.local` — `NEXT_PUBLIC_API`, `NEXT_PUBLIC_CDN`, `NEXT_PUBLIC_SITE_KEY`, `NEXT_PUBLIC_APP_URL`, `NEXT_PUBLIC_ENV`

**Mobile (Expo):**

- [ ] `core/render.tsx` — Converts PageComponent tree to React Native (with handler compilation)
- [ ] `components/index.tsx` — Component registry (90+ components)
- [ ] `components/WapScreen.tsx` — Page loading and render component
- [ ] `services/contextService.ts` — API data fetching service
- [ ] `store/store.ts` — Zustand global state
- [ ] `.env` — `EXPO_PUBLIC_WAP_API`, `EXPO_PUBLIC_WAP_CDN`, `EXPO_PUBLIC_WAP_SITE_KEY`, `EXPO_PUBLIC_ENV`

### 4. Component Implementation Philosophy

Components receive their **props contract** from the Wappa admin schema (defined in `elements.ts` and `constants.ts`). The implementation is framework-agnostic:

```
Admin Schema (constants.ts / elements.ts)
  ↓ defines what props the admin can configure
TypeScript Interface (components.md)
  ↓ you implement this interface with any UI framework
Your Component Implementation
  ↓ renders with gluestack-ui v4, shadcn/ui, plain HTML, or anything else
```

**Rule: Every component must accept the exact props defined in its schema contract. Ignore unknown props gracefully.**

### 5. Sub-Skills (Load for Detailed Implementation)

| Sub-Skill                 | File                             | When to Use                                                                                                   |
| ------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `wappa-skills:components` | [components.md](./components.md) | Full props contracts for all 90+ components (from admin schema). Load this before implementing any component. |
| `wappa-skills:web`        | [web.md](./web.md)               | Next.js project setup, routing, GluestackUIProvider for web                                                   |
| `wappa-skills:mobile`     | [mobile.md](./mobile.md)         | Expo project setup, WapScreen, contextService, mobile render                                                  |
| `wappa-skills:theme`      | [theme.md](./theme.md)           | Wappa theme system + gluestack-ui v4 theming                                                                  |

### 6. Setup Steps (Apply in Order)

1. Ask project type (Web / Mobile)
2. Ask UI framework (default: gluestack-ui v4)
3. Create project directory
4. Install `@appaflytech/wappa-client`
5. Create `.env` file
6. Create `core/render.tsx`
7. Create `components/index.tsx` (registry)
8. Implement all components (load `wappa-skills:components` first)
9. Create platform-specific entry files (page.tsx / WapScreen.tsx)
10. Test

---

## 1. What Is the Wappa Schema System?

Every page in Wappa is a **`PageComponent` tree**. Built via drag-and-drop in the admin panel, the tree is stored as JSON and delivered via the backend API. The frontend converts this JSON into real React components.

```
Admin UI (drag-drop page builder)
  ↓ PageComponent[] JSON
Backend API (Ui.API)
  ↓ GET /pages?path=...
wappa-client SDK (mapPage)
  ↓ resolved Page object
render() function
  ↓ React / React Native components (your implementation)
```

### Component Groups (from Admin Schema)

The admin organizes all components into 9 groups. These map to the tab panel in the builder:

| Group key     | Turkish Label | Component names                                                                                     |
| ------------- | ------------- | --------------------------------------------------------------------------------------------------- |
| `layout`      | Düzen         | container, array-repeater, box, center, hstack, vstack, grid, pressable, row, column, section, view |
| `typography`  | Metin         | heading, paragraph, html, icon                                                                      |
| `media`       | Medya         | image, video, iframe                                                                                |
| `interactive` | Butonlar      | button, link, fab                                                                                   |
| `display`     | Kartlar       | card, card-list, avatar, badge, divider, table, skeleton                                            |
| `feedback`    | Bildirim      | spinner, alert, progress, toast                                                                     |
| `disclosure`  | Sekmeler      | accordion, tabs                                                                                     |
| `overlay`     | Pop-up        | modal, drawer, actionsheet, menu, popover, alert-dialog, tooltip                                    |
| `form`        | Form          | form-control, input, select, switch, checkbox, radio, textarea, slider, calendar, date-time-picker  |

**`isMobile` field:** Each element has `isMobile: true | false`. Components with `isMobile: false` are web-only (e.g., `container`, `iframe`). On mobile, skip registering those or render `null`.

---

## 2. Core Types

### `PageComponent` — The Unit of Every Component

```ts
import { PageComponent } from "@appaflytech/wappa-client/constants";

type PageComponent = {
  id: string; // Unique ID on the page
  title: string; // Display name in admin
  name: string; // Component registry key (e.g. "heading", "button")
  icon: string; // Icon in Admin UI
  props: Record<string, any>; // Static props set by admin
  refs: Record<string, any>; // Resolved dynamic values (images, links, query results)
  children: PageComponent[]; // Child components (for droppable/wrapper components)
};
```

### `Page` — Full Page Object

```ts
import { Page } from "@appaflytech/wappa-client/services";

type Page = {
  id: number;
  title: string;
  theme: string;
  path: string;
  language: string;
  image: ImageProps;
  fields?: Record<string, any>; // DynamicEntity fields
  metatags: PageMetaTags;
  localizations: Array<PageAlternate>;
  layout: Array<PageComponent>; // Root-level components
  views: Record<string, Array<PageComponent>>; // Named view sections (key = view ID)
  refs: RefsObject;
};
```

### `RefsObject` — Resolved Reference Data

```ts
type RefsObject = {
  fields: Record<string, string>;
  files: Record<string, FileReference>;
  links: Record<string, LinkReference>;
  navigations: Record<string, NavigationReference[]>;
  page: PageReference;
  queries: Record<string, any>; // Query results
  queryOptions: Record<string, any>;
  componentqueries: Record<string, any>; // Parameter-based queries
  showcases: Record<string, any>;
  strings: Record<string, string>; // i18n translation strings
  widgets: Record<string, PageComponent[]>; // Widget schemas
};
```

### `EnvironmentContext` — API Configuration

```ts
type EnvironmentContext = {
  cdn: string; // CDN base URL
  environment: string; // "development" | "production"
  key: string; // Site key
  service: string; // API URL (base + "/" + siteKey)
  url: string; // Site domain URL
};
```

---

## 3. wappa-client SDK Setup

```bash
npm install @appaflytech/wappa-client
# or
bun add @appaflytech/wappa-client
```

### Subpath Exports

| Import Path                                 | Contents                                                                                                                 |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `@appaflytech/wappa-client/constants`       | `PageComponent`, `EnvironmentContext`, `ImageProps`                                                                      |
| `@appaflytech/wappa-client/constants/types` | All TypeScript types                                                                                                     |
| `@appaflytech/wappa-client/constants/enums` | `NodeEnv` enum                                                                                                           |
| `@appaflytech/wappa-client/services`        | `pageService`, `configService`, `siteService`, `queryService`                                                            |
| `@appaflytech/wappa-client/core/classes`    | `Environment` class                                                                                                      |
| `@appaflytech/wappa-client/core/components` | `ArrayRepeater`, `Error` components                                                                                      |
| `@appaflytech/wappa-client/core/contexts`   | `AppContextProvider`, `useApp`                                                                                           |
| `@appaflytech/wappa-client/core/hooks`      | `useClone`, `useMounted`, `useMobile`, `useOrientation`, `useSearchParams`, `useVar`, `useWidthResize`, `useCombineRefs` |
| `@appaflytech/wappa-client/core/utils`      | color, path, string utilities, handler compiler                                                                          |
| `@appaflytech/wappa-client/core/buses`      | `dataBus`, `formBus`, `uiBus` — reactive cross-component communication                                                   |

---

## 4. Environment Setup

```ts
import { Environment } from "@appaflytech/wappa-client/core/classes";

const env = new Environment();
env.update({
  cdn: process.env.NEXT_PUBLIC_CDN || "",
  key: "my-site-key",
  service: `${process.env.NEXT_PUBLIC_API}/my-site-key`,
  url: "",
  environment: "development",
});
```

---

## 5. Service Layer

```ts
import {
  pageService,
  configService,
  siteService,
} from "@appaflytech/wappa-client/services";

// Page by path
const page = await pageService.get(env.context, {
  path: "/home",
  isMobile: false,
});

// Preview mode
const page = await pageService.preview(env.context, previewId);

// Site config (themes, languages, settings)
// NOTE: Second argument is now an object, not a bare string
const config = await configService.get(env.context, { language: "en-us" });
// → { settings, themes, languages }

// Site info
const site = await siteService.get(env.context, siteKey);
```

---

## 6. Render System

### Web (Next.js) — `core/render.tsx`

```tsx
import React from "react";
import getComponent from "@/components";
import { PageComponent } from "@appaflytech/wappa-client/constants";

export const render = (
  components: PageComponent[],
  views: Record<string, PageComponent[]>,
): React.ReactNode => {
  if (!components?.length) return null;

  return components.map(({ id, name, props, refs, children }) => {
    if (name === "view") {
      const view = views[id];
      return view ? (
        <React.Fragment key={id}>{render(view, views)}</React.Fragment>
      ) : null;
    }

    const Component: any = getComponent(name);
    if (!Component) return null;

    // refs spread first, then props — static admin values take priority
    return children?.length ? (
      <Component key={id} {...refs} {...props}>
        {render(children, views)}
      </Component>
    ) : (
      <Component key={id} {...refs} {...props} />
    );
  });
};
```

### Mobile (Expo) — `core/render.tsx`

The mobile render system compiles **handler values** and normalizes styles before passing props to components. See `wappa-skills:mobile` for the complete implementation.

```tsx
import React from "react";
import { PageComponent } from "@appaflytech/wappa-client/constants/types";
import {
  createHandlerCompiler,
  isHandlerValue,
} from "@appaflytech/wappa-client/core/utils";
import { dataBus, formBus, uiBus } from "@appaflytech/wappa-client/core/buses";
import getComponent from "../components";

// Handler values look like: { __handler: true, code: "router.push('/')" }
// They are compiled into real functions at render time with full context
const compileHandler = createHandlerCompiler(getMobileHandlerContext);

function transformProps(props: Record<string, any>) {
  const compiled: Record<string, any> = {};
  for (const [k, v] of Object.entries(props ?? {})) {
    compiled[k] = isHandlerValue(v) ? compileHandler(v) : v;
  }
  return compiled;
}

export const render = (
  componentList: PageComponent[],
  views: Record<string, PageComponent[]>,
): React.ReactNode => {
  // ... see mobile.md for full implementation
};
```

**Render rules:**

- `props` = static values set in admin
- `refs` = dynamic resolved values (query results, images, links)
- `children` = child `PageComponent[]` → render recursively
- `refs` spread BEFORE `props` → admin static values always win
- Handler values (`{ __handler: true, code: "..." }`) are compiled to real functions
- `normalizeStyle()` strips legacy nested style shapes from admin-ui

---

## 7. Component Registry

Maps `name` string → your React component. One registry file, two flavors.

### Web — `components/index.tsx`

Use **direct imports** (not `next/dynamic`). Registry is a `Record<string, ComponentType<any>>` object. Components live in `components/ui/<name>/`.

```tsx
import type { ComponentType } from "react";
import { ArrayRepeater } from "@appaflytech/wappa-client/core/components";

// Layout
import Container from "./ui/container/Container";
import Box from "./ui/box/Box";
import Row from "./ui/row/Row";
import Column from "./ui/column/Column";
import Section from "./ui/section/Section";
// ... all other imports

const registry: Record<string, ComponentType<any>> = {
  container: Container,
  box: Box,
  row: Row,
  column: Column,
  section: Section,
  // ... all components
  "array-repeater": ArrayRepeater,
  "array-row": ArrayRepeater, // alias — same component
};

export default function getComponent(name: string): ComponentType<any> | null {
  return registry[name] ?? null;
}
```

**Important:** `array-row` is an alias for `array-repeater` — register both to the same component.

### Mobile — same pattern, direct imports, 90+ components (see `wappa-skills:mobile`)

---

## 8. gluestack-ui v4 — Default Implementation

When using gluestack-ui v4, generate base components with the CLI:

```bash
npx gluestack-ui@latest init -y
npx gluestack-ui@latest add --all -y
```

Docs: https://v4.gluestack.io/ui/docs/components/all-components

Place generated files in `components/ui/` (web) or `components/base-ui/ui/` (mobile).

**Critical gluestack-ui v4 rules:**

1. **Compound components** — `<Button>` requires `<ButtonText>` inside
2. **Semantic tokens** — use `text-foreground`, `bg-primary-500`, never raw colors
3. **Component props first** — use `size`, `variant`, `action` before `className`
4. **tva for custom variants** — never hardcode variant styles inline
5. **Check sub-components** — every component has documented sub-components in v4 docs

---

## 9. Component Schema Contracts

Load `wappa-skills:components` for complete TypeScript interfaces for all 90+ components. They are derived directly from the admin panel schemas (`elements.ts` + `constants.ts`).

All components receive `className?: string`, `componentId?: string`, and `style?: object` as global props.

Component `name` values used in the registry:

```
Layout:
  container (web only)   array-repeater   array-row (alias)
  box        center       hstack           vstack
  grid       pressable    row              column
  section    view         script/code-block

Typography:
  heading    paragraph    html (web only)  icon    text

Media:
  image      image-background    video
  iframe (web only)

Interactive:
  button     link         fab

Display:
  card       card-list    avatar           badge
  divider    table        skeleton

Feedback:
  spinner    alert        progress         toast

Disclosure:
  accordion  accordion-item   tabs         tab-panel

Overlay:
  modal      drawer       actionsheet      menu
  popover    alert-dialog tooltip          portal

Form:
  form-control  input      select          switch
  checkbox       radio      textarea        slider
  calendar       date-time-picker    date-picker
  otp-input      image-picker        file-picker
  phone-input    color-picker

Mobile-native only:
  mobile-view / view-native        safe-area-view
  keyboard-avoiding-view           scroll-view
  flat-list      section-list      virtualized-list
  refresh-control                  carousel
  bottomsheet (+ sub-components)
  tab-view       webview           lottie
  map-view       map-marker
  bar-chart      line-chart        pie-chart
  camera / barcode-scanner
  progress-ring  image-cropper     context-menu
  error-boundary
```
