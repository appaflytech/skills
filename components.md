# Wappa Component Schema Contracts

> **Load this file when implementing any component.**
> Props contracts are derived from the Wappa admin schema (`elements.ts` + `constants.ts`).
> The contract defines what props the admin can configure. Your implementation must accept all of them.
> Framework is your choice — examples use gluestack-ui v4 (default).

---

## Shared Types

```ts
// Image prop — resolved by refs (do not construct manually)
type ImageProps = {
  src: string
  alt?: string
  width?: number
  height?: number
}

// Link prop — resolved by refs
type LinkProps = {
  href: string
  target?: '_blank' | '_self'
  rel?: string
}

// Column breakpoint config (used by row/column)
type BreakpointConfig = {
  direction?: 'row' | 'column' | 'row-reverse' | 'column-reverse'
  align?: 'start' | 'center' | 'end' | 'stretch' | 'baseline'
  justify?: 'start' | 'center' | 'end' | 'between' | 'around' | 'evenly'
  wrap?: 'wrap' | 'nowrap' | 'wrap-reverse'
  gutter?: string
}

type ColumnBreakpoint = {
  size?: 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12
  align?: 'start' | 'center' | 'end'
  offset?: 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12
  order?: 'first' | 'last'
}

type SpaceToken = 'none' | 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl' | '3xl' | '4xl'
type SizeToken  = 'xs' | 'sm' | 'md' | 'lg' | 'xl'
```

---

## GROUP: layout

### `container` — isMobile: false (web only)

```ts
interface ContainerProps {
  size?: 'fluid' | 'extended' | 'wide' | 'medium' | 'narrow'
  className?: string
  children?: React.ReactNode
}
```

**gluestack-ui v4 default:**
```tsx
export default function Container({ size = 'wide', className, children }: ContainerProps) {
  const sizeMap = { fluid: 'w-full', extended: 'max-w-screen-2xl', wide: 'max-w-screen-xl', medium: 'max-w-screen-lg', narrow: 'max-w-screen-md' }
  return <div className={`mx-auto px-4 ${sizeMap[size]} ${className ?? ''}`}>{children}</div>
}
```

---

### `array-repeater` — isMobile: true

Built-in SDK component — import and use directly in registry:

```ts
import { ArrayRepeater } from "@appaflytech/wappa-client/core/components"
// "array-repeater": ArrayRepeater
```

---

### `box` — isMobile: true

```ts
interface BoxProps {
  className?: string
  children?: React.ReactNode
}
```

gluestack-ui v4: `import { Box } from "@/components/ui/box"`

---

### `center` — isMobile: true

```ts
interface CenterProps {
  className?: string
  children?: React.ReactNode
}
```

gluestack-ui v4: `import { Center } from "@/components/ui/center"`

---

### `hstack` — isMobile: true

```ts
interface HStackProps {
  space?: SpaceToken
  reversed?: boolean
  className?: string
  children?: React.ReactNode
}
```

gluestack-ui v4: `import { HStack } from "@/components/ui/hstack"` — pass `space`, `reversed` directly.

---

### `vstack` — isMobile: true

```ts
interface VStackProps {
  space?: SpaceToken
  reversed?: boolean
  className?: string
  children?: React.ReactNode
}
```

gluestack-ui v4: `import { VStack } from "@/components/ui/vstack"` — pass `space`, `reversed` directly.

---

### `grid` — isMobile: true

```ts
interface GridProps {
  numColumns?: number
  gap?: '0' | 'xs' | 'sm' | 'md' | 'lg' | 'xl'
  className?: string
  children?: React.ReactNode
}
```

gluestack-ui v4: `import { Grid } from "@/components/ui/grid"` — pass `numColumns`, `gap` directly.

---

### `pressable` — isMobile: true

```ts
interface PressableProps {
  className?: string
  children?: React.ReactNode
  onPress?: () => void
}
```

gluestack-ui v4: `import { Pressable } from "@/components/ui/pressable"` — pass `onPress` directly.

---

### `row` — isMobile: false (web grid row)

```ts
interface RowProps {
  xs?: BreakpointConfig
  sm?: BreakpointConfig
  md?: BreakpointConfig
  lg?: BreakpointConfig
  xl?: BreakpointConfig
  className?: string
  children?: React.ReactNode
}
```

Map breakpoint configs to CSS flex classes. Typically wraps columns.

---

### `column` — isMobile: false (web grid column)

```ts
interface ColumnProps {
  xs?: ColumnBreakpoint
  sm?: ColumnBreakpoint
  md?: ColumnBreakpoint
  lg?: ColumnBreakpoint
  xl?: ColumnBreakpoint
  className?: string
  children?: React.ReactNode
}
```

Use `col-{size}`, `offset-{offset}`, breakpoint prefixes for responsive layout.

---

### `section` — isMobile: false

```ts
interface SectionProps {
  title?: string
  description?: string
  anchor?: LinkProps
  container?: { size?: 'fluid' | 'extended' | 'wide' | 'medium' | 'narrow' }
  className?: string
  children?: React.ReactNode
}
```

---

### `view` — isMobile: true

> **Special component**: The render system uses its `id` to look up content in `page.views[id]`.
> The render function handles `view` natively — **no component implementation needed**.

---

---

## GROUP: typography

### `heading` — isMobile: true

```ts
interface HeadingProps {
  children: string
  nodeType?: 'h1' | 'h2' | 'h3' | 'h4' | 'h5' | 'h6'
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl' | '3xl' | '4xl' | '5xl'
  bold?: boolean
  isTruncated?: boolean
  numberOfLines?: number
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Heading } from "@/components/ui/heading"
export default function WapHeading({ children, nodeType = 'h2', size = 'xl', bold, isTruncated, className }: HeadingProps) {
  return (
    <Heading as={nodeType} size={size} bold={bold} isTruncated={isTruncated} className={className}>
      {children}
    </Heading>
  )
}
```

---

### `paragraph` — isMobile: true

```ts
interface ParagraphProps {
  children: string
  size?: '2xs' | 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl' | '3xl' | '4xl' | '5xl' | '6xl'
  bold?: boolean
  italic?: boolean
  isTruncated?: boolean
  numberOfLines?: number
  className?: string
}
```

**gluestack-ui v4 default:** Use `Text` component — `import { Text } from "@/components/ui/text"`

---

### `html` — isMobile: false (web only)

```ts
interface HtmlProps {
  content: string   // Raw HTML string (rich text editor output)
  className?: string
}
```

```tsx
export default function WapHtml({ content, className }: HtmlProps) {
  return <div className={`prose max-w-none ${className ?? ''}`} dangerouslySetInnerHTML={{ __html: content }} />
}
```

---

### `icon` — isMobile: true

```ts
interface WapIconProps {
  as: string           // Lucide icon name (e.g. "ArrowRight", "Star")
  size?: '2xs' | 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl'
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Icon } from "@/components/ui/icon"
import * as LucideIcons from "lucide-react-native"  // or lucide-react on web

export default function WapIcon({ as: iconName, size = 'md', className }: WapIconProps) {
  const LucideIcon = (LucideIcons as any)[iconName]
  if (!LucideIcon) return null
  return <Icon as={LucideIcon} size={size} className={className} />
}
```

---

---

## GROUP: media

### `image` — isMobile: true

```ts
interface WapImageProps {
  source: ImageProps      // resolved from refs (src, alt, width, height)
  alt?: string
  size?: '2xs' | 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl' | 'full' | 'none'
  resizeMode?: 'cover' | 'contain' | 'stretch' | 'center'
  rounded?: 'none' | 'sm' | 'md' | 'lg' | 'xl' | '2xl' | '3xl' | 'full'
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Image } from "@/components/ui/image"
import { path } from "@appaflytech/wappa-client/core/utils"

export default function WapImage({ source, alt, size = 'full', resizeMode = 'cover', rounded = 'none', className }: WapImageProps) {
  if (!source?.src) return null
  return (
    <Image
      source={{ uri: path.cdn(source.src) }}
      alt={alt || source.alt || ''}
      size={size}
      className={`object-${resizeMode} rounded-${rounded} ${className ?? ''}`}
    />
  )
}
```

---

### `video` — isMobile: true

```ts
interface VideoProps {
  source: { src: string }
  autoPlay?: boolean
  muted?: boolean
  loop?: boolean
  controls?: boolean
  className?: string
}
```

**Web:** use `<video>` element. **Mobile:** use `expo-av` Video component.

---

### `iframe` — isMobile: false (web only)

```ts
interface IframeProps {
  children: string    // URL to embed
  title?: string
  className?: string
}
```

```tsx
export default function WapIframe({ children: src, title = '', className }: IframeProps) {
  return <iframe src={src} title={title} className={`w-full ${className ?? ''}`} allowFullScreen />
}
```

---

---

## GROUP: interactive

### `button` — isMobile: true

```ts
interface WapButtonProps {
  children: string
  anchor?: LinkProps
  action?: 'primary' | 'secondary' | 'positive' | 'negative' | 'default'
  variant?: 'solid' | 'outline' | 'link'
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl'
  isDisabled?: boolean
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Button, ButtonText } from "@/components/ui/button"
import Link from "next/link"

export default function WapButton({ children, anchor, action = 'primary', variant = 'solid', size = 'md', isDisabled, className }: WapButtonProps) {
  const btn = (
    <Button action={action} variant={variant} size={size} isDisabled={isDisabled} className={className}>
      <ButtonText>{children}</ButtonText>
    </Button>
  )
  if (!anchor?.href) return btn
  return <Link href={anchor.href} target={anchor.target}>{btn}</Link>
}
```

---

### `link` — isMobile: true

```ts
interface WapLinkProps {
  source: LinkProps
  isExternal?: boolean
  className?: string
  children?: React.ReactNode
}
```

**gluestack-ui v4 default:**
```tsx
import { Link, LinkText } from "@/components/ui/link"
export default function WapLink({ source, isExternal, className, children }: WapLinkProps) {
  return (
    <Link href={source?.href ?? '#'} isExternal={isExternal} className={className}>
      <LinkText>{children}</LinkText>
    </Link>
  )
}
```

---

### `fab` — isMobile: true

```ts
interface FabProps {
  label?: string
  iconName?: string    // Lucide icon name
  size?: 'sm' | 'md' | 'lg'
  placement?: 'bottom left' | 'bottom right' | 'top left' | 'top right'
  isDisabled?: boolean
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Fab, FabLabel, FabIcon } from "@/components/ui/fab"
import * as LucideIcons from "lucide-react-native"

export default function WapFab({ label, iconName, size = 'md', placement = 'bottom right', isDisabled, className }: FabProps) {
  const LucideIcon = iconName ? (LucideIcons as any)[iconName] : null
  return (
    <Fab size={size} placement={placement} isDisabled={isDisabled} className={className}>
      {LucideIcon && <FabIcon as={LucideIcon} />}
      {label && <FabLabel>{label}</FabLabel>}
    </Fab>
  )
}
```

---

---

## GROUP: display

### `card` — isMobile: true

```ts
interface CardProps {
  title?: string
  subtitle?: string
  description?: string
  image?: ImageProps
  anchor?: LinkProps
  size?: 'sm' | 'md' | 'lg'
  variant?: 'ghost' | 'outline' | 'filled' | 'elevated'
  className?: string
  children?: React.ReactNode
}
```

**Note:** gluestack-ui v4 does not have a first-class `Card` component. Build using `Box`, `VStack`, `Heading`, `Text`, `Image` from gluestack-ui v4.

---

### `card-list` — isMobile: true

```ts
interface CardListProps {
  cards?: CardProps[]
  xs?: { size?: number }
  sm?: { size?: number }
  md?: { size?: number }
  lg?: { size?: number }
  xl?: { size?: number }
  className?: string
}
```

Render a grid of `Card` components from the `cards` array. Use breakpoint `size` to determine column count.

---

### `avatar` — isMobile: true

```ts
interface AvatarProps {
  source?: ImageProps
  name?: string
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl'
  showBadge?: boolean
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Avatar, AvatarImage, AvatarFallbackText, AvatarBadge } from "@/components/ui/avatar"

export default function WapAvatar({ source, name = '', size = 'md', showBadge, className }: AvatarProps) {
  return (
    <Avatar size={size} className={className}>
      {source?.src ? <AvatarImage source={{ uri: source.src }} /> : null}
      <AvatarFallbackText>{name}</AvatarFallbackText>
      {showBadge && <AvatarBadge />}
    </Avatar>
  )
}
```

---

### `badge` — isMobile: true

```ts
interface WapBadgeProps {
  text: string
  action?: 'info' | 'success' | 'warning' | 'error' | 'muted'
  variant?: 'solid' | 'outline'
  size?: 'sm' | 'md' | 'lg'
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Badge, BadgeText } from "@/components/ui/badge"
export default function WapBadge({ text, action = 'info', variant = 'solid', size = 'md', className }: WapBadgeProps) {
  return <Badge action={action} variant={variant} size={size} className={className}><BadgeText>{text}</BadgeText></Badge>
}
```

---

### `divider` — isMobile: true

```ts
interface WapDividerProps {
  orientation?: 'horizontal' | 'vertical'
  className?: string
}
```

gluestack-ui v4: `import { Divider } from "@/components/ui/divider"` — pass `orientation` directly.

---

### `table` — isMobile: false (web only)

```ts
interface TableColumnDef {
  header: string
  key: string
}

interface WapTableProps {
  dataKey: string                    // query key — data is in refs.queries[dataKey] or props[dataKey]
  columns: TableColumnDef[]
  showCaption?: boolean
  caption?: string
  className?: string
  [key: string]: any                 // dynamic data from refs
}
```

**Implementation note:** Access row data via `props[dataKey]` or `refs.queries[dataKey]` (resolved into spread props).

---

### `skeleton` — isMobile: true

```ts
interface SkeletonProps {
  variant?: 'rounded' | 'sharp' | 'circular'
  speed?: number
  isText?: boolean
  lines?: number
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Skeleton, SkeletonText } from "@/components/ui/skeleton"
export default function WapSkeleton({ variant = 'rounded', isText, lines = 3, className }: SkeletonProps) {
  if (isText) return <SkeletonText lines={lines} className={className} />
  return <Skeleton variant={variant} className={`h-24 ${className ?? ''}`} />
}
```

---

---

## GROUP: feedback

### `spinner` — isMobile: true

```ts
interface SpinnerProps {
  size?: 'small' | 'large'
  color?: string
  className?: string
}
```

gluestack-ui v4: `import { Spinner } from "@/components/ui/spinner"` — pass all props directly.

---

### `alert` — isMobile: true

```ts
interface WapAlertProps {
  text: string
  action?: 'info' | 'success' | 'warning' | 'error' | 'muted'
  variant?: 'solid' | 'outline'
  showIcon?: boolean
  iconName?: string
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Alert, AlertText, AlertIcon } from "@/components/ui/alert"
import * as LucideIcons from "lucide-react-native"

export default function WapAlert({ text, action = 'info', variant = 'solid', showIcon, iconName, className }: WapAlertProps) {
  const LucideIcon = iconName ? (LucideIcons as any)[iconName] : null
  return (
    <Alert action={action} variant={variant} className={className}>
      {showIcon && LucideIcon && <AlertIcon as={LucideIcon} />}
      <AlertText>{text}</AlertText>
    </Alert>
  )
}
```

---

### `progress` — isMobile: true

```ts
interface ProgressProps {
  value: number        // 0–100
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl'
  showLabel?: boolean
  label?: string
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Progress, ProgressFilledTrack } from "@/components/ui/progress"
export default function WapProgress({ value, size = 'md', showLabel, label, className }: ProgressProps) {
  return (
    <div>
      {showLabel && <span className="text-sm">{label ?? `${value}%`}</span>}
      <Progress value={value} size={size} className={className}>
        <ProgressFilledTrack />
      </Progress>
    </div>
  )
}
```

---

### `toast` — isMobile: true

```ts
interface WapToastProps {
  title: string
  description?: string
  action?: 'info' | 'success' | 'warning' | 'error' | 'muted'
  variant?: 'solid' | 'outline'
  placement?: 'top' | 'top left' | 'top right' | 'bottom' | 'bottom left' | 'bottom right'
  duration?: number
  className?: string
}
```

**Implementation note:** Toast is triggered programmatically. Use `useToast()` hook from gluestack-ui v4. In the page builder, render a preview toast or a trigger button.

---

---

## GROUP: disclosure

### `accordion` — isMobile: true

```ts
interface AccordionItem {
  title: string
  content: string
  isDisabled?: boolean
}

interface AccordionProps {
  type?: 'single' | 'multiple'
  variant?: 'filled' | 'unfilled'
  items: AccordionItem[]
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import {
  Accordion, AccordionItem, AccordionHeader, AccordionTrigger,
  AccordionTitleText, AccordionContent, AccordionContentText,
} from "@/components/ui/accordion"

export default function WapAccordion({ type = 'single', variant = 'filled', items, className }: AccordionProps) {
  return (
    <Accordion type={type} variant={variant} className={className}>
      {items.map((item, i) => (
        <AccordionItem key={i} value={`item-${i}`} isDisabled={item.isDisabled}>
          <AccordionHeader>
            <AccordionTrigger>
              <AccordionTitleText>{item.title}</AccordionTitleText>
            </AccordionTrigger>
          </AccordionHeader>
          <AccordionContent>
            <AccordionContentText>{item.content}</AccordionContentText>
          </AccordionContent>
        </AccordionItem>
      ))}
    </Accordion>
  )
}
```

---

### `tabs` — isMobile: true

```ts
interface TabItem {
  label: string
  value: string
  iconName?: string
  content: string
}

interface TabsProps {
  variant?: 'underlined' | 'filled' | 'enclosed'
  orientation?: 'horizontal' | 'vertical'
  size?: 'sm' | 'md' | 'lg'
  items: TabItem[]
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Tabs, TabsList, TabsTrigger, TabsContent, TabsTriggerText } from "@/components/ui/tabs"

export default function WapTabs({ items, orientation = 'horizontal', className }: TabsProps) {
  return (
    <Tabs defaultValue={items[0]?.value} orientation={orientation} className={className}>
      <TabsList>
        {items.map((item) => (
          <TabsTrigger key={item.value} value={item.value}>
            <TabsTriggerText>{item.label}</TabsTriggerText>
          </TabsTrigger>
        ))}
      </TabsList>
      {items.map((item) => (
        <TabsContent key={item.value} value={item.value}>{item.content}</TabsContent>
      ))}
    </Tabs>
  )
}
```

---

---

## GROUP: overlay

> Overlay components require open/close state. In the page builder they render static preview.
> In production, the trigger is typically a Button that opens the overlay.

### `modal` — isMobile: true

```ts
interface ModalProps {
  title?: string
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'full'
  showCloseButton?: boolean
  className?: string
  children?: React.ReactNode
}
```

---

### `drawer` — isMobile: true

```ts
interface DrawerProps {
  title?: string
  placement?: 'left' | 'right' | 'top' | 'bottom'
  showCloseButton?: boolean
  className?: string
  children?: React.ReactNode
}
```

---

### `actionsheet` — isMobile: true

```ts
interface ActionsheetItem {
  text: string
  iconName?: string
  isDisabled?: boolean
}

interface ActionsheetProps {
  snapPoints?: number[]
  items: ActionsheetItem[]
  className?: string
}
```

---

### `menu` — isMobile: true

```ts
interface MenuItem {
  label: string
  key: string
  iconName?: string
}

interface MenuProps {
  placement?: 'top' | 'bottom' | 'left' | 'right'
  items: MenuItem[]
  className?: string
  children?: React.ReactNode
}
```

---

### `popover` — isMobile: true

```ts
interface PopoverProps {
  title?: string
  body?: string
  placement?: 'top' | 'bottom' | 'left' | 'right'
  showArrow?: boolean
  className?: string
  children?: React.ReactNode
}
```

---

### `alert-dialog` — isMobile: true

```ts
interface AlertDialogProps {
  title?: string
  body?: string
  confirmText?: string
  cancelText?: string
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'full'
  className?: string
  children?: React.ReactNode
}
```

---

### `tooltip` — isMobile: true

```ts
interface TooltipProps {
  text: string
  placement?: 'top' | 'bottom' | 'left' | 'right'
  className?: string
  children?: React.ReactNode
}
```

**gluestack-ui v4 default:**
```tsx
import { Tooltip, TooltipContent, TooltipText } from "@/components/ui/tooltip"
export default function WapTooltip({ text, placement = 'top', className, children }: TooltipProps) {
  return (
    <Tooltip placement={placement} className={className}>
      {children}
      <TooltipContent><TooltipText>{text}</TooltipText></TooltipContent>
    </Tooltip>
  )
}
```

---

---

## GROUP: form

### `form-control` — isMobile: true

```ts
interface FormControlProps {
  label?: string
  helperText?: string
  errorText?: string
  isDisabled?: boolean
  isRequired?: boolean
  isInvalid?: boolean
  isReadOnly?: boolean
  className?: string
  children?: React.ReactNode
}
```

**gluestack-ui v4 default:**
```tsx
import { FormControl, FormControlLabel, FormControlLabelText,
         FormControlHelper, FormControlHelperText,
         FormControlError, FormControlErrorText } from "@/components/ui/form-control"

export default function WapFormControl({ label, helperText, errorText, isDisabled, isRequired, isInvalid, isReadOnly, className, children }: FormControlProps) {
  return (
    <FormControl isDisabled={isDisabled} isRequired={isRequired} isInvalid={isInvalid} isReadOnly={isReadOnly} className={className}>
      {label && <FormControlLabel><FormControlLabelText>{label}</FormControlLabelText></FormControlLabel>}
      {children}
      {helperText && <FormControlHelper><FormControlHelperText>{helperText}</FormControlHelperText></FormControlHelper>}
      {errorText && isInvalid && <FormControlError><FormControlErrorText>{errorText}</FormControlErrorText></FormControlError>}
    </FormControl>
  )
}
```

---

### `input` — isMobile: true

```ts
interface InputProps {
  label?: string
  placeholder?: string
  type?: 'text' | 'password' | 'email' | 'number'
  variant?: 'outline' | 'underlined' | 'rounded'
  size?: 'sm' | 'md' | 'lg' | 'xl'
  helperText?: string
  errorText?: string
  isDisabled?: boolean
  isRequired?: boolean
  isInvalid?: boolean
  isReadOnly?: boolean
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { FormControl, FormControlLabel, FormControlLabelText,
         FormControlHelper, FormControlHelperText,
         FormControlError, FormControlErrorText } from "@/components/ui/form-control"
import { Input, InputField } from "@/components/ui/input"

export default function WapInput({ label, placeholder, type = 'text', variant = 'outline', size = 'md', helperText, errorText, isDisabled, isRequired, isInvalid, isReadOnly, className }: InputProps) {
  return (
    <FormControl isDisabled={isDisabled} isRequired={isRequired} isInvalid={isInvalid} isReadOnly={isReadOnly}>
      {label && <FormControlLabel><FormControlLabelText>{label}</FormControlLabelText></FormControlLabel>}
      <Input variant={variant} size={size} className={className}>
        <InputField type={type} placeholder={placeholder} />
      </Input>
      {helperText && <FormControlHelper><FormControlHelperText>{helperText}</FormControlHelperText></FormControlHelper>}
      {errorText && isInvalid && <FormControlError><FormControlErrorText>{errorText}</FormControlErrorText></FormControlError>}
    </FormControl>
  )
}
```

---

### `select` — isMobile: true

```ts
interface SelectOption {
  label: string
  value: string
}

interface SelectProps {
  label?: string
  placeholder?: string
  options: SelectOption[]
  variant?: 'outline' | 'underlined' | 'rounded'
  size?: 'sm' | 'md' | 'lg' | 'xl'
  helperText?: string
  isDisabled?: boolean
  isRequired?: boolean
  isInvalid?: boolean
  isReadOnly?: boolean
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Select, SelectTrigger, SelectInput, SelectPortal,
         SelectBackdrop, SelectContent, SelectItem } from "@/components/ui/select"

export default function WapSelect({ label, placeholder, options = [], variant = 'outline', size = 'md', isDisabled, className }: SelectProps) {
  return (
    <Select isDisabled={isDisabled}>
      <SelectTrigger variant={variant} size={size} className={className}>
        <SelectInput placeholder={placeholder ?? 'Select...'} />
      </SelectTrigger>
      <SelectPortal>
        <SelectBackdrop />
        <SelectContent>
          {options.map((opt) => (
            <SelectItem key={opt.value} label={opt.label} value={opt.value} />
          ))}
        </SelectContent>
      </SelectPortal>
    </Select>
  )
}
```

---

### `switch` — isMobile: true

```ts
interface SwitchProps {
  label?: string
  size?: 'sm' | 'md' | 'lg'
  isDisabled?: boolean
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Switch } from "@/components/ui/switch"
import { HStack } from "@/components/ui/hstack"
import { Text } from "@/components/ui/text"

export default function WapSwitch({ label, size = 'md', isDisabled, className }: SwitchProps) {
  return (
    <HStack space="sm" className={`items-center ${className ?? ''}`}>
      <Switch size={size} isDisabled={isDisabled} />
      {label && <Text size="sm">{label}</Text>}
    </HStack>
  )
}
```

---

### `checkbox` — isMobile: true

```ts
interface CheckboxOption {
  label: string
  value: string
}

interface CheckboxProps {
  label?: string
  options?: CheckboxOption[]
  size?: 'sm' | 'md' | 'lg'
  isDisabled?: boolean
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { CheckboxGroup, Checkbox, CheckboxIndicator, CheckboxIcon, CheckboxLabel } from "@/components/ui/checkbox"
import { CheckIcon } from "lucide-react-native"

export default function WapCheckbox({ options = [], size = 'md', isDisabled, className }: CheckboxProps) {
  return (
    <CheckboxGroup isDisabled={isDisabled} className={className}>
      {options.map((opt) => (
        <Checkbox key={opt.value} value={opt.value} size={size}>
          <CheckboxIndicator><CheckboxIcon as={CheckIcon} /></CheckboxIndicator>
          <CheckboxLabel>{opt.label}</CheckboxLabel>
        </Checkbox>
      ))}
    </CheckboxGroup>
  )
}
```

---

### `radio` — isMobile: true

```ts
interface RadioOption {
  label: string
  value: string
}

interface RadioProps {
  options: RadioOption[]
  size?: 'sm' | 'md' | 'lg'
  isDisabled?: boolean
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { RadioGroup, Radio, RadioIndicator, RadioIcon, RadioLabel } from "@/components/ui/radio"
import { CircleIcon } from "lucide-react-native"

export default function WapRadio({ options, size = 'md', isDisabled, className }: RadioProps) {
  return (
    <RadioGroup isDisabled={isDisabled} className={className}>
      {options.map((opt) => (
        <Radio key={opt.value} value={opt.value} size={size}>
          <RadioIndicator><RadioIcon as={CircleIcon} /></RadioIndicator>
          <RadioLabel>{opt.label}</RadioLabel>
        </Radio>
      ))}
    </RadioGroup>
  )
}
```

---

### `textarea` — isMobile: true

```ts
interface TextareaProps {
  label?: string
  placeholder?: string
  size?: 'sm' | 'md' | 'lg' | 'xl'
  numberOfLines?: number
  helperText?: string
  errorText?: string
  isDisabled?: boolean
  isRequired?: boolean
  isInvalid?: boolean
  isReadOnly?: boolean
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Textarea, TextareaInput } from "@/components/ui/textarea"
export default function WapTextarea({ placeholder, size = 'md', numberOfLines, isDisabled, className }: TextareaProps) {
  return (
    <Textarea size={size} isDisabled={isDisabled} className={className}>
      <TextareaInput placeholder={placeholder} numberOfLines={numberOfLines} />
    </Textarea>
  )
}
```

---

### `slider` — isMobile: true

```ts
interface SliderProps {
  value?: number
  minValue?: number
  maxValue?: number
  step?: number
  size?: 'sm' | 'md' | 'lg'
  orientation?: 'horizontal' | 'vertical'
  showLabel?: boolean
  isDisabled?: boolean
  className?: string
}
```

**gluestack-ui v4 default:**
```tsx
import { Slider, SliderTrack, SliderFilledTrack, SliderThumb } from "@/components/ui/slider"
export default function WapSlider({ value = 50, minValue = 0, maxValue = 100, step = 1, size = 'md', orientation = 'horizontal', isDisabled, className }: SliderProps) {
  return (
    <Slider defaultValue={value} minValue={minValue} maxValue={maxValue} step={step}
            size={size} orientation={orientation} isDisabled={isDisabled} className={className}>
      <SliderTrack><SliderFilledTrack /></SliderTrack>
      <SliderThumb />
    </Slider>
  )
}
```

---

### `calendar` — isMobile: true

```ts
interface CalendarProps {
  label?: string
  mode?: 'single' | 'multiple' | 'range'
  helperText?: string
  isDisabled?: boolean
  isRequired?: boolean
  isInvalid?: boolean
  isReadOnly?: boolean
  className?: string
}
```

---

### `date-time-picker` — isMobile: true

```ts
interface DateTimePickerProps {
  label?: string
  mode?: 'date' | 'time' | 'datetime'
  helperText?: string
  isDisabled?: boolean
  isRequired?: boolean
  isInvalid?: boolean
  isReadOnly?: boolean
  className?: string
}
```

---

## Summary: All 36 Components

| name | group | isMobile | gluestack-ui v4 primitive |
|------|-------|----------|--------------------------|
| container | layout | false | custom div |
| array-repeater | layout | true | SDK built-in |
| box | layout | true | Box |
| center | layout | true | Center |
| hstack | layout | true | HStack |
| vstack | layout | true | VStack |
| grid | layout | true | Grid |
| pressable | layout | true | Pressable |
| row | layout | false | custom div |
| column | layout | false | custom div |
| section | layout | false | custom section element |
| view | layout | true | handled by render() — no implementation needed |
| heading | typography | true | Heading |
| paragraph | typography | true | Text |
| html | typography | false | div + dangerouslySetInnerHTML |
| icon | typography | true | Icon + lucide-react(-native) |
| image | media | true | Image |
| video | media | true | video / expo-av Video |
| iframe | media | false | iframe |
| button | interactive | true | Button + ButtonText |
| link | interactive | true | Link + LinkText |
| fab | interactive | true | Fab + FabIcon + FabLabel |
| card | display | true | Box + VStack (no native Card) |
| card-list | display | true | Grid of Card components |
| avatar | display | true | Avatar + AvatarImage + AvatarFallbackText |
| badge | display | true | Badge + BadgeText |
| divider | display | true | Divider |
| table | display | false | Table + TableHeader + TableRow + TableData |
| skeleton | display | true | Skeleton / SkeletonText |
| spinner | feedback | true | Spinner |
| alert | feedback | true | Alert + AlertIcon + AlertText |
| progress | feedback | true | Progress + ProgressFilledTrack |
| toast | feedback | true | useToast() hook |
| accordion | disclosure | true | Accordion + AccordionItem + ... |
| tabs | disclosure | true | Tabs + TabsList + TabsTrigger + TabsContent |
| modal | overlay | true | Modal + ModalContent + ModalHeader + ... |
| drawer | overlay | true | Drawer + DrawerContent + ... |
| actionsheet | overlay | true | Actionsheet + ActionsheetItem + ... |
| menu | overlay | true | Menu + MenuItem + ... |
| popover | overlay | true | Popover + PopoverContent + ... |
| alert-dialog | overlay | true | AlertDialog + AlertDialogContent + ... |
| tooltip | overlay | true | Tooltip + TooltipContent + TooltipText |
| form-control | form | true | FormControl + FormControlLabel + ... |
| input | form | true | Input + InputField |
| select | form | true | Select + SelectTrigger + SelectContent + ... |
| switch | form | true | Switch |
| checkbox | form | true | CheckboxGroup + Checkbox + ... |
| radio | form | true | RadioGroup + Radio + ... |
| textarea | form | true | Textarea + TextareaInput |
| slider | form | true | Slider + SliderTrack + SliderFilledTrack + SliderThumb |
| calendar | form | true | Calendar |
| date-time-picker | form | true | DateTimePicker |
