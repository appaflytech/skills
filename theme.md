---
name: wappa-skills:theme
description: Wappa CMS theme system integrated with gluestack-ui v4. Web uses CSS variables auto-injected by admin. Mobile uses ThemeProvider to map wappa mobileNeutrals to gluestack config.
---

# Wappa Theme System — gluestack-ui v4

---

## Wappa Theme Object

Themes are configured in the Wappa admin and delivered via `configService.get()`:

```ts
type WappaTheme = {
  id: string;
  name: string;
  // Mobile: color tokens
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

## Web — CSS Variables

On web, the Wappa admin injects CSS variables into `:root` automatically.
gluestack-ui v4 maps to these via Tailwind's CSS variable extension.

### Standard CSS Variables (auto-injected)

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

### Connecting to gluestack-ui Tokens (tailwind.config.js)

```js
// tailwind.config.js
module.exports = {
  content: ["./app/**/*.{ts,tsx}", "./components/**/*.{ts,tsx}"],
  presets: [require("@gluestack-ui/nativewind-utils/tailwind")],
  theme: {
    extend: {
      colors: {
        // Map wappa CSS vars to custom utilities if needed
        "wappa-primary": "var(--color-primary)",
        "wappa-background": "var(--color-background)",
      },
    },
  },
};
```

### gluestack-ui v4 Semantic Token Reference

**Always use these — never raw hex values:**

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

## Mobile — Gluestack Config Override (Optional)

If you need wappa theme colors to affect gluestack's built-in semantic tokens
(so `bg-primary-500` reflects the wappa primary), override the config:

```tsx
// components/ThemeProvider.tsx — extended version
import { GluestackUIProvider } from "@/components/ui/gluestack-ui-provider";
import { config } from "@gluestack-ui/config"; // base config

export function ThemeProvider({
  theme,
  children,
}: {
  theme?: any;
  children: React.ReactNode;
}) {
  const overriddenConfig = theme?.mobileNeutrals
    ? {
        ...config,
        tokens: {
          ...config.tokens,
          colors: {
            ...config.tokens.colors,
            // Map wappa primary → gluestack primary500
            primary400:
              theme.mobileNeutrals.primary || config.tokens.colors.primary400,
            primary500:
              theme.mobileNeutrals.primary || config.tokens.colors.primary500,
            backgroundLight0:
              theme.mobileNeutrals.background ||
              config.tokens.colors.backgroundLight0,
            backgroundDark950:
              theme.mobileNeutrals.background ||
              config.tokens.colors.backgroundDark950,
          },
        },
      }
    : config;

  return (
    // Re-provide with overridden config to affect all gluestack tokens
    <GluestackUIProvider config={overriddenConfig}>
      {children}
    </GluestackUIProvider>
  );
}
```

> **Note:** This re-renders all gluestack tokens with the theme colors. Use sparingly — only when the admin theme must directly control gluestack's primary/background colors.

---

## Dark Mode

### Web

```tsx
// app/layout.tsx
import { ThemeProvider } from "next-themes";

<ThemeProvider attribute="class" defaultTheme="system" enableSystem>
  <GluestackUIProvider mode={resolvedTheme === "dark" ? "dark" : "light"}>
    {children}
  </GluestackUIProvider>
</ThemeProvider>;
```

### Mobile

```tsx
// app/_layout.tsx
import { useColorScheme } from "react-native";

const colorScheme = useColorScheme();
<GluestackUIProvider mode={colorScheme === "dark" ? "dark" : "light"}>
```

In components, semantic tokens adapt automatically:

```tsx
// Automatically adapts: white in light, dark bg in dark mode
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
