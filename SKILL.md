---
name: wappa-skills
description: Complete guide for building Next.js (web) or Expo React Native (mobile) projects with Wappa CMS. Framework-agnostic — the component props contract comes from the admin schema; you may implement it with any UI library. Covers @appaflytech/wappa-client SDK, PageComponent render system, component registry, and component schema contracts (derived from admin elements.ts / constants.ts). Reference stacks: wappa-web uses plain Tailwind CSS; wappa-mobile uses gluestack-ui v3 primitives + NativeWind v4. Ask the user which UI framework they want.
---

# Wappa Schema Infrastructure — Developer Guide

> **IMPORTANT**: This is a SKILL file, NOT a project. Never run `npm install` or `bun install` in this folder. When creating a new project, always ask the user for the project path or create it in a separate directory (e.g., `~/Projects/my-app`).

---

## MANDATORY REQUIREMENTS

Whenever the user says "create a wappa project" or "build an app with wappa", **always** do the following **before writing any code**:

### 1. Determine Project Type

**Ask:** Web (Next.js) or Mobile (Expo React Native)?

### 2. Choose UI Framework

**Ask:** Which UI framework do you want to use? The props contract is framework-agnostic, so any of these works. The columns below reflect what the **reference implementations actually ship**:

| Option                                       | Web                       | Mobile                     |
| -------------------------------------------- | ------------------------- | -------------------------- |
| **Plain Tailwind CSS** ⭐ web reference      | ✅ hand-written `className` | ❌ (RN has no DOM)         |
| **gluestack-ui v3 + NativeWind v4** ⭐ mobile reference | ⚠️ possible               | ✅ what `wappa-mobile` uses |
| shadcn/ui                                    | ✅ Radix-based            | ❌ Not supported           |
| NativeWind only                              | ⚠️                        | ✅ StyleSheet + NativeWind |
| Custom                                       | Any                       | Any                        |

If the user does not specify → match the reference stacks: **web → plain Tailwind CSS (v3)**, **mobile → gluestack-ui v3 primitives + NativeWind v4**. (Note: the reference impls do NOT use gluestack-ui **v4** — older docs were wrong about this.)

### 3. Required Files (Every Project)

**Web (Next.js):**

- [ ] `core/render.tsx` — Converts PageComponent tree to React
- [ ] `components/index.tsx` — Component registry (name → React component)
- [ ] `components/ui/` — Component implementations (plain Tailwind in the web reference)
- [ ] `services/contextService.ts` — API data fetching (server)
- [ ] `app/[[...pathname]]/page.tsx` — SSR entry
- [ ] `app/[[...pathname]]/client.tsx` — AppContextProvider + render
- [ ] `.env.local` — `NEXT_PUBLIC_API`, `NEXT_PUBLIC_CDN`, `NEXT_PUBLIC_SITE_KEY`, `NEXT_PUBLIC_APP_URL`, `NEXT_PUBLIC_ENV`

**Mobile (Expo):**

- [ ] `core/render.tsx` — Converts PageComponent tree to React Native (with handler compilation)
- [ ] `components/index.tsx` — Component registry (127 components)
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
  ↓ renders with plain Tailwind, gluestack-ui, shadcn/ui, plain HTML, or anything else
```

**Rule: Every component must accept the exact props defined in its schema contract. Ignore unknown props gracefully.**

### 5. Sub-Skills (Load for Detailed Implementation)

| Sub-Skill                 | File                             | When to Use                                                                                                   |
| ------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `wappa-skills:components` | [components.md](./components.md) | Full props contracts for all 127 components (from admin schema). Load this before implementing any component. |
| `wappa-skills:web`        | [web.md](./web.md)               | Next.js 15 / React 19 project setup, App Router, plain-Tailwind registry (wappa-web reference)                |
| `wappa-skills:mobile`     | [mobile.md](./mobile.md)         | Expo SDK 57 setup, the app + shared `@appaflytech/wappa-mobile-ui` package model, WapScreen, handler-compiling render |
| `wappa-skills:theme`      | [theme.md](./theme.md)           | Wappa theme system (web CSS vars + mobile ThemeProvider, `mobileColors`/`mobileFontSizes`)                    |

### 6. Setup Steps (Apply in Order)

1. Ask project type (Web / Mobile)
2. Ask UI framework (reference default: web → plain Tailwind, mobile → gluestack-ui v3 + NativeWind v4)
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

The admin organizes all components into **10 groups**. Group keys live on each element's `group` field in `elements.ts`; the user-facing Turkish labels come from `ELEMENT_GROUPS` in `page-builder/PageBuilderHeader.tsx` (the group key is NOT the label — e.g. `typography` → "Metin"). These map to the tab panel in the builder:

| Group key     | Turkish Label | Component `name`s                                                                                                                                                                                                            |
| ------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `layout`      | Düzen         | container, array-repeater, box, center, hstack, vstack, grid, pressable, row, column, section, article, main, nav, aside, header, footer, ul, ol, li, view, mobile-view, safe-area-view, scroll-view, flat-list, keyboard-avoiding-view, status-bar, section-list, virtualized-list, swipeable, draggable-list |
| `typography`  | Metin         | heading, h1, h2, h3, h4, h5, h6, span, strong, em, blockquote, pre, code, time, text, html, icon                                                                                                                            |
| `media`       | Medya         | image, video, iframe, carousel, image-background, webview, lottie, map-view, map-marker, camera, barcode-scanner, audio, image-cropper                                                                                       |
| `interactive` | Butonlar      | button, link, fab, touchable-opacity, touchable-highlight, touchable-without-feedback, touchable-native-feedback, touchable-link                                                                                             |
| `display`     | Kartlar       | card, card-list, avatar, badge, divider, table, skeleton, bar-chart, line-chart, pie-chart                                                                                                                                  |
| `feedback`    | Bildirim      | spinner, welcome-onboarding, alert, progress, toast, refresh-control, error-boundary, progress-ring                                                                                                                         |
| `disclosure`  | Sekmeler      | accordion, accordion-item, tabs, tab-panel, tab-view                                                                                                                                                                        |
| `overlay`     | Pop-up        | modal, drawer, actionsheet, menu, popover, alert-dialog, tooltip, bottomsheet, portal, context-menu                                                                                                                         |
| `form`        | Form          | text-input, form-control, input, select, switch, checkbox, radio, textarea, slider, calendar, date-time-picker, input-accessory-view, date-picker, otp-input, image-picker, file-picker, phone-input, multi-select, segmented-control, color-picker |
| `logic`       | Mantık        | script, data-fetch, websocket, notification, deep-link (invisible logic/data components)                                                                                                                                    |

> **Count:** the schema defines **130 element entries → 127 unique component `name`s** (`column` is registered 4× with different default sizes 12/6/4/3, same `name`). This is the number to trust; older docs said "90+" or "27".

**`isMobile` field:** Each element has `isMobile: true | false`. Components with `isMobile: false` are **web-only**: the `column` size variants, `article`, `main`, `nav`, `aside`, `header`, `footer`, `ul`, `ol`, `li`, `h1`–`h6`, `span`, `strong`, `em`, `blockquote`, `pre`, `code`, `time`, `iframe`, `calendar`, `date-time-picker`. On mobile, skip registering those or render `null`. Many mobile-only entries also carry `isReactNativePrimitive: true` (a native fast-path renderer).

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

### Mobile — same `Record<string, ComponentType>` pattern

In the reference mobile stack the registry lives in the shared package `@appaflytech/wappa-mobile-ui` (`components/index.tsx`, ~130 registry keys → ~90 modules, 84 component dirs), and the app extends it via `registerComponents(...)`. `array-repeater` is imported from `@appaflytech/wappa-client/core/components`. See `wappa-skills:mobile`.

---

## 8. Reference UI Implementations

The props contract is framework-agnostic; pick any UI library. What the two reference apps actually ship:

### Web — `wappa-web` (plain Tailwind CSS)

- **No gluestack, no NativeWind.** Next.js 15 + React 19. Each component in `components/ui/<name>/` is a thin wrapper that maps schema props to hand-written Tailwind utility strings and renders real DOM (`div`, `button`, `a`). Icons via `lucide-react`.
- Example: `Button` maps `action`/`variant`/`size` to class strings by hand; renders `<a>` when `anchor.href` is set, else `<button onClick>`.

### Mobile — `wappa-mobile` (gluestack-ui **v3** + NativeWind v4)

- Expo SDK 57, RN 0.86, React 19. Components use gluestack v3 primitives (`@gluestack-ui/core`, `createButton`/`tva`/`withStyleContext`/`cssInterop`) styled through NativeWind `className`. Icons via `lucide-react-native`. Dark mode = `class`.
- If you generate gluestack base components, follow gluestack **v3** conventions (compound components like `<Button><ButtonText/>`, NativeWind semantic classes). Do **not** assume gluestack v4 APIs (`config=` prop, `@gluestack-ui/config` tokens) — the reference does not use them.
- Generated UI lives in `components/ui/` (shared package `@appaflytech/wappa-mobile-ui`), not `components/base-ui/ui/`.

**Universal rules regardless of framework:** every component must accept the schema props for its `name` (see `wappa-skills:components`), accept the global props `className` / `componentId` / `style`, and ignore unknown props gracefully.

---

## 9. Component Schema Contracts

Load `wappa-skills:components` for complete TypeScript interfaces for all **127 components**. They are derived directly from the admin panel schema (`page-builder/definitions/elements.ts` + `constants.ts`).

Every element appends `GLOBAL_PROPS`: `componentId?: string` (unique id used by `form.getValue("id")`, `data.set("id", v)`, `context.ui.open("id")`), `className?: string` (Tailwind), `style?: object` (RN style) — except a handful of invisible/native components (`script`, `data-fetch`, `websocket`, `notification`, `deep-link`, `status-bar`, `map-marker`, `refresh-control`) that omit some or all globals.

The complete registry `name` list, by group (127 unique names; `column` has 4 default-size variants under one name):

```
layout (Düzen):
  container* array-repeater box center hstack vstack grid pressable row column
  section article* main* nav* aside* header* footer* ul* ol* li* view mobile-view
  safe-area-view scroll-view flat-list keyboard-avoiding-view status-bar
  section-list virtualized-list swipeable draggable-list

typography (Metin):
  heading h1* h2* h3* h4* h5* h6* span* strong* em* blockquote* pre* code* time*
  text html* icon

media (Medya):
  image video iframe* carousel image-background webview lottie map-view map-marker
  camera barcode-scanner audio image-cropper

interactive (Butonlar):
  button link fab touchable-opacity touchable-highlight touchable-without-feedback
  touchable-native-feedback touchable-link

display (Kartlar):
  card card-list avatar badge divider table skeleton bar-chart line-chart pie-chart

feedback (Bildirim):
  spinner welcome-onboarding alert progress toast refresh-control error-boundary progress-ring

disclosure (Sekmeler):
  accordion accordion-item tabs tab-panel tab-view

overlay (Pop-up):
  modal drawer actionsheet menu popover alert-dialog tooltip bottomsheet portal context-menu

form (Form):
  text-input form-control input select switch checkbox radio textarea slider
  calendar* date-time-picker* input-accessory-view date-picker otp-input image-picker
  file-picker phone-input multi-select segmented-control color-picker

logic (Mantık — invisible logic/data components):
  script data-fetch websocket notification deep-link
```

`*` = **web-only** (`isMobile: false`). The mobile registry additionally aliases `array-row` → ArrayRepeater and `view`/`container`/`column`/`section`/`html`/`iframe` → Box; see `wappa-skills:mobile`.
