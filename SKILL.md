---
name: wappa-skills
description: Complete guide for building Next.js (web) or Expo React Native (mobile) projects with Wappa CMS. All components use gluestack-ui v4 for styling on both platforms. Covers @appaflytech/wappa-client SDK, PageComponent render system, component registry, AppContext, and theme system.
---

# Wappa Schema Infrastructure — Comprehensive Developer Guide

> **IMPORTANT**: This is a SKILL file, NOT a project. Never run `npm install` or `bun install` in this folder. When creating a new project, always ask the user for the project path or create it in a separate directory (e.g., `~/Projects/my-app`).

This document is the complete guide for setting up a new **Next.js (web) or Expo (mobile)** project using the Wappa CMS schema infrastructure.

---

## MANDATORY REQUIREMENTS

Whenever the user says "create a wappa project" or "build an app with wappa", **always** do the following:

### 1. Determine Project Type

**Ask the user:** Web (Next.js) or Mobile (Expo React Native)?

### 2. UI Library — Always gluestack-ui v4

| Platform      | UI Library      | Notes                                             |
| ------------- | --------------- | ------------------------------------------------- |
| Web (Next.js) | gluestack-ui v4 | `@gluestack-ui/nativewind-utils`, NativeWind, RSC |
| Mobile (Expo) | gluestack-ui v4 | `GluestackUIProvider` + NativeWind                |

**Never use:** shadcn/ui, Radix UI, Tamagui, plain RN primitives (`<View>`, `<Text>`, `<div>`) when a gluestack component exists.

### 3. gluestack-ui v4 Rules (Always Apply)

1. **Gluestack components over primitives** — Use `<Box>` not `<div>`, `<Text>` not `<Text>` from RN
2. **Semantic tokens ONLY** — `text-foreground`, `bg-primary-500`, `border-outline-200`. Never `red-500`, `gray-600`
3. **Component props over className** — Use `size`, `variant`, `action` props first, then className utilities
4. **tva for custom variants** — Never hardcode variant styles inline
5. **Compound components always** — `<Button><ButtonText>` not `<Button text="...">`
6. **Read gluestack docs** before implementing any new component

### 4. Required Files (Every Project)

**Web:**

- [ ] `core/render.tsx` — Converts PageComponent tree to React
- [ ] `components/index.tsx` — Component registry (name → React component)
- [ ] `components/wap/` — All core component implementations
- [ ] `app/[[...pathname]]/page.tsx` — SSR entry
- [ ] `app/[[...pathname]]/context.tsx` — Data fetching (server)
- [ ] `app/[[...pathname]]/client.tsx` — AppContextProvider + render
- [ ] `.env.local` — API URL, CDN, site key

**Mobile:**

- [ ] `core/render.tsx` — Converts PageComponent tree to React Native
- [ ] `components/index.tsx` — Component registry
- [ ] `components/wap/` — All core component implementations (RN)
- [ ] `components/WapScreen.tsx` — Page loading and render component
- [ ] `components/ThemeProvider.tsx` — Theme system
- [ ] `services/contextService.ts` — API data fetching service
- [ ] `store/store.ts` — Zustand global state
- [ ] `utils/path.ts` — CDN URL helper
- [ ] `.env` — API URL, CDN

### 5. Sub-Skills (Load for Detailed Implementation)

| Sub-Skill                 | File                             | When to Use                                                            |
| ------------------------- | -------------------------------- | ---------------------------------------------------------------------- |
| `wappa-skills:components` | [components.md](./components.md) | Full props reference + gluestack implementations for all 27 components |
| `wappa-skills:web`        | [web.md](./web.md)               | Next.js project setup, routing, GluestackUIProvider for web            |
| `wappa-skills:mobile`     | [mobile.md](./mobile.md)         | Expo project setup, ThemeProvider, WapScreen, contextService           |
| `wappa-skills:theme`      | [theme.md](./theme.md)           | Wappa theme system + gluestack-ui v4 theming (web + mobile)            |

### 6. Setup Steps (Apply in Order)

1. Create project (Next.js or Expo)
2. Install `@appaflytech/wappa-client`
3. Create `.env` file
4. Create `core/render.tsx`
5. Create `components/index.tsx` (registry)
6. Create all core components
7. Create platform-specific entry files (page.tsx / WapScreen.tsx)
8. Test

---

## 1. What Is the Wappa Schema System?

In Wappa, every page is represented as **`PageComponent` trees**. Built via drag-and-drop in the admin panel (wappa-admin-ui), these trees are delivered as JSON from the backend API. The frontend's job is to convert this JSON into real React / React Native components.

```
Admin UI (drag-drop builder)
  ↓ PageComponent[] JSON
Backend API (Ui.API port 5001)
  ↓ GET /pages/path?path=...
wappa-client SDK (mapPage)
  ↓ resolved Page object
render() function
  ↓ React / React Native components
```

---

## 2. Core Types

### `PageComponent` — The Unit of Every Component

```ts
import { PageComponent } from "@appaflytech/wappa-client/constants";

type PageComponent = {
  id: string; // Unique ID
  title: string; // Display name in admin
  name: string; // Component registry key (e.g. "heading", "button")
  icon: string; // Icon in Admin UI
  props: Record<string, any>; // Static props
  refs: Record<string, any>; // Resolved references (query results, images, links, etc.)
  children: PageComponent[]; // Child components
};
```

### `Page` — Full Page Object

```ts
import { Page } from "@appaflytech/wappa-client/services";

type Page = {
  id: number;
  title: string;
  theme: string; // Theme ID
  path: string; // URL path
  language: string;
  image: ImageProps;
  fields?: Record<string, any>; // DynamicEntity fields
  metatags: PageMetaTags;
  localizations: Array<PageAlternate>;
  layout: Array<PageComponent>; // Page layout components
  views: Record<string, Array<PageComponent>>; // View sections (key = view ID)
  refs: RefsObject; // Raw reference data
};
```

### `RefsObject` — Resolved References

```ts
type RefsObject = {
  fields: Record<string, string>; // Meta field values
  files: Record<string, FileReference>; // File references
  links: Record<string, LinkReference>; // Link references
  navigations: Record<string, NavigationReference[]>; // Navigation/menu data
  page: PageReference; // Page info (title, path, image)
  queries: Record<string, any>; // Query results
  queryOptions: Record<string, any>;
  componentqueries: Record<string, any>; // Parameter-based queries
  showcases: Record<string, any>; // Showcase data
  strings: Record<string, string>; // Translation strings
  widgets: Record<string, PageComponent[]>; // Widget schemas
};
```

### `EnvironmentContext` — Service Configuration

```ts
type EnvironmentContext = {
  cdn: string; // CDN base URL
  environment: string; // "development" | "production"
  key: string; // Site key
  service: string; // API service URL (base + "/siteKey")
  url: string; // Site domain URL
};
```

---

## 3. wappa-client SDK Setup

### Package Installation

```bash
# npm
npm install @appaflytech/wappa-client

# bun
bun add @appaflytech/wappa-client
```

### Subpath Exports

| Import Path                                 | Contents                                                      |
| ------------------------------------------- | ------------------------------------------------------------- |
| `@appaflytech/wappa-client/constants`       | `PageComponent`, `EnvironmentContext`, `ImageProps` etc.      |
| `@appaflytech/wappa-client/constants/types` | All TypeScript types                                          |
| `@appaflytech/wappa-client/constants/enums` | `NodeEnv` enum                                                |
| `@appaflytech/wappa-client/services`        | `pageService`, `configService`, `siteService`, `queryService` |
| `@appaflytech/wappa-client/core/classes`    | `Environment` class                                           |
| `@appaflytech/wappa-client/core/components` | `ArrayRepeater`, `Error` components                           |
| `@appaflytech/wappa-client/core/contexts`   | `AppContextProvider`, `useApp`                                |
| `@appaflytech/wappa-client/core/hooks`      | `useClone`, `useMounted`, `useMobile`                         |
| `@appaflytech/wappa-client/core/utils`      | color, path, string utilities                                 |

---

## 4. Environment Setup

### `Environment` Class Usage

```ts
import { Environment } from "@appaflytech/wappa-client/core/classes";

const env = new Environment();

// Development
env.update({
  cdn: process.env.EXPO_PUBLIC_WAP_CDN || "",
  key: "my-site-key",
  service: `${process.env.EXPO_PUBLIC_WAP_API}/my-site-key`,
  url: "",
  environment: "development",
});

// Production (site info fetched from API)
const site = await siteService.get(env.context, siteKey);
env.update({
  cdn: process.env.EXPO_PUBLIC_WAP_CDN || "",
  key: site.key,
  service: `${site.service}/${site.key}`,
  url: site.url,
  environment: "production",
});
```

### Environment Variables

#### Next.js (wappa-web)

```env
NEXT_PUBLIC_API=https://api.your-service.com
NEXT_PUBLIC_CDN=https://cdn.your-service.com
NEXT_PUBLIC_APP_URL=https://your-site.com
NEXT_PUBLIC_ENV=development
```

#### Expo (wappa-mobile)

```env
EXPO_PUBLIC_WAP_API=https://api.your-service.com
EXPO_PUBLIC_WAP_CDN=https://cdn.your-service.com
EXPO_PUBLIC_ENV=development
```

---

## 5. Service Layer

### pageService — Page Data

```ts
import { pageService } from "@appaflytech/wappa-client/services";

// Fetch page by path
const page = await pageService.get(environment.context, {
  path: "/home",
  isMobile: false, // true for mobile
});

// Preview mode
const page = await pageService.preview(environment.context, previewId);
```

### configService — Site Configuration

```ts
import { configService } from "@appaflytech/wappa-client/services";

// Theme, language, settings
const config = await configService.get(environment.context, language);
// → { settings, themes, languages }
```

### siteService — Site Info

```ts
import { siteService } from "@appaflytech/wappa-client/services";

const site = await siteService.get(environment.context, siteKey);
// → { key, service, url, ... }
```

---

## 6. Render System

### Web (Next.js) — `core/render.tsx`

```tsx
import React from "react";
import nodes from "@/components"; // Component registry
import { PageComponent } from "@appaflytech/wappa-client/constants";

export const render = (
  components: PageComponent[],
  views: Record<string, PageComponent[]>,
) => {
  if (!components?.length) return null;

  return components.map((component) => {
    const { id, name, props, refs, children } = component;

    if (name === "view") {
      // View reference — fetch from views dictionary
      const view = views[id];
      if (view) {
        return <React.Fragment key={id}>{render(view, views)}</React.Fragment>;
      }
    } else {
      const Component: any = nodes(name);
      if (Component) {
        return children?.length ? (
          <Component key={id} {...refs} {...props}>
            {render(children, views)}
          </Component>
        ) : (
          <Component key={id} {...refs} {...props} />
        );
      }
    }
  });
};
```

### Mobile (React Native) — `core/render.tsx`

```tsx
import React from "react";
import { View } from "react-native";
import { PageComponent } from "@appaflytech/wappa-client/constants/types";
import components from "../components";

export const render = (
  componentList: PageComponent[],
  views: Record<string, PageComponent[]>,
  isMappingRender: boolean = false,
): React.ReactNode => {
  if (!componentList?.length) return null;

  return componentList.map((component, index) => {
    const { id, name, props, refs, children } = component;

    if (name === "view") {
      const view = views[id];
      if (view) {
        return <View key={id || index}>{render(view, views, false)}</View>;
      }
    } else {
      const Component = components(name);
      if (Component) {
        // isMappingRender: inside array-repeater, mappedValue takes priority
        const mappingProps = isMappingRender ? props?.mappedValue || {} : {};
        const combinedProps = { ...refs, ...mappingProps, ...props };

        return children?.length ? (
          <Component key={id || index} {...combinedProps}>
            {render(children, views, true)}
          </Component>
        ) : (
          <Component key={id || index} {...combinedProps} />
        );
      } else {
        console.warn(`Unknown component: ${name}`);
        return (
          <View
            key={id || index}
            style={{ padding: 10, backgroundColor: "#ffeeee" }}
          >
            {children?.length ? render(children, views, false) : null}
          </View>
        );
      }
    }
    return null;
  });
};
```

**Key rule:**

- `props` = static values (set in admin)
- `refs` = dynamic/resolved values (query results, images, links)
- `children` = child `PageComponent[]` (recursive render)

---

## 7. Component Registry

The component registry maps a `name` string to the actual React component.

### Web — `components/index.tsx`

```tsx
import dynamic from "next/dynamic";
import { ComponentType } from "react";
import { ArrayRepeater } from "@appaflytech/wappa-client/core/components";

// Core components — lazy load
const Container = dynamic(() => import("./wap/gridview/container"));
const Row = dynamic(() => import("./wap/gridview/row"));
const Column = dynamic(() => import("./wap/gridview/column"));
const Heading = dynamic(() => import("./wap/heading"));
const Paragraph = dynamic(() => import("./wap/paragraph"));
const Html = dynamic(() => import("./wap/html"));
const Image = dynamic(() => import("./wap/image"));
const Video = dynamic(() => import("./wap/video"));
const Button = dynamic(() => import("./wap/button"));
const Section = dynamic(() => import("./wap/section"));
const Anchor = dynamic(() => import("./wap/anchor"));
const Iframe = dynamic(() => import("./wap/iframe"));

// Widget components (custom)
const MyWidget = dynamic(() => import("./widgets/my-widget"));

export default function getter(name: string): ComponentType | null {
  switch (name) {
    case "container":
      return Container;
    case "row":
      return Row;
    case "column":
      return Column;
    case "heading":
      return Heading;
    case "paragraph":
      return Paragraph;
    case "html":
      return Html;
    case "image":
      return Image;
    case "video":
      return Video;
    case "button":
      return Button;
    case "section":
      return Section;
    case "link":
      return Anchor;
    case "iframe":
      return Iframe;
    case "array-repeater":
    case "array-row":
      return ArrayRepeater;
    case "my-widget":
      return MyWidget;
    default:
      return null;
  }
}
```

### Mobile — `components/index.tsx`

```tsx
import React from "react";
import { View } from "react-native";
import { ArrayRepeater } from "@appaflytech/wappa-client/core/components";

// Core components
import Container from "./wap/container/Container";
import Row from "./wap/row/Row";
import Column from "./wap/column/Column";
import Heading from "./wap/heading/Heading";
import Paragraph from "./wap/paragraph/Paragraph";
import Html from "./wap/html/Html";
import Image from "./wap/image/Image";
import Video from "./wap/video/Video";
import Button from "./wap/button/Button";
import Section from "./wap/section/Section";
import Anchor from "./wap/anchor/Anchor";

// Custom widgets
import MyWidget from "./widgets/my-widget/MyWidget";

export default function getComponent(
  name: string,
): React.ComponentType<any> | null {
  switch (name) {
    case "container":
      return Container;
    case "row":
      return Row;
    case "column":
      return Column;
    case "heading":
      return Heading;
    case "paragraph":
      return Paragraph;
    case "html":
      return Html;
    case "image":
      return Image;
    case "video":
      return Video;
    case "button":
      return Button;
    case "section":
      return Section;
    case "link":
    case "anchor":
      return Anchor;
    case "array-repeater":
      return ArrayRepeater as any;
    case "my-widget":
      return MyWidget;
    default:
      return null;
  }
}
```

---

## 8. Available Core Components

> For full props + gluestack implementation code, see **[wappa-skills:components](./components.md)**.

### Layout Components

| Schema Name      | gluestack Component         | Key Props                                                       |
| ---------------- | --------------------------- | --------------------------------------------------------------- |
| `container`      | `Box` (max-width wrapper)   | `size`: `fluid/extended/wide/medium/narrow`, `className`        |
| `row`            | `HStack` / responsive `Box` | `xs/sm/md/lg/xl`: `{ direction, align, justify, wrap, gutter }` |
| `column`         | `Box` (flex fraction)       | `xs/sm/md/lg/xl`: `{ size: 1-12, align, offset, order }`        |
| `section`        | `Box` as `<section>`        | `title`, `description`, `anchor`, `container`                   |
| `view`           | view dict lookup            | ID-based, no props                                              |
| `array-repeater` | SDK `ArrayRepeater`         | `repeater` (query ref)                                          |

### Content Components

| Schema Name | gluestack Component           | Key Props                                                      |
| ----------- | ----------------------------- | -------------------------------------------------------------- |
| `heading`   | `Heading`                     | `children`, `nodeType(h1-h6)`, `size(xs-5xl)`, `numberOfLines` |
| `paragraph` | `Text`                        | `children`, `size(2xs-2xl)`, `bold`, `italic`, `isTruncated`   |
| `html`      | `div` / `RenderHTML`          | `content` (HTML string)                                        |
| `image`     | `Image`                       | `source{src,alt}`, `size(2xs-full)`, `resizeMode`, `rounded`   |
| `video`     | native `video` / Expo `Video` | `source{src}`                                                  |
| `iframe`    | `iframe` (web only)           | `children` (URL string)                                        |

### Interactive Components

| Schema Name | gluestack Component     | Key Props                                                                                                                              |
| ----------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `button`    | `Button` + `ButtonText` | `children`, `anchor{href,target}`, `action(primary/secondary/positive/negative/default)`, `variant(solid/outline/link)`, `size(xs-xl)` |
| `link`      | `Link` + `LinkText`     | `source{href,target,title}`, droppable children                                                                                        |

### Display Components

| Schema Name | gluestack Component                             | Key Props                                                                                                         |
| ----------- | ----------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `card`      | `Card`                                          | `title`, `subtitle`, `description`, `image`, `anchor`, `size(sm/md/lg)`, `variant(ghost/outline/filled/elevated)` |
| `card-list` | `FlatList` + `Card`                             | `cards[]` + column group props                                                                                    |
| `avatar`    | `Avatar` + `AvatarImage` + `AvatarFallbackText` | `source`, `name`, `size(xs-2xl)`, `showBadge`                                                                     |
| `badge`     | `Badge` + `BadgeText`                           | `text`, `action(info/success/warning/error/muted)`, `variant(solid/outline)`, `size(sm/md/lg)`                    |
| `divider`   | `Divider`                                       | `orientation(horizontal/vertical)`                                                                                |

### Feedback Components

| Schema Name | gluestack Component                 | Key Props                                                                                |
| ----------- | ----------------------------------- | ---------------------------------------------------------------------------------------- |
| `spinner`   | `Spinner`                           | `size(small/large)`, `color`                                                             |
| `alert`     | `Alert` + `AlertIcon` + `AlertText` | `text`, `action(info/success/warning/error/muted)`, `variant(solid/outline)`, `showIcon` |
| `progress`  | `Progress` + `ProgressFilledTrack`  | `value(0-100)`, `size(xs-xl)`, `showLabel`, `label`                                      |

### Form Components

| Schema Name | gluestack Component                          | Key Props                                                                                                                              |
| ----------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `input`     | `FormControl` + `Input` + `InputField`       | `label`, `placeholder`, `type(text/password)`, `size`, `variant(outline/underlined/rounded)`, `helperText`, `isDisabled`, `isRequired` |
| `select`    | `FormControl` + `Select` compound            | `label`, `placeholder`, `options[]`, `size`, `variant`, `isDisabled`                                                                   |
| `switch`    | `Switch`                                     | `label`, `size(sm/md/lg)`, `isDisabled`                                                                                                |
| `checkbox`  | `Checkbox` compound                          | `label`, `options[]`, `size(sm/md/lg)`, `isDisabled`                                                                                   |
| `radio`     | `RadioGroup` + `Radio` compound              | `options[]`, `size(sm/md/lg)`, `isDisabled`                                                                                            |
| `textarea`  | `FormControl` + `Textarea` + `TextareaInput` | `label`, `placeholder`, `size`, `numberOfLines`, `helperText`, `isDisabled`, `isReadOnly`                                              |
| `slider`    | `Slider` compound                            | `value`, `minValue`, `maxValue`, `step`, `size(sm/md/lg)`, `showLabel`, `isDisabled`                                                   |

---

## 9. Creating a New Component

> All new components must use gluestack-ui v4. See **[wappa-skills:components](./components.md)** for full examples.

### Web Component — `components/wap/my-component/index.tsx`

```tsx
import { Box } from "@/components/ui/box";
import { Text } from "@/components/ui/text";

interface MyProps {
  title?: string;
  children?: React.ReactNode;
  className?: string;
}

export default function MyComponent({ title, children, className }: MyProps) {
  return (
    <Box className={className}>
      {title && <Text bold>{title}</Text>}
      {children}
    </Box>
  );
}
```

### Mobile Component — `components/wap/my-component/MyComponent.tsx`

````tsx
import { Box } from "@/components/ui/box";
import { Text } from "@/components/ui/text";
import { safeText } from "../../utils/path";

interface MyProps {
  title?: string;
  children?: React.ReactNode;
  className?: string;
}

const MyComponent: React.FC<MyProps> = ({ title, children, className }) => {
  return (
    <Box className={`w-full ${className || ""}`}>
      {title && <Text bold>{safeText(title)}</Text>}
      {children}
    </Box>
  );
};

export default MyComponent;

---

## 10. ArrayRepeater — Dynamic List

`array-repeater` appears as "Dynamic List" in admin. It repeats the child schema for each row of a query.

### Props

```ts
// comes via refs.repeater or props.repeater
{
  repeater: {
    type: 'query' | 'raw' | 'showcase',
    ref: string,         // query ID
    mappings: Array<{
      name: string,      // abstract name (e.g. "title")
      path: string,      // query column path (e.g. "Name")
      type: string
    }>
  }
}
````

### Admin Schema Definition (elements.ts)

```ts
{
  name: 'array-repeater',
  id: 'array-repeater',
  title: 'Dynamic List',
  droppable: true,
  isMobile: true,
  definition: {
    props: [{
      name: 'repeater',
      title: 'Data Source (Repeater)',
      type: 'text',
    }]
  }
}
```

### Usage — Child Component Mapping

Child components (e.g. `heading`) are bound to abstract names as follows:

```json
{
  "name": "heading",
  "props": {
    "mappedValue": {}
  },
  "refs": {
    "children": {
      "type": "repeater",
      "ref": "title"
    }
  }
}
```

---

## 11. References (Refs) System

Each value inside `refs` is a `Reference` object, resolved by the SDK mapper:

```ts
type Reference = {
  type: ReferenceType; // 'query' | 'link' | 'image' | 'field' | 'widget' | 'showcase' | ...
  ref: string; // ID / key
  label: string;
  mappings: ReferenceMapping[];
  parameters?: ReferenceParameter[];
};
```

### Reference Types

| type         | Description                               |
| ------------ | ----------------------------------------- |
| `static`     | Static value — use directly               |
| `field`      | Meta/DynamicEntity field                  |
| `link`       | Link href value                           |
| `navigation` | Navigation data (`NavigationReference[]`) |
| `query`      | Query result (processed via `mapQuery`)   |
| `raw`        | Raw query result                          |
| `showcase`   | Showcase data                             |
| `widget`     | Widget schema (`PageComponent[]`)         |
| `image`      | Image URL                                 |
| `file`       | File info (path, mimetype, size)          |
| `page`       | Page info (title, path, image)            |
| `string`     | Translation string                        |
| `repeater`   | array-repeater row data                   |

### Manual Reference Resolution

```ts
import { getReference } from "@appaflytech/wappa-client/services";
// Note: getReference is the SDK's internal mapper — runs automatically within mapPage
// At component level, refs arrive already resolved
```

---

## 12. AppContext — Global State

### Web (Next.js) Setup

```tsx
// app/[[...pathname]]/client.tsx
"use client";
import { AppContextProvider } from "@appaflytech/wappa-client/core/contexts";
import render from "@/core/render";

export function ClientPage({
  page,
  settings,
  themes,
  language,
  languages,
  environment,
}) {
  const contextValue = {
    page,
    settings,
    theme: themes.find((t) => t.id === page.theme),
    themes,
    language,
    languages,
    environment,
    container: "browser",
    cookies: {},
    headers: {},
    params: {},
    customData: null,
    setData: () => {},
    setCustomData: () => {},
  };

  return (
    <AppContextProvider value={contextValue}>
      {render(page.layout, page.views)}
    </AppContextProvider>
  );
}
```

### Usage Inside a Component

```tsx
import { useApp } from "@appaflytech/wappa-client/core/contexts";

function MyComponent() {
  const { page, language, theme, settings, user } = useApp();

  return <div>{page.title}</div>;
}
```

---

## 13. Full Web Project Setup (Next.js)

### File Structure

```
my-wappa-web/
├── app/
│   └── [[...pathname]]/
│       ├── page.tsx        # SSR entry
│       ├── context.tsx     # Data fetching
│       └── client.tsx      # Client boundary
├── components/
│   ├── index.tsx           # Component registry
│   └── wap/
│       ├── gridview/
│       │   ├── container/
│       │   ├── row/
│       │   └── column/
│       ├── heading/
│       ├── paragraph/
│       ├── html/
│       ├── image/
│       ├── video/
│       ├── button/
│       ├── section/
│       ├── anchor/
│       └── iframe/
├── core/
│   └── render.tsx
└── services/
    └── environment.ts
```

### SSR Entry — `app/[[...pathname]]/page.tsx`

```tsx
import { Metadata } from "next";
import { getContext } from "./context";
import { ClientPage } from "./client";
import { notFound } from "next/navigation";

export async function generateMetadata({
  params,
  searchParams,
}): Promise<Metadata> {
  const context = await getContext({ params, searchParams });
  if (!context?.page) return {};

  return {
    title: context.page.metatags?.title || context.page.title,
    description: context.page.metatags?.description,
  };
}

export default async function Page({ params, searchParams }) {
  const context = await getContext({ params, searchParams });

  if (!context?.page) notFound();

  return <ClientPage {...context} />;
}
```

### Context — `app/[[...pathname]]/context.tsx`

```tsx
import { headers, cookies } from "next/headers";
import { Environment } from "@appaflytech/wappa-client/core/classes";
import {
  siteService,
  configService,
  pageService,
} from "@appaflytech/wappa-client/services";

export async function getContext({ params, searchParams }) {
  const headersList = await headers();
  const host = headersList.get("host") || "";
  const pathSegments = (params?.pathname as string[]) || [];
  const language = pathSegments[0]?.match(/^[a-z]{2}-[a-z]{2}$/)
    ? pathSegments[0]
    : "tr-tr";
  const path = "/" + pathSegments.join("/");

  const env = new Environment();
  env.update({
    cdn: process.env.NEXT_PUBLIC_CDN || "",
    key: process.env.NEXT_PUBLIC_SITE_KEY || "",
    service: `${process.env.NEXT_PUBLIC_API}/${process.env.NEXT_PUBLIC_SITE_KEY}`,
    url: `https://${host}`,
    environment: process.env.NEXT_PUBLIC_ENV || "development",
  });

  try {
    const [config, page] = await Promise.all([
      configService.get(env.context, language),
      pageService.get(env.context, { path, isMobile: false }),
    ]);

    return {
      page,
      settings: config.settings,
      themes: config.themes,
      language,
      languages: config.languages,
      environment: env.context,
    };
  } catch (error) {
    return { error };
  }
}
```

---

## 14. Tam Mobile Projesi Kurulumu (Expo)

### File Structure

```
my-wappa-mobile/
├── app/
│   ├── index.tsx           # Main route
│   └── [siteKey]/
│       └── [...pathname].tsx
├── components/
│   ├── index.tsx           # Component registry
│   ├── WapScreen.tsx       # Page render component
│   ├── AppConfigProvider.tsx
│   └── wap/
│       ├── container/
│       ├── row/
│       ├── column/
│       ├── heading/
│       ├── paragraph/
│       ├── html/
│       ├── image/
│       ├── video/
│       ├── button/
│       ├── section/
│       └── anchor/
├── core/
│   └── render.tsx
├── services/
│   └── contextService.ts
└── store/
    └── store.ts
```

### AppConfigProvider — `components/AppConfigProvider.tsx`

```tsx
import React, { useEffect } from "react";
import { Environment } from "@appaflytech/wappa-client/core/classes";
import { siteService, configService } from "@appaflytech/wappa-client/services";
import { NodeEnv } from "@appaflytech/wappa-client/constants/enums";

interface AppConfigProviderProps {
  children: React.ReactNode;
  siteKey: string;
}

export function AppConfigProvider({
  children,
  siteKey,
}: AppConfigProviderProps) {
  useEffect(() => {
    loadConfig();
  }, [siteKey]);

  const loadConfig = async () => {
    const env = new Environment();

    if (process.env.EXPO_PUBLIC_ENV === NodeEnv.Production) {
      const site = await siteService.get(env.context, siteKey);
      env.update({
        cdn: process.env.EXPO_PUBLIC_WAP_CDN || "",
        key: site.key,
        service: `${site.service}/${site.key}`,
        url: site.url,
        environment: "production",
      });
    } else {
      env.update({
        cdn: process.env.EXPO_PUBLIC_WAP_CDN || "",
        key: siteKey,
        service: `${process.env.EXPO_PUBLIC_WAP_API}/${siteKey}`,
        url: "",
        environment: "development",
      });
    }

    const config = await configService.get(env.context, "tr-tr");
    // write to store
  };

  return <>{children}</>;
}
```

### WapScreen — Basic Page Component

```tsx
import React, { useEffect, useState } from "react";
import { View, ScrollView, ActivityIndicator } from "react-native";
import { contextService } from "../services/contextService";
import { render } from "../core/render";
import { AppContextProps } from "@appaflytech/wappa-client/core/contexts";

interface WapScreenProps {
  siteKey?: string;
  pathname?: string;
  language?: string;
  previewId?: string;
}

export function WapScreen({
  siteKey,
  pathname,
  language,
  previewId,
}: WapScreenProps) {
  const [state, setState] = useState<{
    data: Partial<AppContextProps>;
    loading: boolean;
  }>({
    data: {},
    loading: true,
  });

  useEffect(() => {
    contextService
      .getContext({ siteKey, pathname, language, previewId })
      .then((context) => setState({ data: context as any, loading: false }));
  }, [siteKey, pathname, language, previewId]);

  if (state.loading) {
    return (
      <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
        <ActivityIndicator />
      </View>
    );
  }

  const { page } = state.data;
  if (!page) return null;

  return <ScrollView>{render(page.layout, page.views)}</ScrollView>;
}
```

---

## 15. Defining Schemas in Admin UI (wappa-admin-ui)

To add a new component to the admin page builder, register it in the `elements.ts` file:

```ts
// wappa-admin-ui/src/components/page-builder/definitions/elements.ts
import { ComponentNode } from "services/http/component";
import { AccessType, Status } from "services/http";

const elements: ComponentNode[] = [
  // ... existing components ...
  {
    name: "my-card", // registry key (must match components/index.tsx)
    id: "my-card",
    title: "Card",
    description: "Image content card",
    icon: "card",
    droppable: false, // true = can drop components inside
    isMobile: true, // Mobile support
    type: "content",
    status: Status.Active,
    access: { type: AccessType.Global },
    definition: {
      groups: [],
      props: [
        { name: "title", title: "Title", type: "text" },
        { name: "description", title: "Description", type: "long-text" },
        { name: "image", title: "Image", type: "image" },
        { name: "href", title: "Link", type: "link" },
        { name: "className", title: "CSS Class", type: "text" },
      ],
    },
  },
];
```

### ComponentProperty Types

| type         | Description                               |
| ------------ | ----------------------------------------- |
| `text`       | Single-line text                          |
| `long-text`  | Multi-line text                           |
| `number`     | Number                                    |
| `dropdown`   | Option list (`options: [{label, value}]`) |
| `switch`     | Boolean toggle                            |
| `color`      | Color picker                              |
| `image`      | Image reference                           |
| `file`       | File reference                            |
| `link`       | URL reference                             |
| `video`      | Video reference                           |
| `date`       | Date                                      |
| `datetime`   | Date + time                               |
| `time`       | Time                                      |
| `price`      | Price                                     |
| `array`      | Array (repeating object)                  |
| `object`     | Nested object (sub `props`)               |
| `reference`  | Query/widget reference                    |
| `schema`     | Nested schema                             |
| `navigation` | Navigation reference                      |
| `editor`     | Rich text editor                          |
| `picture`    | Image collection                          |

---

## 16. Mapping System

The admin's "mapping" feature feeds a prop from a dynamic source instead of a static value:

| Reference Type | Source                           |
| -------------- | -------------------------------- |
| Static (`''`)  | Fixed value entered in admin     |
| Widget         | Current widget fields            |
| Page           | Page metadata fields             |
| Meta           | DynamicEntity meta fields        |
| Query          | Selected query result            |
| Raw Query      | Raw query result                 |
| Showcase       | Showcase data                    |
| Repeater       | Parent `array-repeater` row data |

### Page Fields (pageFields)

```ts
// Available in mapping sources
{ label: 'ID', value: 'id', type: FieldType.Text }
{ label: 'Title', value: 'title', type: FieldType.Text }
{ label: 'URL', value: 'path', type: FieldType.Link }
{ label: 'Image', value: 'image', type: FieldType.Image }
```

---

## 17. Theme System

### Theme Structure

```ts
type Theme = {
  id: string;
  name: string;
  mobileNeutrals: Record<string, string>; // Mobile color values
  mobileFontSizes: Record<string, number>; // Mobile font sizes
  // For web, CSS variables are automatically injected
};
```

### Mobile Theme Usage

```tsx
import { useTheme } from "../components/ThemeProvider";

function MyComponent() {
  const { getThemeColor, themeData } = useTheme();

  return (
    <View style={{ backgroundColor: getThemeColor("primary") }}>
      <Text style={{ color: getThemeColor("text") }}>Hello</Text>
    </View>
  );
}
```

### Web Theme Usage

Themes configured in admin are injected into the HTML root as CSS variables:

```css
:root {
  --color-primary: #3b82f6;
  --color-text: #1f2937;
  /* ... */
}
```

---

## 18. Quick Start Checklist

### Web Project

- [ ] `npm install @appaflytech/wappa-client`
- [ ] Add `NEXT_PUBLIC_API`, `NEXT_PUBLIC_CDN`, `NEXT_PUBLIC_SITE_KEY` to `.env`
- [ ] Create `core/render.tsx`
- [ ] Create `components/index.tsx` registry
- [ ] Implement core components (`container`, `row`, `column`, `heading`, `paragraph`, `html`, `image`, `video`, `button`, `section`, `link`, `iframe`)
- [ ] Create `page.tsx`, `context.tsx`, `client.tsx` under `app/[[...pathname]]/`
- [ ] Add `ArrayRepeater` to registry

### Mobile Project

- [ ] `npm install @appaflytech/wappa-client`
- [ ] Add `EXPO_PUBLIC_WAP_API`, `EXPO_PUBLIC_WAP_CDN` to `.env`
- [ ] Create `core/render.tsx` (React Native version)
- [ ] Create `components/index.tsx` registry (React Native components)
- [ ] NativeWind setup (TailwindCSS → React Native)
- [ ] Implement core components with React Native
- [ ] Create `services/contextService.ts`
- [ ] Create `components/WapScreen.tsx`
- [ ] Create `app/[siteKey]/[...pathname].tsx` route

---

## 19. Common Scenarios

### Unknown Component

Fallback for components not defined in registry:

```tsx
// Web
default: return null  // Component is not rendered

// Mobile
default:
  console.warn(`Unknown component: ${name}`)
  return <View style={{ padding: 10, backgroundColor: '#ffeeee' }}>{...}</View>
```

### Empty Page / Loading State

```tsx
if (state.loading) return <LoadingSpinner />;
if (!page) return <NotFound />;
if (error) throw error; // Caught by Next.js error.tsx or RN error boundary
```

### Array Repeater on Mobile

When `isMappingRender = true` in mobile render:

```ts
const mappingProps = isMappingRender ? props?.mappedValue || {} : {};
const combinedProps = { ...refs, ...mappingProps, ...props };
```

`mappedValue` carries the row data from the repeater.

---

## 20. Package.json Dependencies

### Web

```json
{
  "dependencies": {
    "@appaflytech/wappa-client": "^0.0.5",
    "next": "^14.x",
    "react": "^18.x",
    "react-dom": "^18.x"
  }
}
```

### Mobile

```json
{
  "dependencies": {
    "@appaflytech/wappa-client": "^0.0.5",
    "expo": "~54.x",
    "react": "19.1.0",
    "react-native": "0.81.4",
    "expo-router": "~6.0.6",
    "nativewind": "^4.1.23",
    "tailwindcss": "^3.x",
    "zustand": "^4.5.1",
    "@react-native-async-storage/async-storage": "^1.x"
  }
}
```

---

## 21. Full Mobile Component Implementations

The code in this section consists of real implementations taken from the **wappa-mobile** project.

### `utils/path.ts`

```ts
export const getCDNImage = (image: string): string => {
  return `${process.env.EXPO_PUBLIC_WAP_CDN}/${image}`;
};

/**
 * The WAP render system may sometimes send unresolved object values like ref/type/label.
 * This helper protects Text renders that expect strings from such cases.
 */
export const safeText = (value: any): string => {
  if (value === null || value === undefined) return "";
  if (typeof value === "string") return value;
  if (typeof value === "number") return String(value);
  return ""; // empty string for unexpected types like objects or arrays
};
```

### `store/store.ts` — Zustand Global State

```ts
import { create } from "zustand";
import { PageComponent } from "@appaflytech/wappa-client/constants/types";

export interface AppContextState {
  environment?: any;
  container: "mobile" | "browser";
  settings?: any;
  themes?: any[];
  theme?: any;
  languages?: any[];
  language?: string;
  page?: {
    id: string | number;
    title: string;
    path: string;
    layout: PageComponent[];
    views: Record<string, PageComponent[]>;
    image?: any;
    metatags?: any;
    theme?: string | number;
  };
  headers?: Record<string, string>;
  cookies?: Record<string, string>;
  params?: Record<string, any>;
  error?: any;
  isLoading: boolean;
  isInitialized: boolean;
}

interface AppActions {
  setEnvironment: (environment: any) => void;
  setSettings: (settings: any) => void;
  setThemes: (themes: any[]) => void;
  setTheme: (theme: any) => void;
  setLanguages: (languages: any[]) => void;
  setLanguage: (language: string) => void;
  setPage: (page: AppContextState["page"]) => void;
  setError: (error: any) => void;
  setLoading: (loading: boolean) => void;
  setInitialized: (initialized: boolean) => void;
  initialize: (context: Partial<AppContextState>) => void;
  reset: () => void;
}

type AppStore = AppContextState & AppActions;

const initialState: AppContextState = {
  container: "mobile",
  isLoading: false,
  isInitialized: false,
};

export const useAppStore = create<AppStore>((set) => ({
  ...initialState,

  setEnvironment: (environment) => set({ environment }),
  setSettings: (settings) => set({ settings }),
  setThemes: (themes) => set({ themes }),
  setTheme: (theme) => set({ theme }),
  setLanguages: (languages) => set({ languages }),
  setLanguage: (language) => set({ language }),
  setPage: (page) => set({ page }),
  setError: (error) => set({ error }),
  setLoading: (isLoading) => set({ isLoading }),
  setInitialized: (isInitialized) => set({ isInitialized }),

  initialize: (context) =>
    set({
      ...context,
      isInitialized: true,
      isLoading: false,
    }),

  reset: () => set(initialState),
}));
```

### `components/ThemeProvider.tsx`

```tsx
import React, {
  createContext,
  useContext,
  ReactNode,
  useState,
  useCallback,
} from "react";

interface DynamicThemeData {
  colors?: Record<string, Record<string, string>>;
  fontSizes?: Record<string, number>;
  [key: string]: any;
}

interface ThemeContextType {
  themeData: DynamicThemeData | null;
  setThemeData: (data: DynamicThemeData) => void;
  isDarkMode: boolean;
  toggleMode: () => void;
  setMode: (mode: "dark" | "light") => void;
  getColor: (shade: string) => string;
  fontSizes: Record<string, number>;
  setFontSizes: (sizes: Record<string, number>) => void;
  getFontSize: (size: string) => number;
}

const ThemeContext = createContext<ThemeContextType | null>(null);

const DEFAULT_FONT_SIZES: Record<string, number> = {
  xs: 12,
  sm: 14,
  base: 16,
  lg: 18,
  xl: 20,
  "2xl": 24,
  "3xl": 30,
  "4xl": 36,
  "5xl": 48,
  "6xl": 60,
};

export function ThemeProvider({
  children,
  initialThemeData,
}: {
  children: ReactNode;
  initialThemeData?: DynamicThemeData;
}) {
  const [themeData, setThemeDataState] = useState<DynamicThemeData | null>(
    initialThemeData || null,
  );
  const [isDarkMode, setIsDarkMode] = useState(false);
  const [fontSizes, setFontSizesState] = useState(DEFAULT_FONT_SIZES);

  const setThemeData = useCallback((data: DynamicThemeData) => {
    setThemeDataState((prev) => {
      if (prev && JSON.stringify(prev) === JSON.stringify(data)) return prev;
      return data;
    });
  }, []);

  const toggleMode = useCallback(() => setIsDarkMode((v) => !v), []);
  const setMode = useCallback(
    (mode: "dark" | "light") => setIsDarkMode(mode === "dark"),
    [],
  );

  const getColor = useCallback(
    (shade: string): string => {
      const mode = isDarkMode ? "dark" : "light";
      return (
        themeData?.colors?.[mode]?.[shade] || (isDarkMode ? "#fff" : "#000")
      );
    },
    [themeData, isDarkMode],
  );

  const setFontSizes = useCallback((sizes: Record<string, number>) => {
    setFontSizesState(sizes);
  }, []);

  const getFontSize = useCallback(
    (size: string): number => fontSizes[size] ?? fontSizes.base ?? 16,
    [fontSizes],
  );

  return (
    <ThemeContext.Provider
      value={{
        themeData,
        setThemeData,
        isDarkMode,
        toggleMode,
        setMode,
        getColor,
        fontSizes,
        setFontSizes,
        getFontSize,
      }}
    >
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme(): ThemeContextType {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error("useTheme must be used within ThemeProvider");
  return ctx;
}
```

### `components/wap/container/Container.tsx`

```tsx
import React from "react";
import { View } from "react-native";

interface ContainerProps {
  children?: React.ReactNode;
  size?: "fluid" | "extended" | "wide" | "medium" | "narrow";
  className?: string;
  style?: any;
  [key: string]: any;
}

const Container: React.FC<ContainerProps> = ({
  children,
  size = "fluid",
  className = "flex-1",
  style,
  ...props
}) => {
  const getSizeClasses = () => {
    switch (size) {
      case "fluid":
        return "w-full px-4";
      case "extended":
        return "w-full max-w-7xl mx-auto px-4";
      case "wide":
        return "w-full max-w-5xl mx-auto px-4";
      case "medium":
        return "w-full max-w-3xl mx-auto px-4";
      case "narrow":
        return "w-full max-w-xl mx-auto px-4";
      default:
        return "w-full px-4";
    }
  };

  const containerClasses = `${getSizeClasses()} ${className}`.trim();
  const bg = style?.background?.backgroundColor || "transparent";
  const dynamicStyle = {
    backgroundColor: bg,
    ...style?.margin,
    ...style?.padding,
    ...style?.layout,
    ...style?.border,
    ...style?.background,
    ...(style || {}),
  };

  return (
    <View className={containerClasses} style={dynamicStyle} {...props}>
      {children}
    </View>
  );
};

export default Container;
```

### `components/wap/row/Row.tsx`

```tsx
import React from "react";
import { View } from "react-native";

interface ScreenConfig {
  direction?: "row" | "row-reverse" | "column" | "column-reverse";
  align?: "start" | "center" | "end" | "baseline" | "stretch";
  justify?: "start" | "center" | "end" | "around" | "between" | "evenly";
  wrap?: "nowrap" | "wrap" | "wrap-reverse";
  gutter?: "none" | "xs" | "sm" | "md" | "lg" | "xl";
}

interface RowProps {
  children?: React.ReactNode;
  xs?: ScreenConfig;
  className?: string;
  style?: any;
  [key: string]: any;
}

const Row: React.FC<RowProps> = ({
  children,
  xs,
  className = "",
  style,
  ...props
}) => {
  const direction = xs?.direction || "row";
  const align = xs?.align || "start";
  const justify = xs?.justify || "start";

  const dirMap: Record<string, string> = {
    row: "flex-row",
    "row-reverse": "flex-row-reverse",
    column: "flex-col",
    "column-reverse": "flex-col-reverse",
  };
  const alignMap: Record<string, string> = {
    start: "items-start",
    center: "items-center",
    end: "items-end",
    baseline: "items-baseline",
    stretch: "items-stretch",
  };
  const justifyMap: Record<string, string> = {
    start: "justify-start",
    center: "justify-center",
    end: "justify-end",
    around: "justify-around",
    between: "justify-between",
    evenly: "justify-evenly",
  };

  const classes = [
    dirMap[direction] || "flex-row",
    alignMap[align] || "items-start",
    justifyMap[justify] || "justify-start",
    className,
  ]
    .join(" ")
    .trim();

  const bg = style?.background?.backgroundColor || "transparent";
  const dynamicStyle = {
    backgroundColor: bg,
    ...style?.margin,
    ...style?.padding,
    ...style?.layout,
    ...style?.border,
    ...style?.background,
    ...(style || {}),
  };

  return (
    <View className={classes} style={dynamicStyle} {...props}>
      {children}
    </View>
  );
};

export default Row;
```

### `components/wap/column/Column.tsx`

```tsx
import React from "react";
import { View } from "react-native";

interface ColumnScreenConfig {
  size?: number; // 1-12
  align?: "start" | "center" | "end";
  offset?: number;
}

interface ColumnProps {
  children?: React.ReactNode;
  xs?: ColumnScreenConfig;
  className?: string;
  style?: any;
  [key: string]: any;
}

const Column: React.FC<ColumnProps> = ({
  children,
  xs,
  className = "",
  style,
  ...props
}) => {
  const size = xs?.size;
  const sizeClass = size ? `w-[${((size / 12) * 100).toFixed(2)}%]` : "flex-1";
  const alignClass =
    xs?.align === "center"
      ? "items-center"
      : xs?.align === "end"
        ? "items-end"
        : "";

  const classes = [sizeClass, alignClass, className].filter(Boolean).join(" ");
  const bg = style?.background?.backgroundColor || "transparent";
  const dynamicStyle = {
    backgroundColor: bg,
    ...style?.margin,
    ...style?.padding,
    ...style?.layout,
    ...style?.border,
    ...style?.background,
    ...(style || {}),
  };

  return (
    <View className={classes} style={dynamicStyle} {...props}>
      {children}
    </View>
  );
};

export default Column;
```

### `components/wap/heading/Heading.tsx`

```tsx
import React from "react";
import { Text } from "react-native";
import { useTheme } from "../../ThemeProvider";
import { safeText } from "../../../utils/path";

interface HeadingProps {
  content?: any;
  children?: any;
  nodeType?: "h1" | "h2" | "h3" | "h4" | "h5" | "h6";
  className?: string;
  style?: any;
  color?: string;
  customColor?: string;
  textAlign?: "left" | "center" | "right";
  [key: string]: any;
}

const SIZE_MAP: Record<string, number> = {
  h1: 32,
  h2: 28,
  h3: 24,
  h4: 20,
  h5: 18,
  h6: 16,
};

const WEIGHT_MAP: Record<string, string> = {
  h1: "800",
  h2: "700",
  h3: "600",
  h4: "600",
  h5: "500",
  h6: "500",
};

const Heading: React.FC<HeadingProps> = ({
  content,
  children,
  nodeType = "h2",
  className = "",
  style,
  color,
  customColor,
  textAlign = "left",
}) => {
  const { getColor, isDarkMode } = useTheme();
  const text = safeText(content ?? children);

  if (!text) return null;

  const fontSize = SIZE_MAP[nodeType] || 24;
  const fontWeight = WEIGHT_MAP[nodeType] as any;
  const textColor =
    customColor || color || getColor("900") || (isDarkMode ? "#fff" : "#111");

  return (
    <Text
      className={className}
      style={[
        {
          fontSize,
          fontWeight,
          textAlign,
          color: textColor,
          lineHeight: fontSize * 1.2,
        },
        style?.margin,
        style?.padding,
        style?.layout,
        style?.border,
        style?.background,
        style,
      ].filter(Boolean)}
      accessibilityRole="header"
    >
      {text}
    </Text>
  );
};

export default Heading;
```

### `components/wap/paragraph/Paragraph.tsx`

```tsx
import React from "react";
import { Text } from "react-native";
import { useTheme } from "../../ThemeProvider";
import { safeText } from "../../../utils/path";

interface ParagraphProps {
  content?: any;
  children?: any;
  className?: string;
  style?: any;
  fontSize?: "xs" | "sm" | "base" | "lg" | "xl" | "2xl" | "3xl";
  fontWeight?: "light" | "normal" | "medium" | "semibold" | "bold";
  textAlign?: "left" | "center" | "right" | "justify";
  color?: string;
  customColor?: string;
  lineHeight?: "tight" | "normal" | "relaxed" | "loose";
  numberOfLines?: number;
  [key: string]: any;
}

const WEIGHT_MAP: Record<string, string> = {
  light: "300",
  normal: "400",
  medium: "500",
  semibold: "600",
  bold: "700",
};

const LINE_HEIGHT_MULT: Record<string, number> = {
  tight: 1.2,
  normal: 1.4,
  relaxed: 1.6,
  loose: 1.8,
};

const Paragraph: React.FC<ParagraphProps> = ({
  content,
  children,
  className = "",
  style,
  fontSize = "base",
  fontWeight = "normal",
  textAlign = "left",
  color,
  customColor,
  lineHeight = "relaxed",
  numberOfLines,
}) => {
  const { getColor, isDarkMode, getFontSize } = useTheme();
  const text = safeText(content ?? children);

  if (!text) return null;

  const fs = getFontSize(fontSize);
  const textColor =
    customColor ||
    color ||
    getColor("700") ||
    (isDarkMode ? "#e5e5e5" : "#333");

  return (
    <Text
      className={className}
      numberOfLines={numberOfLines}
      style={[
        {
          fontSize: fs,
          fontWeight: WEIGHT_MAP[fontWeight] as any,
          textAlign: textAlign as any,
          color: textColor,
          lineHeight: LINE_HEIGHT_MULT[lineHeight] * fs,
        },
        style?.margin,
        style?.padding,
        style?.layout,
        style?.border,
        style?.background,
        style,
      ].filter(Boolean)}
    >
      {text}
    </Text>
  );
};

export default Paragraph;
```

### `components/wap/image/Image.tsx`

```tsx
import React, { useState } from "react";
import { Image as RNImage, ActivityIndicator, View, Text } from "react-native";
import { getCDNImage } from "../../../utils/path";

interface ImageProps {
  source?: { src?: string; value?: { src?: string } };
  alt?: string;
  width?: number;
  height?: number;
  className?: string;
  style?: any;
  [key: string]: any;
}

const Image: React.FC<ImageProps> = ({
  source,
  width,
  height,
  alt,
  className = "",
  style,
  ...props
}) => {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(false);

  const imageClasses = `w-full h-auto ${className}`.trim();
  const noImage = !source?.src && !source?.value?.src;

  if (noImage || error) {
    return (
      <View
        className={`${imageClasses} items-center justify-center bg-gray-200`}
        style={[{ height: 200 }, style]}
      >
        <Text className="text-sm text-gray-500">
          {error ? "Image Error" : "No Image"}
        </Text>
      </View>
    );
  }

  const imageUri = getCDNImage(source?.value?.src || source?.src || "");

  return (
    <View className="relative">
      <RNImage
        source={{ uri: imageUri }}
        className={imageClasses}
        style={[width ? { width } : {}, height ? { height } : {}, style]}
        onLoadEnd={() => setLoading(false)}
        onError={() => {
          setLoading(false);
          setError(true);
        }}
        accessible={!!alt}
        accessibilityLabel={alt}
        resizeMode="cover"
        {...props}
      />
      {loading && (
        <View className="absolute inset-0 items-center justify-center bg-black/10">
          <ActivityIndicator color="#666" />
        </View>
      )}
    </View>
  );
};

export default Image;
```

### `components/wap/anchor/Anchor.tsx`

```tsx
import React from "react";
import { TouchableOpacity, Text, Linking } from "react-native";

interface AnchorProps {
  children?: React.ReactNode;
  source?: { href?: string; target?: "_blank" | "_self"; title?: string };
  className?: string;
  style?: any;
  [key: string]: any;
}

const Anchor: React.FC<AnchorProps> = ({
  children,
  source,
  className = "",
  style,
  ...props
}) => {
  const handlePress = async () => {
    if (!source?.href) return;
    try {
      const ok = await Linking.canOpenURL(source.href);
      if (ok) await Linking.openURL(source.href);
    } catch (e) {
      console.error("Anchor open error:", e);
    }
  };

  if (!source?.href) {
    return (
      <Text className={`text-gray-400 ${className}`} style={style}>
        {children || "Link without URL"}
      </Text>
    );
  }

  return (
    <TouchableOpacity onPress={handlePress} activeOpacity={0.7} {...props}>
      <Text
        className={`text-blue-600 underline ${className}`}
        style={style}
        accessibilityRole="link"
        accessibilityHint={source.title || `Open ${source.href}`}
      >
        {children || source.href}
      </Text>
    </TouchableOpacity>
  );
};

export default Anchor;
```

### `components/wap/section/Section.tsx`

```tsx
import React from "react";
import { View, Text, TouchableOpacity, Linking } from "react-native";
import Container from "../container/Container";

interface SectionProps {
  children?: React.ReactNode;
  title?: string;
  description?: string;
  anchor?: { href?: string; target?: "_blank" | "_self"; title?: string };
  container?: {
    size?: "fluid" | "extended" | "wide" | "medium" | "narrow";
    className?: string;
  };
  className?: string;
  style?: any;
  [key: string]: any;
}

const Section: React.FC<SectionProps> = ({
  children,
  title,
  description,
  anchor,
  container,
  className = "",
  style,
  ...props
}) => {
  const handleAnchorPress = async () => {
    if (!anchor?.href) return;
    try {
      const ok = await Linking.canOpenURL(anchor.href);
      if (ok) await Linking.openURL(anchor.href);
    } catch (e) {
      console.error("Section anchor error:", e);
    }
  };

  return (
    <View className={`w-full py-8 ${className}`} style={style} {...props}>
      <Container
        size={container?.size || "fluid"}
        className={container?.className}
      >
        {(title || description || anchor) && (
          <View className="mb-6">
            {title && (
              <Text className="mb-3 text-2xl font-bold text-gray-900">
                {title}
              </Text>
            )}
            {description && (
              <Text className="mb-4 text-base leading-6 text-gray-600">
                {description}
              </Text>
            )}
            {anchor?.href && (
              <TouchableOpacity onPress={handleAnchorPress} activeOpacity={0.7}>
                <Text className="font-medium text-blue-600 underline">
                  {anchor.title || "Learn More →"}
                </Text>
              </TouchableOpacity>
            )}
          </View>
        )}
        {children}
      </Container>
    </View>
  );
};

export default Section;
```

### `components/wap/button/Button.tsx`

```tsx
import React from "react";
import {
  TouchableOpacity,
  Text,
  Linking,
  ActivityIndicator,
} from "react-native";

interface ButtonAnchor {
  href?: string;
  target?: "_blank" | "_self";
  title?: string;
}

interface ButtonProps {
  children?: React.ReactNode;
  anchor?: ButtonAnchor;
  appearance?: "filled" | "outlined" | "ghost";
  kind?: "primary" | "secondary" | "danger" | "success";
  shape?: "rounded" | "square" | "pill";
  disabled?: boolean;
  loading?: boolean;
  onPress?: () => void;
  className?: string;
  style?: any;
  [key: string]: any;
}

const KIND_CLASSES: Record<
  string,
  { bg: string; text: string; border: string }
> = {
  primary: { bg: "bg-blue-600", text: "text-white", border: "border-blue-600" },
  secondary: {
    bg: "bg-gray-600",
    text: "text-white",
    border: "border-gray-600",
  },
  danger: { bg: "bg-red-600", text: "text-white", border: "border-red-600" },
  success: {
    bg: "bg-green-600",
    text: "text-white",
    border: "border-green-600",
  },
};

const SHAPE_CLASSES: Record<string, string> = {
  rounded: "rounded-lg",
  square: "rounded-none",
  pill: "rounded-full",
};

const Button: React.FC<ButtonProps> = ({
  children,
  anchor,
  appearance = "filled",
  kind = "primary",
  shape = "rounded",
  disabled,
  loading,
  onPress,
  className = "",
  style,
}) => {
  const handlePress = async () => {
    if (disabled || loading) return;
    if (onPress) {
      onPress();
      return;
    }
    if (!anchor?.href) return;
    try {
      const ok = await Linking.canOpenURL(anchor.href);
      if (ok) await Linking.openURL(anchor.href);
    } catch (e) {
      console.error("Button link error:", e);
    }
  };

  const colors = KIND_CLASSES[kind] || KIND_CLASSES.primary;
  const shapeClass = SHAPE_CLASSES[shape] || "rounded-lg";

  const bgClass =
    appearance === "filled"
      ? colors.bg
      : appearance === "outlined"
        ? "bg-transparent border-2 " + colors.border
        : "bg-transparent";

  const textClass =
    appearance === "filled"
      ? colors.text
      : appearance === "outlined" || appearance === "ghost"
        ? colors.bg.replace("bg-", "text-")
        : "text-gray-700";

  return (
    <TouchableOpacity
      onPress={handlePress}
      disabled={disabled || loading}
      activeOpacity={0.8}
      className={`px-6 py-3 items-center justify-center ${bgClass} ${shapeClass} ${disabled ? "opacity-50" : ""} ${className}`}
      style={style}
      accessibilityRole="button"
      accessibilityLabel={
        anchor?.title || (typeof children === "string" ? children : undefined)
      }
    >
      {loading ? (
        <ActivityIndicator color={appearance === "filled" ? "#fff" : "#666"} />
      ) : (
        <Text className={`text-base font-semibold ${textClass}`}>
          {children || anchor?.title || "Button"}
        </Text>
      )}
    </TouchableOpacity>
  );
};

export default Button;
```

---

## 22. WapScreen — Page Loading Component (Mobile)

`WapScreen` loads and renders a Wappa CMS page by path.

> **NOTE**: Do NOT add `WapScreen` to `components/index.tsx` due to circular dependency. Import it directly only.

```tsx
// components/WapScreen.tsx
import React, { useEffect } from "react";
import { ScrollView, View, ActivityIndicator, Text } from "react-native";
import { useAppStore } from "../store/store";
import { render } from "../core/render";
import { ThemeProvider } from "./ThemeProvider";

interface WapScreenProps {
  path: string;
  scrollable?: boolean;
  renderHeader?: () => React.ReactNode;
  renderFooter?: () => React.ReactNode;
  onPageLoad?: (page: any) => void;
}

export default function WapScreen({
  path,
  scrollable = true,
  renderHeader,
  renderFooter,
  onPageLoad,
}: WapScreenProps) {
  const { page, isLoading, error, theme } = useAppStore();

  // Page loading is handled via contextService (see §23)

  useEffect(() => {
    if (page && onPageLoad) onPageLoad(page);
  }, [page]);

  if (isLoading) {
    return (
      <View className="flex-1 items-center justify-center">
        <ActivityIndicator size="large" color="#3b82f6" />
      </View>
    );
  }

  if (error) {
    return (
      <View className="flex-1 items-center justify-center p-6">
        <Text className="text-lg font-bold text-red-600 mb-2">Page Error</Text>
        <Text className="text-gray-600 text-center">
          {error?.message || "Something went wrong"}
        </Text>
      </View>
    );
  }

  if (!page) {
    return (
      <View className="flex-1 items-center justify-center">
        <Text className="text-gray-400">No page data</Text>
      </View>
    );
  }

  const content = render(page.layout, page.views);

  return (
    <ThemeProvider initialThemeData={theme}>
      {scrollable ? (
        <ScrollView className="flex-1">
          {renderHeader?.()}
          {content}
          {renderFooter?.()}
        </ScrollView>
      ) : (
        <View className="flex-1">
          {renderHeader?.()}
          {content}
          {renderFooter?.()}
        </View>
      )}
    </ThemeProvider>
  );
}
```

---

## 23. contextService — API Data Fetching (Mobile)

```ts
// services/contextService.ts
import { Environment } from "@appaflytech/wappa-client/core/classes";
import {
  pageService,
  configService,
  siteService,
} from "@appaflytech/wappa-client/services";
import { useAppStore } from "../store/store";

const SITE_KEY = process.env.EXPO_PUBLIC_WAP_SITE_KEY || "";
const API_URL = process.env.EXPO_PUBLIC_WAP_API || "";
const CDN_URL = process.env.EXPO_PUBLIC_WAP_CDN || "";
const ENV =
  (process.env.EXPO_PUBLIC_ENV as "development" | "production") ||
  "development";

export const environment = new Environment();
environment.update({
  cdn: CDN_URL,
  key: SITE_KEY,
  service: `${API_URL}/${SITE_KEY}`,
  url: "",
  environment: ENV,
});

/**
 * Load site config (theme, language, settings)
 * Usually called once at app startup.
 */
export async function loadSiteConfig(language = "tr") {
  const store = useAppStore.getState();
  try {
    store.setLoading(true);
    const config = await configService.get(environment.context, language);
    store.setSettings(config.settings);
    store.setThemes(config.themes);
    store.setLanguages(config.languages);
    if (config.themes?.length) store.setTheme(config.themes[0]);
  } catch (err: any) {
    store.setError(err);
  } finally {
    store.setLoading(false);
  }
}

/**
 * Load page at the specified path
 */
export async function loadPage(path: string) {
  const store = useAppStore.getState();
  try {
    store.setLoading(true);
    store.setError(undefined);
    const page = await pageService.get(environment.context, {
      path,
      isMobile: true,
    });
    if (!page) {
      store.setError(new Error("Page not found"));
      return;
    }
    store.setPage({
      id: page.id,
      title: page.title,
      path: page.path,
      layout: page.layout,
      views: page.views,
      image: page.image,
      metatags: page.metatags,
      theme: page.theme,
    });
  } catch (err: any) {
    store.setError(err);
  } finally {
    store.setLoading(false);
  }
}
```

---

## 24. Next.js Full Page Implementation (Web)

### `app/[[...pathname]]/page.tsx` — Server Entry

```tsx
// app/[[...pathname]]/page.tsx
import { notFound } from "next/navigation";
import { Suspense } from "react";
import { getPageData } from "./context";
import WapClient from "./client";

interface PageProps {
  params: { pathname?: string[] };
  searchParams: Record<string, string>;
}

export default async function DynamicPage({ params, searchParams }: PageProps) {
  const pathname = params.pathname ? `/${params.pathname.join("/")}` : "/";
  const data = await getPageData(pathname);

  if (!data) notFound();

  return (
    <Suspense fallback={<div>Loading...</div>}>
      <WapClient initialData={data} />
    </Suspense>
  );
}
```

### `app/[[...pathname]]/context.ts` — Server Data Fetching

```ts
// app/[[...pathname]]/context.ts
import { Environment } from "@appaflytech/wappa-client/core/classes";
import { pageService, configService } from "@appaflytech/wappa-client/services";

const environment = new Environment();
environment.update({
  cdn: process.env.NEXT_PUBLIC_CDN || "",
  key: process.env.NEXT_PUBLIC_SITE_KEY || "",
  service: `${process.env.NEXT_PUBLIC_API}/${process.env.NEXT_PUBLIC_SITE_KEY}`,
  url: process.env.NEXT_PUBLIC_APP_URL || "",
  environment: (process.env.NEXT_PUBLIC_ENV as any) || "development",
});

export async function getPageData(path: string) {
  try {
    const [page, config] = await Promise.all([
      pageService.get(environment.context, { path, isMobile: false }),
      configService.get(environment.context, "tr"),
    ]);
    if (!page) return null;
    return { page, config, environment: environment.context };
  } catch {
    return null;
  }
}
```

### `app/[[...pathname]]/client.tsx` — Client Render

```tsx
"use client";
// app/[[...pathname]]/client.tsx
import { AppContextProvider } from "@appaflytech/wappa-client/core/contexts";
import { render } from "@/core/render";

interface WapClientProps {
  initialData: {
    page: any;
    config: any;
    environment: any;
  };
}

export default function WapClient({ initialData }: WapClientProps) {
  const { page, config, environment } = initialData;

  return (
    <AppContextProvider
      value={{
        page,
        language: "tr",
        theme: config.themes?.[0],
        themes: config.themes,
        settings: config.settings,
        environment,
      }}
    >
      <main>{render(page.layout, page.views)}</main>
    </AppContextProvider>
  );
}
```

---

## 25. UI Library Integration

### Web — Wappa Components with shadcn/ui

Use shadcn/ui components inside Wappa component wrappers:

```tsx
// components/wap/button/index.tsx (shadcn/ui versiyonu)
"use client";
import { Button as ShadButton } from "@/components/ui/button";
import Link from "next/link";

interface ButtonProps {
  children?: React.ReactNode;
  anchor?: { href?: string; target?: string; title?: string };
  appearance?: "filled" | "outlined" | "ghost";
  kind?: "primary" | "secondary" | "danger" | "success";
  disabled?: boolean;
}

const VARIANT_MAP: Record<string, any> = {
  filled: "default",
  outlined: "outline",
  ghost: "ghost",
};

const KIND_MAP: Record<string, any> = {
  primary: "default",
  secondary: "secondary",
  danger: "destructive",
};

export default function Button({
  children,
  anchor,
  appearance = "filled",
  kind = "primary",
  disabled,
}: ButtonProps) {
  const variant = KIND_MAP[kind] || VARIANT_MAP[appearance] || "default";

  if (anchor?.href) {
    return (
      <ShadButton variant={variant} disabled={disabled} asChild>
        <Link href={anchor.href} target={anchor.target || "_self"}>
          {children || anchor.title}
        </Link>
      </ShadButton>
    );
  }

  return (
    <ShadButton variant={variant} disabled={disabled}>
      {children}
    </ShadButton>
  );
}
```

### Mobile — NativeWind + gluestack-ui

Use gluestack-ui components inside Wappa component wrappers:

```tsx
// components/wap/button/Button.tsx (gluestack versiyonu)
import {
  Button as GSButton,
  ButtonText,
  ButtonSpinner,
} from "@gluestack-ui/themed";
import { Linking } from "react-native";

// ... (same props interface)

export default function Button({
  children,
  anchor,
  appearance = "filled",
  kind = "primary",
  disabled,
  loading,
}: ButtonProps) {
  const handlePress = () => {
    if (anchor?.href) Linking.openURL(anchor.href);
  };

  const actionMap: Record<string, any> = {
    primary: "primary",
    secondary: "secondary",
    danger: "negative",
  };

  return (
    <GSButton
      action={actionMap[kind] || "primary"}
      variant={
        appearance === "outlined"
          ? "outline"
          : appearance === "ghost"
            ? "link"
            : "solid"
      }
      isDisabled={disabled}
      onPress={handlePress}
    >
      {loading ? (
        <ButtonSpinner />
      ) : (
        <ButtonText>{children || anchor?.title || "Button"}</ButtonText>
      )}
    </GSButton>
  );
}
```

---

## 26. Defining New Components in Admin

To add a new component to the admin drag-drop builder, edit the `wappa-admin-ui/src/components/page-builder/definitions/elements.ts` file:

```ts
// Example to add to elements.ts
{
  name: 'my-card',
  id: 'my-card',
  title: 'Card',
  icon: 'credit-card',
  droppable: true,      // accepts child components
  isMobile: true,       // show on mobile too
  definition: {
    props: [
      {
        name: 'title',
        title: 'Title',
        type: 'text',
      },
      {
        name: 'description',
        title: 'Description',
        type: 'text',
      },
      {
        name: 'image',
        title: 'Image',
        type: 'file',       // opens file picker
      },
      {
        name: 'anchor',
        title: 'Link',
        type: 'link',       // opens link picker
      },
    ]
  }
}
```

### Prop Types

| `type` Value | Admin UI Control        |
| ------------ | ----------------------- |
| `text`       | Text input field        |
| `number`     | Number input field      |
| `boolean`    | Toggle/checkbox         |
| `select`     | Dropdown (with options) |
| `file`       | File/image picker       |
| `link`       | Link picker (page, URL) |
| `color`      | Color picker            |
| `query`      | Query picker            |
| `navigation` | Navigation picker       |
| `style`      | Style editor            |

---

## 27. Common Mistakes and Solutions

### ❌ Not Using `safeText`

```tsx
// WRONG — crashes if object received
<Text>{props.title}</Text>;

// CORRECT — always guarantees a string
import { safeText } from "../../utils/path";
<Text>{safeText(props.title)}</Text>;
```

### ❌ Adding `WapScreen` to Registry

```tsx
// WRONG — circular dependency!
// components/index.tsx
import WapScreen from './WapScreen';
case 'wap-screen': return WapScreen;

// CORRECT — direct import
import WapScreen from '@/components/WapScreen';
```

### ❌ Assuming `style` Prop Is a Plain Object

```tsx
// WRONG — wappa style object is nested
<View style={style} />

// CORRECT — merge inner objects
<View style={{
  ...style?.margin,
  ...style?.padding,
  ...style?.layout,
  ...style?.border,
  ...style?.background,
  ...(style || {}),
}} />
```

### ❌ Re-creating Environment Everywhere

```tsx
// WRONG — new Environment() in every service
const env = new Environment();

// CORRECT — singleton pattern
// Create once in services/contextService.ts and export
export const environment = new Environment();
```

### ❌ `refs` and `props` Priority on Mobile

```tsx
// WRONG — props overrides refs
<Component {...props} {...refs} />

// CORRECT — refs holds dynamic values, props holds static values
// refs spread first, props overrides (admin override)
<Component {...refs} {...props} />
```
