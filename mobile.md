---
name: wappa-skills:mobile
description: Expo React Native setup for Wappa Schema mobile projects. The reference (wappa-mobile) uses Expo SDK 57 + RN 0.86 + React 19, gluestack-ui v3 primitives + NativeWind v4, expo-router, Zustand v5, and a shared source-only engine package @appaflytech/wappa-mobile-ui (registry + render + navigation). Covers the two-package model, the injection seams (setWappaStore / useWappaConfig / registerComponents), WapScreen, contextService, and the handler-compiling render.
---

# Wappa Schema — Mobile (Expo React Native)

> **Reference stack:** `wappa-mobile` ships **Expo SDK 57 · RN 0.86 · React 19**, UI = **gluestack-ui v3 primitives + NativeWind v4** (NOT gluestack v4), navigation = **expo-router** (file-based) wrapped by a Wappa-driven `DynamicNavigation`, state = **Zustand v5**, icons = `lucide-react-native`. The props contract in `wappa-skills:components` is framework-agnostic; this guide documents what the reference actually does.

---

## 0. Two-package architecture (important)

The reference mobile is split into **two packages**:

| Package | Role |
| ------- | ---- |
| **`wappa-mobile/`** (the app) | Routes (`app/*`), the **app-owned** pieces — `WapScreen`, `services/contextService.ts`, `store/store.ts`, custom components — plus native `ios/`+`android/`, `.env`, EAS/Docker. Depends on `@appaflytech/wappa-mobile-ui`. |
| **`@appaflytech/wappa-mobile-ui`** (shared, source-only) | The **engine**: component registry, `render()`, wrappers, smart overlays, `DynamicNavigation`, the `useWappaConfig` store, and `setWappaStore`. `main`/`source`/`react-native` all point at raw `index.tsx` — **no build step**; Metro compiles it in-app. |

Three **injection seams** connect them (the engine holds no app singletons):

- **`setWappaStore(useAppStore)`** — the app hands its Zustand store to the engine (called once at module load in `store/store.ts`).
- **`useWappaConfig`** — a Zustand config store (`cdn` / `api` / `key` / `env`) the app populates from `EXPO_PUBLIC_*`; media components read `cdn`, services read all four.
- **`registerComponents({...})`** — the app extends/overrides the registry (e.g. `components/custom/index.ts`, which **wappa-mcp auto-generates** via `create_custom_component`).

Metro remaps the package to the sibling **source** dir for live edits (`resolver.extraNodeModules` + a `watchFolders` entry), falling back to the published `node_modules` copy in CI/Docker. `tailwind.config.js` also globs `../wappa-mobile-ui/**` so its classes are scanned.

> **Building a new app?** The fastest correct path is to **depend on `@appaflytech/wappa-mobile-ui`** and only author the app-owned pieces below. Re-implementing the engine by hand is possible but not recommended.

---

## 1. Setup

```bash
# Create Expo project (expo-router template)
npx create-expo-app@latest my-wappa-mobile
cd my-wappa-mobile

# Wappa SDK (logic) + the shared UI/engine package
npm install @appaflytech/wappa-client @appaflytech/wappa-mobile-ui

# State + storage
npm install zustand @react-native-async-storage/async-storage

# UI: gluestack-ui v3 primitives + NativeWind v4
npm install @gluestack-ui/core @gluestack-ui/utils nativewind
npm install -D tailwindcss

# Icons + common native deps used by the registry
npm install lucide-react-native @expo/vector-icons \
  react-native-reanimated react-native-gesture-handler \
  @gorhom/bottom-sheet react-native-safe-area-context
```

Wire NativeWind: `global.css` with the three `@tailwind` directives, `metro.config.js` `withNativeWind({ input: "./global.css" })`, and the `nativewind/babel` preset in `babel.config.js`. Dark mode = `class`.

### `.env`

```env
EXPO_PUBLIC_WAP_API=https://wappa-ui-api.appaflytech.com
EXPO_PUBLIC_WAP_CDN=https://minio.appaflytech.com/wappa-storage
EXPO_PUBLIC_WAP_SITE_KEY=your-site-key
EXPO_PUBLIC_ENV=development
```

These are consumed by `useWappaConfig` (from `@appaflytech/wappa-mobile-ui`): `api` → `service = ${api}/${key}`, `cdn` → asset base, `key` → tenant/site key, `env` → environment label.

---

## 2. File Structure (app-owned)

```
my-wappa-mobile/
├── app/
│   ├── _layout.tsx                # providers + DynamicNavigation(settings.navigation) around <Stack>
│   ├── index.tsx                  # <WapScreen fixedPath="/" />
│   ├── [...pathname].tsx          # catch-all → <WapScreen fixedPath={joined} />
│   └── (auth)/login.tsx
├── components/
│   ├── WapScreen.tsx              # page loader + render (app-owned)
│   └── custom/index.ts            # registerComponents({...}) — wappa-mcp generated
├── services/
│   └── contextService.ts          # Environment + fetchConfig/fetchPage
├── store/
│   └── store.ts                   # Zustand useAppStore + setWappaStore(useAppStore)
├── global.css · metro.config.js · babel.config.js · tailwind.config.js · app.json · .env
```

The registry, `render()`, navigation and UI components live in **`@appaflytech/wappa-mobile-ui`**, not in the app.

---

## 3. App-owned infrastructure

### `store/store.ts` — Zustand + engine binding

```ts
import { create } from "zustand";
import { setWappaStore } from "@appaflytech/wappa-mobile-ui";
import type { PageComponent } from "@appaflytech/wappa-client/constants/types";

interface AppStore {
  isLoading: boolean;
  isInitialized: boolean;
  isConfigLoaded: boolean;
  page?: { id: number; title: string; path: string; theme?: string;
           layout: PageComponent[]; views: Record<string, PageComponent[]> };
  settings?: any; themes?: any[]; theme?: any;
  language?: string; languages?: any[]; environment?: any;
  authProviders?: { googleEnabled: boolean; appleEnabled: boolean; googleWebClientId?: string };
  wapNavigate: ((path: string) => void) | null;

  setWapNavigate: (fn: ((path: string) => void) | null) => void;
  setPage: (page: AppStore["page"]) => void;
  setLoading: (v: boolean) => void;
  setConfig: (c: Partial<AppStore>) => void;   // also flips isConfigLoaded
  initialize: (c: { page: AppStore["page"] }) => void; // sets page + isInitialized
  reset: () => void;
}

export const useAppStore = create<AppStore>((set) => ({
  isLoading: false, isInitialized: false, isConfigLoaded: false, wapNavigate: null,
  setWapNavigate: (wapNavigate) => set({ wapNavigate }),
  setPage: (page) => set({ page }),
  setLoading: (isLoading) => set({ isLoading }),
  setConfig: (c) => set({ ...c, isConfigLoaded: true }),
  initialize: ({ page }) => set({ page, isLoading: false, isInitialized: true }),
  reset: () => set({ page: undefined, isInitialized: false }),
}));

// Hand the store to the render engine (handlers use store.get/set, wapNavigate, etc.)
setWappaStore(useAppStore);
```

### `services/contextService.ts` — data fetching

```ts
import { Environment } from "@appaflytech/wappa-client/core/classes";
import { pageService, configService } from "@appaflytech/wappa-client/services";
import { useWappaConfig } from "@appaflytech/wappa-mobile-ui";

const DEFAULT_LANGUAGE = "en-us";

function makeEnv() {
  const { cdn, api, key, env } = useWappaConfig.getState();
  const environment = new Environment();
  environment.update({ cdn, key, service: `${api}/${key}`, url: "", environment: env });
  return environment;
}

const buildPath = (p: string) => {
  const clean = (p ?? "/").replace(/^\/+/, "");
  return clean.startsWith(DEFAULT_LANGUAGE) ? clean : `${DEFAULT_LANGUAGE}/${clean}`;
};

export const contextService = {
  async fetchConfig() {
    const env = makeEnv();
    if (!env.context.key) return null; // site key not set yet (e.g. QR not scanned)
    // NOTE: configService.get takes an OBJECT { language } — NOT a bare string.
    return configService.get(env.context, { language: DEFAULT_LANGUAGE });
    // → { settings, themes, language, languages, authProviders, environment }
  },
  async fetchPage(pathname: string) {
    const env = makeEnv();
    return pageService.get(env.context, { path: buildPath(pathname), isMobile: true });
  },
};

// Multi-tenant factory: same two methods, key/service from an explicit siteKey.
export function createContextService(siteKey: string, overrides?: { api?: string; cdn?: string }) {
  /* build an Environment from siteKey + overrides, return { fetchConfig, fetchPage } */
}
```

### `render()` lives in the engine — how it works

You import `render` from `@appaflytech/wappa-mobile-ui`; you do **not** write it. Unlike the web render, the **mobile render compiles handler values** and normalizes styles:

- **Handlers:** any prop whose value `isHandlerValue(v)` (shape `{ __handler: true, code: "..." }`) is compiled via `createHandlerCompiler(getMobileHandlerContext)` (from `@appaflytech/wappa-client/core/utils`). The injected RN context exposes: `router` (prefers `store.wapNavigate` for in-app nav), `form` (formBus), `api`, `store` (`getWappaStore().getState/setState`), `alert`, `storage` (AsyncStorage/JSON), `clipboard`, `link` (open/call/mail/sms/maps), `device`, `validate`, `data` (dataBus), `ui` (uiBus open/close/toggle), `share`, `haptics`, `permissions`, `notifications`.
- **`view` slots:** children come from `views[id]` (with a single-slot fallback if the slot id changed after a layout edit).
- **Style:** `normalizeStyle` drops the legacy admin nested style shape; mobile relies on NativeWind `className`.
- **Props order:** `{ ...refs, ...mappingProps, ...compiledProps, ...listProps }` — refs first, admin/compiled values win.
- **Lists** (`flat-list`, `section-list`, `virtualized-list`, `carousel`) receive a `_renderTemplate` callback that recurses through the same pipeline.

Signature: `render(componentList, views, isMappingRender = false)`.

---

## 4. Component Registry (in the engine package)

The registry is a **`Record<string, ComponentType>`** in `@appaflytech/wappa-mobile-ui/components/index.tsx` (**~130 keys → ~90 modules**, 84 `components/ui/*` dirs). API: `getComponent(name)` (returns `null` if missing), `registerComponents(custom)` (Object.assign, later wins), `getRegistry()`. `array-repeater` is imported from `@appaflytech/wappa-client/core/components`.

Notable aliases the engine registers: `array-row` → ArrayRepeater; `view` → View; `text-input` → WapInput; `tab-view` → WapTabs; and the web-only wrappers `container`/`column`/`section`/`html`/`iframe` → Box (so web-authored pages still render on mobile).

**Extend the registry from the app** (do not edit the package):

```ts
// components/custom/index.ts  — wappa-mcp generates this; imported for side-effect in _layout.tsx
import { registerComponents } from "@appaflytech/wappa-mobile-ui";
import MyWidget from "./MyWidget";

registerComponents({ "my-widget": MyWidget });
```

---

## 5. WapScreen — `components/WapScreen.tsx` (app-owned)

```tsx
import React, { useEffect, useMemo } from "react";
import { ActivityIndicator, Text, View } from "react-native";
import { render } from "@appaflytech/wappa-mobile-ui";
import { useAppStore } from "../store/store";
import { contextService } from "../services/contextService";

interface WapScreenProps { siteKey?: string; stripPrefix?: string; fixedPath?: string }

export default function WapScreen({ fixedPath }: WapScreenProps) {
  const pathname = fixedPath ?? "/";
  const { page, isLoading, isInitialized, isConfigLoaded, settings, setConfig, initialize } = useAppStore();

  // fetch config once
  useEffect(() => {
    if (isConfigLoaded) return;
    contextService.fetchConfig().then((c) => c && setConfig(c));
  }, [isConfigLoaded]);

  // fetch the page when the path changes
  useEffect(() => {
    if (pathname === "/") return;
    contextService.fetchPage(pathname).then((page) => page && initialize({ page }));
  }, [pathname]);

  if (!isInitialized || isLoading) return <ActivityIndicator />;
  if (!page) return <View style={{ flex: 1 }}><Text>Page not found</Text></View>;
  return <>{render(page.layout ?? [], page.views ?? {})}</>;
}
```

Routes pass the path explicitly (`fixedPath`) so backgrounded stack screens don't react to other routes: `app/index.tsx` → `<WapScreen fixedPath="/" />`; `app/[...pathname].tsx` joins the segments.

---

## 6. App entry — `app/_layout.tsx`

The root layout mounts the provider stack and wraps expo-router's `<Stack>` in the Wappa-driven `DynamicNavigation` (stack/tab/drawer, driven by `settings.navigation`). It also imports `@/components/custom` for side-effect registration.

```tsx
import "@/global.css";
import "@/components/custom";            // side-effect: registerComponents(...)
import { Stack } from "expo-router";
import { GestureHandlerRootView } from "react-native-gesture-handler";
import { SafeAreaProvider } from "react-native-safe-area-context";
import { BottomSheetModalProvider } from "@gorhom/bottom-sheet";
import { DynamicNavigation } from "@appaflytech/wappa-mobile-ui";
import { useAppStore } from "@/store/store";

export default function RootLayout() {
  const settings = useAppStore((s) => s.settings);
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <SafeAreaProvider>
        <BottomSheetModalProvider>
          <DynamicNavigation navigation={settings?.navigation}>
            <Stack screenOptions={{ headerShown: false }} />
          </DynamicNavigation>
        </BottomSheetModalProvider>
      </SafeAreaProvider>
    </GestureHandlerRootView>
  );
}
```

(For social login/push, wrap in `WappaAuthProvider` and register the FCM token — see the `wappa-auth` / `wappa-notifications` skills.)

---

## 7. Key Mobile Rules

| Rule | Reason |
| ---- | ------ |
| Depend on `@appaflytech/wappa-mobile-ui` — don't reimplement the engine | It owns the registry, render, handler compilation, navigation, overlays |
| Call `setWappaStore(useAppStore)` once at load | Handlers read/write app state via `store` and `wapNavigate` |
| Populate `useWappaConfig` from `EXPO_PUBLIC_*` before fetching | `service`, `cdn`, `key`, `env` all come from it |
| `configService.get(ctx, { language })` — object, not a string | Passing a bare string is the classic bug |
| `pageService.get(ctx, { path, isMobile: true })` | Fetches the mobile-tailored layout |
| Default language is `"en-us"` (not `"en"`) | Matches `buildPath` + the backend |
| Extend the registry via `registerComponents({...})` | Never edit the shared package; `components/custom` is wappa-mcp-generated |
| Web-only components (`iframe`, `article`, `h1`–`h6`, …) alias to Box or render null | RN has no DOM |
