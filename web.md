---
name: wappa-skills:web
description: Next.js setup for Wappa Schema web projects. Framework-agnostic — use any UI library. gluestack-ui v4 is the default. Covers project structure, GluestackUIProvider (optional), routing, data fetching, and the full component registry.
---

# Wappa Schema — Web (Next.js)

> **UI Framework:** This guide shows gluestack-ui v4 as the default. You can swap it for shadcn/ui, Tailwind CSS, or any other library. See `wappa-skills:components` for framework-agnostic component contracts.

---

## 1. Setup

### Install Dependencies

```bash
# Create Next.js project
npx create-next-app@latest my-wappa-web --typescript --tailwind --app
cd my-wappa-web

# Install Wappa SDK
npm install @appaflytech/wappa-client

# Install gluestack-ui v4 (default — skip if using a different UI framework)
npx gluestack-ui@latest init -y
npx gluestack-ui@latest add --all -y
```

> **Using a different UI framework?** Skip the gluestack-ui steps above.
> Install your preferred library (shadcn/ui, Tailwind CSS, etc.) and implement
> components using the contracts defined in `wappa-skills:components`.

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

### `app/layout.tsx` — Root Layout with GluestackUIProvider

```tsx
import { GluestackUIProvider } from "@/components/ui/gluestack-ui-provider";
import "./globals.css";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <GluestackUIProvider mode="light">{children}</GluestackUIProvider>
      </body>
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

Use **direct imports** (NOT `next/dynamic`). The registry is a `Record<string, ComponentType<any>>` object.

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

## 5. gluestack-ui v4 Web Notes

- `GluestackUIProvider` wraps the entire app in `app/layout.tsx`
- Use `mode="light"` or `mode="dark"` — integrate with next-themes for system detection
- All gluestack components are in `components/ui/` (generated by CLI, do not edit)
- Use `asChild` prop on `Button` to render as `<Link>` for navigation buttons
- For SSR: gluestack-ui v4 is RSC-compatible when used with `"use client"` boundary in client.tsx

### Dark Mode with next-themes

```tsx
// app/layout.tsx
import { ThemeProvider } from "next-themes";
// Wrap GluestackUIProvider with ThemeProvider
// Pass mode based on resolved theme
```

### Custom Component Registration

Add a direct import + registry entry:

```tsx
import MyWidget from "./ui/my-widget/MyWidget";
// in registry object:
"my-widget": MyWidget,
```
