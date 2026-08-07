---
name: wappa-skills:web
description: Next.js setup for Wappa Schema web projects. Framework-agnostic — use any UI library. The wappa-web reference uses plain Tailwind CSS (Next.js 15 / React 19), NOT gluestack. Covers project structure, App Router routing, data fetching (contextService + configService/pageService), the render system, and the component registry.
---

# Wappa Schema — Web (Next.js)

> **Reference stack:** `wappa-web` ships **Next.js 15.3 + React 19 + plain Tailwind CSS v3** — hand-written `className` strings in each component, real DOM output, `lucide-react` icons. **No gluestack, no NativeWind.** You may still swap in gluestack/shadcn/etc.; the props contract in `wappa-skills:components` is framework-agnostic.

---

## 1. Setup

### Install Dependencies

```bash
# Create Next.js project (App Router + Tailwind — matches the wappa-web reference)
npx create-next-app@latest my-wappa-web --typescript --tailwind --app
cd my-wappa-web

# Install Wappa SDK
npm install @appaflytech/wappa-client

# Icons used by the reference components
npm install lucide-react
```

> **Reference = plain Tailwind.** The `wappa-web` components are thin wrappers that
> map schema props to Tailwind utility strings by hand (no component library to
> install). If you prefer gluestack-ui/shadcn instead, install it now and implement
> the same contracts from `wappa-skills:components`.

### `.env.local`

```env
NEXT_PUBLIC_API=https://api.your-service.com
NEXT_PUBLIC_CDN=https://cdn.your-service.com
NEXT_PUBLIC_SITE_KEY=your-site-key
NEXT_PUBLIC_APP_URL=https://your-site.com
NEXT_PUBLIC_ENV=development
```

---

## 2. File Structure

```
my-wappa-web/
├── app/
│   ├── globals.css
│   ├── layout.tsx              # GluestackUIProvider
│   └── [[...pathname]]/
│       ├── page.tsx            # SSR entry
│       └── client.tsx          # AppContextProvider + render
├── components/
│   ├── index.tsx               # Component registry (getComponent)
│   └── ui/                     # Component implementations
│       ├── container/
│       ├── box/
│       ├── row/
│       ├── column/
│       ├── section/
│       ├── heading/
│       ├── paragraph/
│       ├── html/
│       ├── icon/
│       ├── image/
│       ├── video/
│       ├── iframe/
│       ├── button/
│       ├── link/
│       ├── fab/
│       ├── card/
│       ├── card-list/
│       ├── avatar/
│       ├── badge/
│       ├── divider/
│       ├── table/
│       ├── skeleton/
│       ├── spinner/
│       ├── alert/
│       ├── progress/
│       ├── toast/
│       ├── accordion/
│       ├── tabs/
│       ├── modal/
│       ├── drawer/
│       ├── actionsheet/
│       ├── menu/
│       ├── popover/
│       ├── alert-dialog/
│       ├── tooltip/
│       ├── form-control/
│       ├── input/
│       ├── select/
│       ├── switch/
│       ├── checkbox/
│       ├── radio/
│       ├── textarea/
│       ├── slider/
│       ├── calendar/
│       └── date-time-picker/
├── core/
│   └── render.tsx
└── services/
    └── contextService.ts       # API data fetching
```

---

## 3. Core Files

### `app/layout.tsx` — Root Layout

The `wappa-web` reference layout is minimal: it just imports `globals.css` and renders `<body>` (with `suppressHydrationWarning`). There is **no provider wrapper** — the theme is picked from the config and passed down through the Wappa `AppContextProvider` in `client.tsx`, and components read plain CSS variables. (If you chose gluestack, this is where you'd add `GluestackUIProvider`.)

```tsx
import "./globals.css";

export const metadata = {
  title: "wappa-web",
  description: "Powered by Wappa CMS",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body suppressHydrationWarning>{children}</body>
    </html>
  );
}
```

### `services/contextService.ts` — Data Fetching

```ts
import { Environment } from "@appaflytech/wappa-client/core/classes";
import { pageService, configService } from "@appaflytech/wappa-client/services";

const environment = new Environment();
environment.update({
  cdn: process.env.NEXT_PUBLIC_CDN ?? "",
  key: process.env.NEXT_PUBLIC_SITE_KEY ?? "",
  service: `${process.env.NEXT_PUBLIC_API}/${process.env.NEXT_PUBLIC_SITE_KEY}`,
  url: process.env.NEXT_PUBLIC_APP_URL ?? "",
  environment: (process.env.NEXT_PUBLIC_ENV as any) ?? "development",
});

const DEFAULT_LANGUAGE = "en-us";

function buildPath(pathname: string | null | undefined): string {
  const cleanPath = (pathname ?? "/").replace(/^\/+/, "") || "";
  return cleanPath.startsWith(DEFAULT_LANGUAGE)
    ? cleanPath
    : `${DEFAULT_LANGUAGE}/${cleanPath}`;
}

export const contextService = {
  async fetchConfig() {
    try {
      const config = await configService.get(environment.context, {
        language: DEFAULT_LANGUAGE,
      });
      return {
        settings: config.settings,
        themes: config.themes,
        language: DEFAULT_LANGUAGE,
        languages: config.languages,
        environment: environment.context,
      };
    } catch (error) {
      console.error("Wappa config error:", error);
      return null;
    }
  },

  async fetchPage(pathname: string | null | undefined) {
    try {
      const page = await pageService.get(environment.context, {
        path: buildPath(pathname),
        isMobile: false,
      });
      return page;
    } catch (error) {
      console.error("Wappa page error:", error);
      return null;
    }
  },
};
```

### `app/[[...pathname]]/page.tsx` — SSR Entry

```tsx
import { notFound } from "next/navigation";
import { contextService } from "@/services/contextService";
import ClientPage from "./client";

interface Props {
  params: Promise<{ pathname?: string[] }>;
}

export default async function Page({ params }: Props) {
  const { pathname: segments } = await params;
  const config = await contextService.fetchConfig();
  const homepage = config?.settings?.homepage ?? "";
  const pathname =
    segments?.join("/") || (typeof homepage === "string" ? homepage : "") || "";
  if (pathname.startsWith(".well-known") || pathname.startsWith("_next")) {
    notFound();
  }
  const page = await contextService.fetchPage(pathname);
  if (!page) notFound();
  return <ClientPage page={page} config={config} />;
}
```

### `app/[[...pathname]]/client.tsx` — Client Boundary

```tsx
"use client";
import { AppContextProvider } from "@appaflytech/wappa-client/core/contexts";
import { render } from "@/core/render";

interface Props {
  page: any;
  config: any;
}

export default function ClientPage({ page, config }: Props) {
  const value = config
    ? {
        container: "browser" as const,
        cookies: undefined,
        headers: undefined,
        environment: config.environment,
        language: config.language,
        languages: config.languages ?? [],
        page,
        params: {},
        settings: config.settings,
        theme:
          config.themes?.find((t: any) => t?.isDefault) ?? config.themes?.[0],
        themes: config.themes ?? [],
        customData: {},
        setData: () => {},
        setCustomData: () => {},
      }
    : null;

  return (
    <AppContextProvider value={value as any}>
      {render(page?.layout ?? [], page?.views ?? {})}
    </AppContextProvider>
  );
}
```

### `core/render.tsx`

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

---

## 4. Component Registry — `components/index.tsx`

Use **direct imports** (NOT `next/dynamic`). The registry is a `Record<string, ComponentType<any>>` object with `getComponent(name)` (returns `null` if missing), plus `registerComponents(custom)` (Object.assign, later wins) and `getRegistry()`.

The list below is a representative subset. The **actual `wappa-web` registry has ~76 keys** backed by ~67 `components/ui/*` dirs: it also includes the semantic HTML5 wrappers (`article`, `main`, `nav`, `aside`, `header`, `footer`, `ul`, `ol`, `li`), typography extras (`span`, `strong`, `em`, `blockquote`, `pre`, `code`, `time`), and the heading aliases `h1`–`h6` → `Headings`. `array-repeater` and `array-row` both map to `ArrayRepeater` from `@appaflytech/wappa-client/core/components`.

```tsx
import type { ComponentType } from "react";
import { ArrayRepeater } from "@appaflytech/wappa-client/core/components";

// Layout
import Container from "./ui/container/Container";
import Box from "./ui/box/Box";
import Center from "./ui/center/Center";
import HStack from "./ui/hstack/HStack";
import VStack from "./ui/vstack/VStack";
import Grid from "./ui/grid/Grid";
import Pressable from "./ui/pressable/Pressable";
import Row from "./ui/row/Row";
import Column from "./ui/column/Column";
import Section from "./ui/section/Section";
// Typography
import Heading from "./ui/heading/Heading";
import Paragraph from "./ui/paragraph/Paragraph";
import Html from "./ui/html/Html";
import Icon from "./ui/icon/Icon";
// Media
import Image from "./ui/image/Image";
import Video from "./ui/video/Video";
import Iframe from "./ui/iframe/Iframe";
// Interactive
import Button from "./ui/button/Button";
import Link from "./ui/link/Link";
import Fab from "./ui/fab/Fab";
// Display
import Card from "./ui/card/Card";
import CardList from "./ui/card-list/CardList";
import Avatar from "./ui/avatar/Avatar";
import Badge from "./ui/badge/Badge";
import Divider from "./ui/divider/Divider";
import Table from "./ui/table/Table";
import Skeleton from "./ui/skeleton/Skeleton";
// Feedback
import Spinner from "./ui/spinner/Spinner";
import Alert from "./ui/alert/Alert";
import Progress from "./ui/progress/Progress";
import Toast from "./ui/toast/Toast";
// Disclosure
import Accordion from "./ui/accordion/Accordion";
import Tabs from "./ui/tabs/Tabs";
// Overlay
import Modal from "./ui/modal/Modal";
import Drawer from "./ui/drawer/Drawer";
import Actionsheet from "./ui/actionsheet/Actionsheet";
import Menu from "./ui/menu/Menu";
import Popover from "./ui/popover/Popover";
import AlertDialog from "./ui/alert-dialog/AlertDialog";
import Tooltip from "./ui/tooltip/Tooltip";
// Form
import FormControl from "./ui/form-control/FormControl";
import Input from "./ui/input/Input";
import Select from "./ui/select/Select";
import Switch from "./ui/switch/Switch";
import Checkbox from "./ui/checkbox/Checkbox";
import Radio from "./ui/radio/Radio";
import Textarea from "./ui/textarea/Textarea";
import Slider from "./ui/slider/Slider";
import Calendar from "./ui/calendar/Calendar";
import DateTimePicker from "./ui/date-time-picker/DateTimePicker";

const registry: Record<string, ComponentType<any>> = {
  container: Container,
  box: Box,
  center: Center,
  hstack: HStack,
  vstack: VStack,
  grid: Grid,
  pressable: Pressable,
  row: Row,
  column: Column,
  section: Section,
  heading: Heading,
  paragraph: Paragraph,
  html: Html,
  icon: Icon,
  image: Image,
  video: Video,
  iframe: Iframe,
  button: Button,
  link: Link,
  fab: Fab,
  card: Card,
  "card-list": CardList,
  avatar: Avatar,
  badge: Badge,
  divider: Divider,
  table: Table,
  skeleton: Skeleton,
  spinner: Spinner,
  alert: Alert,
  progress: Progress,
  toast: Toast,
  accordion: Accordion,
  tabs: Tabs,
  modal: Modal,
  drawer: Drawer,
  actionsheet: Actionsheet,
  menu: Menu,
  popover: Popover,
  "alert-dialog": AlertDialog,
  tooltip: Tooltip,
  "form-control": FormControl,
  input: Input,
  select: Select,
  switch: Switch,
  checkbox: Checkbox,
  radio: Radio,
  textarea: Textarea,
  slider: Slider,
  calendar: Calendar,
  "date-time-picker": DateTimePicker,
  "array-repeater": ArrayRepeater,
  "array-row": ArrayRepeater, // alias — same component
};

export default function getComponent(name: string): ComponentType<any> | null {
  return registry[name] ?? null;
}
```

---

## 5. Plain-Tailwind Web Notes (reference)

- Each component in `components/ui/<name>/` is a hand-written wrapper: it maps schema props (`action`, `variant`, `size`, …) to Tailwind class strings and renders real DOM. Example: `Box` → `<div className>`, `Button` → `<a>` when `anchor.href` is set else `<button onClick>`.
- `render.tsx` does **no** data-binding or handler compilation — it just spreads `{...refs} {...props}` (refs first, props win). Dynamic data (queries, repeaters) is resolved inside the SDK's `mapPage` before render, and the `ArrayRepeater` core component + `AppContextProvider` handle the rest.
- Env: `contextService.ts` reads server-side overrides first (`WAPPA_API_URL`, `WAPPA_SITE_KEY`, `WAPPA_CDN_URL`, `WAPPA_APP_URL`) and falls back to the public `NEXT_PUBLIC_*` vars, so it works both in RSC and the browser.
- `next.config.js` only needs `images.domains` for your CDN host.

### Dark Mode

The reference relies on CSS variables + Tailwind's `dark:` classes (theme colors are injected as `--color-*`; see `wappa-skills:theme`). If you want system/OS switching, add `next-themes` and toggle a `data-theme`/`class` on `<html>`.

### Custom / gluestack alternative

- **Custom component:** add a direct import + a registry entry (or call `registerComponents({ "my-widget": MyWidget })`).
- **Prefer gluestack/shadcn?** Install it, generate the base components, and re-implement the same prop contracts — nothing in the render/registry/contextService layer changes.
