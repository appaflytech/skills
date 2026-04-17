---
name: wappa-schema:components
description: Complete component reference for Wappa Schema projects. Props derived from admin schema (elements.ts/constants.ts). All implementations use gluestack-ui v4 for both web and mobile.
---

# Wappa Components — gluestack-ui v4 Reference

All components receive props via `{...refs, ...mappedValue, ...props}` spread from the Wappa render system.

> **Mobile rule:** Always use `safeText(value)` for string props — the render system may pass unresolved ref objects.

---

## Layout Components

### `container` — Page Width Wrapper

**Admin Props (from `CONTAINER_PROPS`):**

```ts
interface ContainerProps {
  size?: "fluid" | "extended" | "wide" | "medium" | "narrow";
  className?: string;
  children?: React.ReactNode;
}
```

**Web — `components/wap/container/index.tsx`:**

```tsx
import { Box } from "@/components/ui/box";

const SIZE_CLASSES: Record<string, string> = {
  fluid: "w-full",
  extended: "max-w-[1400px] mx-auto px-4",
  wide: "max-w-[1200px] mx-auto px-4",
  medium: "max-w-[900px] mx-auto px-4",
  narrow: "max-w-[640px] mx-auto px-4",
};

export default function Container({
  size = "wide",
  className,
  children,
}: ContainerProps) {
  return (
    <Box
      className={`${SIZE_CLASSES[size] || SIZE_CLASSES.wide} ${className || ""}`}
    >
      {children}
    </Box>
  );
}
```

**Mobile — `components/wap/container/Container.tsx`:**

```tsx
import { Box } from "@/components/ui/box";

export default function Container({ className, children }: ContainerProps) {
  return (
    <Box className={`flex-1 w-full px-4 ${className || ""}`}>{children}</Box>
  );
}
```

---

### `row` — Flex Row (Responsive)

**Admin Props:**

```ts
interface BreakpointConfig {
  direction?: "row" | "row-reverse" | "column" | "column-reverse";
  align?: "start" | "center" | "end" | "baseline" | "stretch";
  justify?: "start" | "center" | "end" | "around" | "between" | "evenly";
  wrap?: "nowrap" | "wrap" | "wrap-reverse";
  gutter?: "none" | "xs" | "sm" | "md" | "lg" | "xl";
}
interface RowProps {
  xs?: BreakpointConfig;
  sm?: BreakpointConfig;
  md?: BreakpointConfig;
  lg?: BreakpointConfig;
  xl?: BreakpointConfig;
  className?: string;
  children?: React.ReactNode;
}
```

**Web — `components/wap/row/index.tsx`:**

```tsx
import { Box } from "@/components/ui/box";

const JUSTIFY: Record<string, string> = {
  start: "justify-start",
  center: "justify-center",
  end: "justify-end",
  around: "justify-around",
  between: "justify-between",
  evenly: "justify-evenly",
};
const ALIGN: Record<string, string> = {
  start: "items-start",
  center: "items-center",
  end: "items-end",
  baseline: "items-baseline",
  stretch: "items-stretch",
};
const GUTTER: Record<string, string> = {
  none: "gap-0",
  xs: "gap-1",
  sm: "gap-2",
  md: "gap-4",
  lg: "gap-6",
  xl: "gap-8",
};
const DIR: Record<string, string> = {
  row: "flex-row",
  "row-reverse": "flex-row-reverse",
  column: "flex-col",
  "column-reverse": "flex-col-reverse",
};

function bpClasses(bp?: BreakpointConfig, prefix = "") {
  if (!bp) return "";
  const p = prefix ? `${prefix}:` : "";
  return [
    bp.direction && `${p}${DIR[bp.direction]}`,
    bp.align && `${p}${ALIGN[bp.align]}`,
    bp.justify && `${p}${JUSTIFY[bp.justify]}`,
    bp.gutter && `${p}${GUTTER[bp.gutter]}`,
    bp.wrap === "wrap"
      ? `${p}flex-wrap`
      : bp.wrap === "nowrap"
        ? `${p}flex-nowrap`
        : "",
  ]
    .filter(Boolean)
    .join(" ");
}

export default function Row({
  xs,
  sm,
  md,
  lg,
  xl,
  className,
  children,
}: RowProps) {
  return (
    <Box
      className={`flex ${bpClasses(xs)} ${bpClasses(sm, "sm")} ${bpClasses(md, "md")} ${bpClasses(lg, "lg")} ${bpClasses(xl, "xl")} ${className || ""}`}
    >
      {children}
    </Box>
  );
}
```

**Mobile — `components/wap/row/Row.tsx`:**

```tsx
import { HStack } from "@/components/ui/hstack";
import { VStack } from "@/components/ui/vstack";

const SPACE: Record<string, string> = {
  none: "gap-0",
  xs: "gap-1",
  sm: "gap-2",
  md: "gap-4",
  lg: "gap-6",
  xl: "gap-8",
};

export default function Row({ xs, className, children }: RowProps) {
  const dir = xs?.direction;
  const gapClass = xs?.gutter ? SPACE[xs.gutter] : "";
  const isVertical = dir === "column" || dir === "column-reverse";
  if (isVertical)
    return (
      <VStack className={`${gapClass} ${className || ""}`}>{children}</VStack>
    );
  return (
    <HStack className={`${gapClass} flex-wrap ${className || ""}`}>
      {children}
    </HStack>
  );
}
```

---

### `column` — Grid Column

**Admin Props:**

```ts
interface ColumnBreakpoint {
  size?: 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12;
  align?: "start" | "center" | "end";
  offset?: number;
  order?: "first" | "last";
}
interface ColumnProps {
  xs?: ColumnBreakpoint;
  sm?: ColumnBreakpoint;
  md?: ColumnBreakpoint;
  lg?: ColumnBreakpoint;
  xl?: ColumnBreakpoint;
  className?: string;
  children?: React.ReactNode;
}
```

**Web — `components/wap/column/index.tsx`:**

```tsx
import { Box } from "@/components/ui/box";

function colClass(bp?: ColumnBreakpoint, prefix = "") {
  if (!bp) return "";
  const p = prefix ? `${prefix}:` : "";
  const ALIGN: Record<string, string> = {
    start: "self-start",
    center: "self-center",
    end: "self-end",
  };
  return [
    bp.size && `${p}w-${bp.size}/12`,
    bp.align && `${p}${ALIGN[bp.align]}`,
    bp.offset && `${p}ml-${bp.offset}/12`,
    bp.order === "first"
      ? `${p}order-first`
      : bp.order === "last"
        ? `${p}order-last`
        : "",
  ]
    .filter(Boolean)
    .join(" ");
}

export default function Column({
  xs,
  sm,
  md,
  lg,
  xl,
  className,
  children,
}: ColumnProps) {
  return (
    <Box
      className={`${colClass(xs)} ${colClass(sm, "sm")} ${colClass(md, "md")} ${colClass(lg, "lg")} ${colClass(xl, "xl")} ${className || ""}`}
    >
      {children}
    </Box>
  );
}
```

**Mobile — `components/wap/column/Column.tsx`:**

```tsx
import { Box } from "@/components/ui/box";

export default function Column({ xs, className, children }: ColumnProps) {
  const flexValue = (xs?.size ?? 12) / 12;
  return (
    <Box className={className || ""} style={{ flex: flexValue }}>
      {children}
    </Box>
  );
}
```

---

### `section` — Section Wrapper

**Admin Props:**

```ts
interface SectionProps {
  title?: string;
  description?: string;
  anchor?: { href?: string; target?: string; title?: string };
  container?: { size?: string };
  children?: React.ReactNode;
}
```

**Web — `components/wap/section/index.tsx`:**

```tsx
import { Box } from "@/components/ui/box";
import Container from "../container";

export default function Section({ container, children }: SectionProps) {
  return (
    <Box as="section">
      <Container size={container?.size as any}>{children}</Container>
    </Box>
  );
}
```

**Mobile:** Simple wrapper — `<Box className="w-full">{children}</Box>`

---

## Content Components

### `heading` — Heading

**Admin Props:**

```ts
interface HeadingProps {
  children?: string;
  nodeType?: "h1" | "h2" | "h3" | "h4" | "h5" | "h6";
  size?: "xs" | "sm" | "md" | "lg" | "xl" | "2xl" | "3xl" | "4xl" | "5xl";
  numberOfLines?: number;
  className?: string;
}
```

**Web & Mobile:**

```tsx
import { Heading } from "@/components/ui/heading";
import { safeText } from "../../utils/path"; // mobile only

export default function WapHeading({
  children,
  nodeType = "h2",
  size = "xl",
  numberOfLines,
  className,
}: HeadingProps) {
  return (
    <Heading
      as={nodeType}
      size={size}
      numberOfLines={numberOfLines}
      className={className}
    >
      {safeText(children)} {/* use safeText on mobile */}
    </Heading>
  );
}
```

---

### `paragraph` — Paragraph Text

**Admin Props:**

```ts
interface ParagraphProps {
  children?: string;
  size?: "2xs" | "xs" | "sm" | "md" | "lg" | "xl" | "2xl";
  bold?: boolean;
  italic?: boolean;
  isTruncated?: boolean;
  className?: string;
}
```

**Web & Mobile:**

```tsx
import { Text } from "@/components/ui/text";

export default function Paragraph({
  children,
  size = "md",
  bold,
  italic,
  isTruncated,
  className,
}: ParagraphProps) {
  return (
    <Text
      size={size}
      bold={bold}
      italic={italic}
      isTruncated={isTruncated}
      className={className}
    >
      {safeText(children)} {/* use safeText on mobile */}
    </Text>
  );
}
```

---

### `html` — Raw HTML Content

**Admin Props:** `content?: string`

**Web:**

```tsx
export default function Html({ content }: { content?: string }) {
  if (!content) return null;
  return <div dangerouslySetInnerHTML={{ __html: content }} />;
}
```

**Mobile:** Use `react-native-render-html`:

```tsx
import RenderHtml from "react-native-render-html";
import { useWindowDimensions } from "react-native";

export default function Html({ content }: { content?: string }) {
  const { width } = useWindowDimensions();
  if (!content) return null;
  return <RenderHtml contentWidth={width} source={{ html: content }} />;
}
```

---

### `image` — Image

**Admin Props:**

```ts
interface WapImageProps {
  source?: { src?: string; alt?: string };
  alt?: string;
  size?: "2xs" | "xs" | "sm" | "md" | "lg" | "xl" | "2xl" | "full" | "none";
  resizeMode?: "cover" | "contain" | "stretch" | "center";
  rounded?: number;
  className?: string;
}
```

**Web:**

```tsx
import { Image } from "@/components/ui/image";

export default function WapImage({
  source,
  alt,
  size = "full",
  resizeMode = "cover",
  rounded,
  className,
}: WapImageProps) {
  if (!source?.src) return null;
  return (
    <Image
      source={{ uri: source.src }}
      alt={alt || source.alt || ""}
      size={size !== "none" ? size : undefined}
      className={`object-${resizeMode} ${rounded ? `rounded-[${rounded}px]` : ""} ${className || ""}`}
    />
  );
}
```

**Mobile:** Same but wrap `source.src` with `getCDNImage()`:

```tsx
import { getCDNImage } from "../../utils/path";
const uri = source?.src ? getCDNImage(source.src) : undefined;
```

---

### `video` — Video Player

**Admin Props:** `source?: { src?: string }`

**Web:** `<video src={source?.src} controls className="w-full" />`

**Mobile:** Use `expo-video` or `expo-av`:

```tsx
import { Video, ResizeMode } from "expo-av";
// <Video source={{ uri: source?.src }} useNativeControls resizeMode={ResizeMode.CONTAIN} />
```

---

### `iframe` — Embedded Iframe (Web only)

**Admin Props:** `children?: string` (URL)

**Web:** `<iframe src={children} className="w-full h-64 border-0" />`

**Mobile:** Not supported — render null or a `<Link>` to open in browser.

---

## Interactive Components

### `button` — Button

**Admin Props (from `BUTTON_PROPS`):**

```ts
interface ButtonProps {
  children?: string;
  anchor?: { href?: string; target?: string; title?: string };
  action?: "primary" | "secondary" | "positive" | "negative" | "default";
  variant?: "solid" | "outline" | "link";
  size?: "xs" | "sm" | "md" | "lg" | "xl";
  className?: string;
}
```

**Web:**

```tsx
"use client";
import { Button, ButtonText } from "@/components/ui/button";
import Link from "next/link";

export default function WapButton({
  children,
  anchor,
  action = "primary",
  variant = "solid",
  size = "md",
  className,
}: ButtonProps) {
  const label = children || anchor?.title || "";
  if (anchor?.href) {
    return (
      <Button
        action={action}
        variant={variant}
        size={size}
        className={className}
        asChild
      >
        <Link href={anchor.href} target={anchor.target || "_self"}>
          <ButtonText>{label}</ButtonText>
        </Link>
      </Button>
    );
  }
  return (
    <Button action={action} variant={variant} size={size} className={className}>
      <ButtonText>{label}</ButtonText>
    </Button>
  );
}
```

**Mobile:**

```tsx
import { Button, ButtonText } from "@/components/ui/button";
import { Linking } from "react-native";

export default function WapButton({
  children,
  anchor,
  action = "primary",
  variant = "solid",
  size = "md",
  className,
}: ButtonProps) {
  const label = safeText(children || anchor?.title);
  return (
    <Button
      action={action}
      variant={variant}
      size={size}
      className={className}
      onPress={() => anchor?.href && Linking.openURL(anchor.href)}
    >
      <ButtonText>{label}</ButtonText>
    </Button>
  );
}
```

---

### `link` — Link (droppable)

**Admin Props:**

```ts
interface LinkProps {
  source?: { href?: string; target?: string; title?: string };
  className?: string;
  children?: React.ReactNode;
}
```

**Web:**

```tsx
import { Link, LinkText } from "@/components/ui/link";

export default function WapLink({ source, className, children }: LinkProps) {
  if (!source?.href) return <>{children}</>;
  return (
    <Link
      href={source.href}
      isExternal={source.target === "_blank"}
      className={className}
    >
      {children || <LinkText>{source.title}</LinkText>}
    </Link>
  );
}
```

**Mobile:** Same pattern but use `Linking.openURL` in `onPress`.

---

## Display Components

### `card` — Card

**Admin Props (from `CARD_PROPS`):**

```ts
interface CardProps {
  title?: string;
  subtitle?: string;
  description?: string;
  image?: { src?: string; alt?: string };
  anchor?: { href?: string; target?: string; title?: string };
  size?: "sm" | "md" | "lg";
  variant?: "ghost" | "outline" | "filled" | "elevated";
  className?: string;
}
```

**Web:**

```tsx
import { Card } from "@/components/ui/card";
import { Heading } from "@/components/ui/heading";
import { Text } from "@/components/ui/text";
import { Image } from "@/components/ui/image";
import { Link, LinkText } from "@/components/ui/link";

export default function WapCard({
  title,
  subtitle,
  description,
  image,
  anchor,
  size = "md",
  variant = "outline",
  className,
}: CardProps) {
  return (
    <Card size={size} variant={variant} className={className}>
      {image?.src && (
        <Image
          source={{ uri: image.src }}
          alt={image.alt || title || ""}
          className="w-full h-48 object-cover"
        />
      )}
      {title && <Heading size="md">{title}</Heading>}
      {subtitle && (
        <Text size="sm" className="text-typography-500">
          {subtitle}
        </Text>
      )}
      {description && <Text size="sm">{description}</Text>}
      {anchor?.href && (
        <Link href={anchor.href} isExternal={anchor.target === "_blank"}>
          <LinkText>{anchor.title || "Read more"}</LinkText>
        </Link>
      )}
    </Card>
  );
}
```

**Mobile:**

```tsx
import { Card } from "@/components/ui/card";
import { Pressable } from "@/components/ui/pressable";
import { Linking } from "react-native";
import { getCDNImage } from "../../utils/path";

export default function WapCard({
  title,
  subtitle,
  description,
  image,
  anchor,
  size = "md",
  variant = "outline",
  className,
}: CardProps) {
  const imgSrc = image?.src ? getCDNImage(image.src) : undefined;
  return (
    <Pressable onPress={() => anchor?.href && Linking.openURL(anchor.href)}>
      <Card size={size} variant={variant} className={className}>
        {imgSrc && (
          <Image
            source={{ uri: imgSrc }}
            alt={image?.alt || ""}
            className="w-full h-48"
          />
        )}
        {title && <Heading size="md">{safeText(title)}</Heading>}
        {subtitle && (
          <Text size="sm" className="text-typography-500">
            {safeText(subtitle)}
          </Text>
        )}
        {description && <Text size="sm">{safeText(description)}</Text>}
      </Card>
    </Pressable>
  );
}
```

---

### `card-list` — Card Grid

**Admin Props:** `cards?: CardProps[]` + column group breakpoints (same as `ColumnProps`)

**Web:**

```tsx
import { Box } from "@/components/ui/box";
import WapCard from "../card";

export default function CardList({
  cards = [],
  className,
}: {
  cards?: any[];
  className?: string;
}) {
  return (
    <Box className={`flex flex-wrap gap-4 ${className || ""}`}>
      {cards.map((card, i) => (
        <WapCard key={i} {...card} />
      ))}
    </Box>
  );
}
```

**Mobile:** Use `FlatList` with `WapCard`.

---

### `avatar` — Avatar

**Admin Props (from `AVATAR_PROPS`):**

```ts
interface AvatarProps {
  source?: { src?: string };
  name?: string;
  size?: "xs" | "sm" | "md" | "lg" | "xl" | "2xl";
  showBadge?: boolean;
  className?: string;
}
```

**Web & Mobile:**

```tsx
import {
  Avatar,
  AvatarImage,
  AvatarFallbackText,
  AvatarBadge,
} from "@/components/ui/avatar";

export default function WapAvatar({
  source,
  name,
  size = "md",
  showBadge,
  className,
}: AvatarProps) {
  return (
    <Avatar size={size} className={className}>
      {source?.src ? (
        <AvatarImage source={{ uri: source.src }} />
      ) : (
        <AvatarFallbackText>{safeText(name)}</AvatarFallbackText>
      )}
      {showBadge && <AvatarBadge />}
    </Avatar>
  );
}
```

---

### `badge` — Badge

**Admin Props (from `BADGE_PROPS`):**

```ts
interface BadgeProps {
  text?: string;
  action?: "info" | "success" | "warning" | "error" | "muted";
  variant?: "solid" | "outline";
  size?: "sm" | "md" | "lg";
  className?: string;
}
```

**Web & Mobile:**

```tsx
import { Badge, BadgeText } from "@/components/ui/badge";

export default function WapBadge({
  text,
  action = "info",
  variant = "solid",
  size = "md",
  className,
}: BadgeProps) {
  return (
    <Badge action={action} variant={variant} size={size} className={className}>
      <BadgeText>{safeText(text)}</BadgeText>
    </Badge>
  );
}
```

---

### `divider` — Divider

**Admin Props (from `DIVIDER_PROPS`):** `orientation?: 'horizontal' | 'vertical'`

```tsx
import { Divider } from "@/components/ui/divider";
export default function WapDivider({
  orientation = "horizontal",
  className,
}: any) {
  return <Divider orientation={orientation} className={className} />;
}
```

---

## Feedback Components

### `spinner` — Loading Spinner

**Admin Props (from `SPINNER_PROPS`):** `size?: 'small' | 'large'`, `color?: string`

```tsx
import { Spinner } from "@/components/ui/spinner";
export default function WapSpinner({ size = "small", className }: any) {
  return <Spinner size={size} className={className} />;
}
```

---

### `alert` — Alert Message

**Admin Props (from `ALERT_PROPS`):**

```ts
interface AlertProps {
  text?: string;
  action?: "info" | "success" | "warning" | "error" | "muted";
  variant?: "solid" | "outline";
  showIcon?: boolean;
  className?: string;
}
```

**Web & Mobile:**

```tsx
import { Alert, AlertIcon, AlertText } from "@/components/ui/alert";
import {
  InfoIcon,
  CheckCircleIcon,
  AlertTriangleIcon,
  AlertCircleIcon,
} from "@/components/ui/icon";

const ACTION_ICONS: Record<string, any> = {
  info: InfoIcon,
  success: CheckCircleIcon,
  warning: AlertTriangleIcon,
  error: AlertCircleIcon,
  muted: InfoIcon,
};

export default function WapAlert({
  text,
  action = "info",
  variant = "solid",
  showIcon = true,
  className,
}: AlertProps) {
  return (
    <Alert action={action} variant={variant} className={className}>
      {showIcon && <AlertIcon as={ACTION_ICONS[action]} />}
      <AlertText>{safeText(text)}</AlertText>
    </Alert>
  );
}
```

---

### `progress` — Progress Bar

**Admin Props (from `PROGRESS_PROPS`):**

```ts
interface ProgressProps {
  value?: number;
  size?: "xs" | "sm" | "md" | "lg" | "xl";
  showLabel?: boolean;
  label?: string;
  className?: string;
}
```

**Web & Mobile:**

```tsx
import { Progress, ProgressFilledTrack } from "@/components/ui/progress";
import { Text } from "@/components/ui/text";
import { VStack } from "@/components/ui/vstack";

export default function WapProgress({
  value = 0,
  size = "md",
  showLabel,
  label,
  className,
}: ProgressProps) {
  return (
    <VStack className={className}>
      {showLabel && <Text size="sm">{label || `${value}%`}</Text>}
      <Progress value={value} size={size}>
        <ProgressFilledTrack />
      </Progress>
    </VStack>
  );
}
```

---

## Form Components

> All form components use `FormControl` wrapper for `label`, `helperText`, and `isDisabled` / `isRequired` state.

### `input` — Text Input

**Admin Props (from `INPUT_PROPS`):**

```ts
interface InputProps {
  label?: string;
  placeholder?: string;
  type?: "text" | "password";
  size?: "sm" | "md" | "lg" | "xl";
  variant?: "outline" | "underlined" | "rounded";
  helperText?: string;
  isDisabled?: boolean;
  isRequired?: boolean;
  className?: string;
}
```

**Web & Mobile:**

```tsx
import {
  FormControl,
  FormControlLabel,
  FormControlLabelText,
  FormControlHelper,
  FormControlHelperText,
} from "@/components/ui/form-control";
import { Input, InputField } from "@/components/ui/input";

export default function WapInput({
  label,
  placeholder,
  type = "text",
  size = "md",
  variant = "outline",
  helperText,
  isDisabled,
  isRequired,
  className,
}: InputProps) {
  return (
    <FormControl
      isDisabled={isDisabled}
      isRequired={isRequired}
      className={className}
    >
      {label && (
        <FormControlLabel>
          <FormControlLabelText>{label}</FormControlLabelText>
        </FormControlLabel>
      )}
      <Input size={size} variant={variant}>
        <InputField placeholder={placeholder} type={type} />
      </Input>
      {helperText && (
        <FormControlHelper>
          <FormControlHelperText>{helperText}</FormControlHelperText>
        </FormControlHelper>
      )}
    </FormControl>
  );
}
```

---

### `select` — Select Dropdown

**Admin Props (from `SELECT_PROPS`):**

```ts
interface SelectOption {
  label: string;
  value: string;
}
interface SelectProps {
  label?: string;
  placeholder?: string;
  options?: SelectOption[];
  size?: "sm" | "md" | "lg" | "xl";
  variant?: "outline" | "underlined" | "rounded";
  isDisabled?: boolean;
  className?: string;
}
```

**Web & Mobile:**

```tsx
import {
  Select,
  SelectTrigger,
  SelectInput,
  SelectPortal,
  SelectContent,
  SelectItem,
  SelectDragIndicatorWrapper,
  SelectDragIndicator,
} from "@/components/ui/select";
import {
  FormControl,
  FormControlLabel,
  FormControlLabelText,
} from "@/components/ui/form-control";

export default function WapSelect({
  label,
  placeholder,
  options = [],
  size = "md",
  variant = "outline",
  isDisabled,
  className,
}: SelectProps) {
  return (
    <FormControl isDisabled={isDisabled} className={className}>
      {label && (
        <FormControlLabel>
          <FormControlLabelText>{label}</FormControlLabelText>
        </FormControlLabel>
      )}
      <Select>
        <SelectTrigger variant={variant} size={size}>
          <SelectInput placeholder={placeholder} />
        </SelectTrigger>
        <SelectPortal>
          <SelectContent>
            <SelectDragIndicatorWrapper>
              <SelectDragIndicator />
            </SelectDragIndicatorWrapper>
            {options.map((opt) => (
              <SelectItem key={opt.value} label={opt.label} value={opt.value} />
            ))}
          </SelectContent>
        </SelectPortal>
      </Select>
    </FormControl>
  );
}
```

---

### `switch` — Toggle Switch

**Admin Props (from `SWITCH_PROPS`):**

```ts
interface SwitchProps {
  label?: string;
  size?: "sm" | "md" | "lg";
  isDisabled?: boolean;
  className?: string;
}
```

```tsx
import { Switch } from "@/components/ui/switch";
import { HStack } from "@/components/ui/hstack";
import { Text } from "@/components/ui/text";

export default function WapSwitch({
  label,
  size = "md",
  isDisabled,
  className,
}: SwitchProps) {
  return (
    <HStack space="sm" className={`items-center ${className || ""}`}>
      {label && <Text size="sm">{safeText(label)}</Text>}
      <Switch size={size} isDisabled={isDisabled} />
    </HStack>
  );
}
```

---

### `checkbox` — Checkbox

**Admin Props (from `CHECKBOX_PROPS`):**

```ts
interface CheckboxOption {
  label: string;
  value: string;
}
interface CheckboxProps {
  label?: string;
  options?: CheckboxOption[];
  size?: "sm" | "md" | "lg";
  isDisabled?: boolean;
  className?: string;
}
```

```tsx
import {
  Checkbox,
  CheckboxIndicator,
  CheckboxLabel,
  CheckboxIcon,
} from "@/components/ui/checkbox";
import { CheckIcon } from "@/components/ui/icon";
import { VStack } from "@/components/ui/vstack";

export default function WapCheckbox({
  label,
  options,
  size = "md",
  isDisabled,
  className,
}: CheckboxProps) {
  if (options?.length) {
    return (
      <VStack space="sm" className={className}>
        {options.map((opt) => (
          <Checkbox
            key={opt.value}
            value={opt.value}
            size={size}
            isDisabled={isDisabled}
          >
            <CheckboxIndicator>
              <CheckboxIcon as={CheckIcon} />
            </CheckboxIndicator>
            <CheckboxLabel>{opt.label}</CheckboxLabel>
          </Checkbox>
        ))}
      </VStack>
    );
  }
  return (
    <Checkbox
      value="default"
      size={size}
      isDisabled={isDisabled}
      className={className}
    >
      <CheckboxIndicator>
        <CheckboxIcon as={CheckIcon} />
      </CheckboxIndicator>
      {label && <CheckboxLabel>{safeText(label)}</CheckboxLabel>}
    </Checkbox>
  );
}
```

---

### `radio` — Radio Group

**Admin Props (from `RADIO_PROPS`):**

```ts
interface RadioOption {
  label: string;
  value: string;
}
interface RadioProps {
  options?: RadioOption[];
  size?: "sm" | "md" | "lg";
  isDisabled?: boolean;
  className?: string;
}
```

```tsx
import {
  RadioGroup,
  Radio,
  RadioIndicator,
  RadioLabel,
  RadioIcon,
} from "@/components/ui/radio";
import { CircleIcon } from "@/components/ui/icon";
import { VStack } from "@/components/ui/vstack";

export default function WapRadio({
  options = [],
  size = "md",
  isDisabled,
  className,
}: RadioProps) {
  return (
    <RadioGroup className={className}>
      <VStack space="sm">
        {options.map((opt) => (
          <Radio
            key={opt.value}
            value={opt.value}
            size={size}
            isDisabled={isDisabled}
          >
            <RadioIndicator>
              <RadioIcon as={CircleIcon} />
            </RadioIndicator>
            <RadioLabel>{opt.label}</RadioLabel>
          </Radio>
        ))}
      </VStack>
    </RadioGroup>
  );
}
```

---

### `textarea` — Multi-line Input

**Admin Props (from `TEXTAREA_PROPS`):**

```ts
interface TextareaProps {
  label?: string;
  placeholder?: string;
  size?: "sm" | "md" | "lg" | "xl";
  numberOfLines?: number;
  helperText?: string;
  isDisabled?: boolean;
  isReadOnly?: boolean;
  className?: string;
}
```

```tsx
import {
  FormControl,
  FormControlLabel,
  FormControlLabelText,
  FormControlHelper,
  FormControlHelperText,
} from "@/components/ui/form-control";
import { Textarea, TextareaInput } from "@/components/ui/textarea";

export default function WapTextarea({
  label,
  placeholder,
  size = "md",
  numberOfLines,
  helperText,
  isDisabled,
  isReadOnly,
  className,
}: TextareaProps) {
  return (
    <FormControl
      isDisabled={isDisabled}
      isReadOnly={isReadOnly}
      className={className}
    >
      {label && (
        <FormControlLabel>
          <FormControlLabelText>{label}</FormControlLabelText>
        </FormControlLabel>
      )}
      <Textarea size={size}>
        <TextareaInput
          placeholder={placeholder}
          numberOfLines={numberOfLines}
        />
      </Textarea>
      {helperText && (
        <FormControlHelper>
          <FormControlHelperText>{helperText}</FormControlHelperText>
        </FormControlHelper>
      )}
    </FormControl>
  );
}
```

---

### `slider` — Slider

**Admin Props (from `SLIDER_PROPS`):**

```ts
interface SliderProps {
  value?: number;
  minValue?: number;
  maxValue?: number;
  step?: number;
  size?: "sm" | "md" | "lg";
  showLabel?: boolean;
  isDisabled?: boolean;
  className?: string;
}
```

```tsx
import {
  Slider,
  SliderTrack,
  SliderFilledTrack,
  SliderThumb,
} from "@/components/ui/slider";
import { Text } from "@/components/ui/text";
import { VStack } from "@/components/ui/vstack";

export default function WapSlider({
  value = 0,
  minValue = 0,
  maxValue = 100,
  step = 1,
  size = "md",
  showLabel,
  isDisabled,
  className,
}: SliderProps) {
  return (
    <VStack className={className}>
      {showLabel && <Text size="sm">{value}</Text>}
      <Slider
        defaultValue={value}
        minValue={minValue}
        maxValue={maxValue}
        step={step}
        size={size}
        isDisabled={isDisabled}
      >
        <SliderTrack>
          <SliderFilledTrack />
        </SliderTrack>
        <SliderThumb />
      </Slider>
    </VStack>
  );
}
```

---

## `array-repeater` — Dynamic List

Uses the SDK `ArrayRepeater` component directly — no gluestack wrapping needed.

```tsx
import { ArrayRepeater } from "@appaflytech/wappa-client/core/components";
// In registry: case "array-repeater": return ArrayRepeater as any;
```

Props via `refs.repeater`:

```ts
{
  repeater: {
    type: 'query' | 'raw' | 'showcase',
    ref: string,
    mappings: Array<{ name: string; path: string; type: string }>
  }
}
```

Child components bind to abstract names via `refs`:

```json
{
  "name": "heading",
  "refs": { "children": { "type": "repeater", "ref": "title" } }
}
```
