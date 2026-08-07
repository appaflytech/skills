---
name: wappa-skills:theme
description: Wappa CMS theme system. Themes come from configService.get() and carry both web tokens (colors/neutrals/fontSizes) and mobile tokens (mobileColors/mobileNeutrals/mobileFontSizes). Web reference = plain Tailwind + CSS variables; mobile reference = gluestack-ui v3 primitives + NativeWind v4 + a ThemeProvider that maps the mobile tokens.
---

# Wappa Theme System

> **Reference stacks:** web (`wappa-web`) themes with **plain Tailwind + CSS variables** (no gluestack); mobile (`wappa-mobile`) themes with **gluestack-ui v3 primitives + NativeWind v4** plus a `ThemeProvider` that reads the theme's `mobile*` tokens. The gluestack-specific sections below apply only if you chose gluestack.

---

## Wappa Theme Object

Themes are configured in the Wappa admin and delivered via `configService.get(env, { language })` (as `config.themes`). Each `Theme` carries **both** web and mobile token sets:

```ts
type WappaTheme = {
  id: string;
  name: string;
  isDefault?: boolean;
  // Web token sets
  colors?: Record<string, string>;
  neutrals?: Record<string, string>;
  fontSizes?: Record<string, number>;
  // Mobile token sets (mirror of the web ones)
  mobileColors?: Record<string, string>;
  mobileFontSizes: {
    xs?: number; sm?: number; md?: number; lg?: number; xl?: number; "2xl"?: number;
    [key: string]: number | undefined;
  };
  // Mobile: neutral/semantic color tokens
  mobileNeutrals: {
    primary?: string; // e.g. '#3b82f6'
    secondary?: string;
    background?: string;
    surface?: string;
    text?: string;
    textMuted?: string;
    error?: string;
    success?: string;
    warning?: string;
    info?: string;
    border?: string;
    [key: string]: string | undefined;
  };
  // Mobile: font size tokens
  mobileFontSizes: {
    xs?: number; // e.g. 12
    sm?: number; // e.g. 14
    md?: number; // e.g. 16
    lg?: number; // e.g. 18
    xl?: number; // e.g. 20
    "2xl"?: number; // e.g. 24
    [key: string]: number | undefined;
  };
  // Web: CSS variables injected into :root by Wappa admin layer
};
```

---

## Web — CSS Variables (plain Tailwind reference)

The `wappa-web` reference themes purely with **CSS variables + Tailwind** — no gluestack, no token engine. The theme colors are exposed as `--color-*` variables and your hand-written component classes read them (e.g. `style={{ background: "var(--color-primary)" }}` or a Tailwind color mapped to the var). Resolve the active theme from `config.themes` and set the vars on `:root` (or a wrapper) yourself.

### Standard CSS Variables

```css
:root {
  --color-primary: #3b82f6;
  --color-secondary: #64748b;
  --color-background: #ffffff;
  --color-foreground: #1f2937;
  --color-error: #ef4444;
  --color-success: #22c55e;
  --color-warning: #f59e0b;
  --color-info: #3b82f6;
  --color-border: #e5e7eb;
  --color-muted: #6b7280;
}
```

### Mapping CSS vars to Tailwind utilities (web, plain Tailwind)

```js
// tailwind.config.js — stock Tailwind v3 in the web reference (no gluestack preset)
module.exports = {
  content: ["./app/**/*.{ts,tsx}", "./components/**/*.{ts,tsx}"],
  theme: {
    extend: {
      colors: {
        // Expose the injected CSS vars as Tailwind color utilities
        "wappa-primary": "var(--color-primary)",
        "wappa-background": "var(--color-background)",
      },
    },
  },
};
```

### gluestack Semantic Token Reference (mobile / if you use gluestack)

Only relevant if you implement components with **gluestack-ui v3 + NativeWind** (the mobile reference). With plain Tailwind you use your own utilities instead. **Prefer semantic classes over raw hex:**

| className Token       | Maps to                       |
| --------------------- | ----------------------------- |
| `text-foreground`     | `--foreground` (primary text) |
| `text-typography-500` | Muted/secondary text          |
| `text-typography-400` | Placeholder text              |
| `bg-background-0`     | Page background               |
| `bg-background-50`    | Subtle card background        |
| `bg-background-100`   | Input background              |
| `bg-primary-500`      | Primary brand color           |
| `bg-primary-400`      | Primary hover state           |
| `bg-error-500`        | Error/destructive             |
| `bg-success-500`      | Success                       |
| `bg-warning-500`      | Warning                       |
| `bg-info-500`         | Info                          |
| `border-outline-200`  | Light border                  |
| `border-outline-300`  | Default border                |
| `border-outline-400`  | Strong border                 |

---

## Mobile — ThemeProvider

Maps Wappa `mobileNeutrals` and `mobileFontSizes` into React context.
Components can read theme values via `useTheme()`.

### `components/ThemeProvider.tsx`

```tsx
import React, { createContext, useContext } from "react";

interface ThemeContextValue {
  getColor: (key: string, fallback?: string) => string;
  getFontSize: (key: string, fallback?: number) => number;
  theme?: any;
}

const ThemeContext = createContext<ThemeContextValue>({
  getColor: (_, fb) => fb || "",
  getFontSize: (_, fb) => fb || 16,
});

export function ThemeProvider({
  theme,
  children,
}: {
  theme?: any;
  children: React.ReactNode;
}) {
  const getColor = (key: string, fallback = ""): string =>
    theme?.mobileNeutrals?.[key] || fallback;

  const getFontSize = (key: string, fallback = 16): number =>
    theme?.mobileFontSizes?.[key] || fallback;

  return (
    <ThemeContext.Provider value={{ getColor, getFontSize, theme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export const useTheme = () => useContext(ThemeContext);
```

### Usage in a Component

```tsx
import { useTheme } from "../ThemeProvider";
import { Box } from "@/components/ui/box";

function ThemedBanner() {
  const { getColor } = useTheme();
  return (
    <Box style={{ backgroundColor: getColor("primary", "#3b82f6") }}>
      {/* content */}
    </Box>
  );
}
```

---

## Mobile — driving gluestack/NativeWind colors from the theme

> **Do NOT use the legacy `@gluestack-ui/config` + `<GluestackUIProvider config={...}>` token-override API.** That is gluestack v2/v3-legacy `config`; the mobile reference styles with **gluestack v3 primitives + NativeWind v4 (className)**, so theme colors flow through **CSS variables / NativeWind**, not a `tokens` object.

To make NativeWind semantic classes (`bg-primary-500`, `bg-background-0`, …) reflect the admin theme, write the theme's `mobileNeutrals`/`mobileColors` into the NativeWind CSS variables that `tailwind.config.js` maps those classes to — e.g. via `vars()` from `nativewind` on a wrapper `View`, or by setting the variables in `global.css` and swapping values per active theme. Then components keep using plain `className` and adapt automatically.

For components that need a raw value (inline `style`, chart colors), read it from the `ThemeProvider` above (`getColor("primary")` / `getFontSize("md")`).

---

## Dark Mode

### Web (plain Tailwind)

Add `next-themes`, switch a `class`/`data-theme` on `<html>`, and define light/dark values for the `--color-*` variables. Tailwind `dark:` utilities and the CSS vars then adapt automatically:

```tsx
// app/layout.tsx
import { ThemeProvider } from "next-themes";

<ThemeProvider attribute="class" defaultTheme="system" enableSystem>
  {children}
</ThemeProvider>;
```

### Mobile (gluestack v3 + NativeWind)

NativeWind dark mode is `class`-based. Follow the device scheme (or the admin theme) and toggle it; semantic `className`s adapt:

```tsx
// app/_layout.tsx
import { useColorScheme } from "react-native";
import { colorScheme } from "nativewind";

const scheme = useColorScheme();
colorScheme.set(scheme ?? "light"); // drives NativeWind dark: variants
```

In components, semantic classes adapt automatically:

```tsx
// white in light, dark bg in dark mode
<Box className="bg-background-0">
<Text className="text-foreground">
```

---

## Theme Selection UI

If your app supports multiple themes (selectable in admin), resolve the active theme:

```tsx
// In WapScreen or _layout
const { themes, setTheme } = useAppStore();
const activeTheme = themes?.find((t) => t.id === page.theme) || themes?.[0];

<ThemeProvider theme={activeTheme}>{/* content */}</ThemeProvider>;
```
