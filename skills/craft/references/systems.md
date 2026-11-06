# Systems: shadcn, DESIGN.md themes, design-system pages, AI chat UI, library picks

Sources: shadcn's skill (rules, customization, design-system, design-system-page), Vercel AI Elements, and Emil Kowalski's pick-ui-library, reconciled with the Floor and house values in SKILL.md.

1. shadcn rules: principles, critical rules, base vs radix, component selection, CLI and presets, `info`, theming, house adjustments
2. DESIGN.md → themed app
3. Design-system page spec
4. AI chat UI: shadcn chat primitives, AI Elements
5. Library picks

---

## 1. shadcn

### 1.1 Principles

1. **Existing components first.** `npx shadcn@latest search` (community registries included) before writing one.
2. **Compose, don't reinvent.** Settings page = Tabs + Card + form controls. Dashboard = Sidebar + Card + Chart + Table.
3. **Built-in variants before custom styles** (`variant="outline"`, `size="sm"`).
4. **Semantic colors only** (`bg-primary`, `text-muted-foreground`), never `bg-blue-500`.

Always use the project's runner: `npx shadcn@latest`, `pnpm dlx shadcn@latest` or `bunx --bun shadcn@latest`. Run `info --json` first (§1.6).

### 1.2 Critical rules

**Styling.** `className` is for layout (`max-w-md`, `mx-auto`, `mt-4`), never to override component color or type. Customize in this order: variant → semantic token → CSS variable → new cva variant → wrapper component.

```tsx
// Incorrect
<div className="space-y-4"> <Avatar className="w-10 h-10" /> <p className="overflow-hidden text-ellipsis whitespace-nowrap" />
<Card className="bg-white dark:bg-gray-950" /> <span className="text-green-500">Active</span>
<Button className={`px-4 ${active ? "bg-primary" : ""}`} /> <DialogContent className="z-[999]" />
<p className="animate-pulse bg-clip-text">Thinking…</p>

// Correct
<div className="flex flex-col gap-4"> <Avatar className="size-10" /> <p className="truncate" />
<Card /> /* bg-card text-card-foreground from tokens */ <Badge variant="secondary">Active</Badge>
<Button className={cn("px-4", active && "bg-primary")} /> <DialogContent />
<p className="shimmer">Thinking…</p> /* scroll edges: scroll-fade / scroll-fade-x / scroll-fade-b */
```
- No `space-x-*`/`space-y-*`; use `flex gap-*`. Gaps stay on the house spacing scale (§1.8).
- No manual `dark:` color overrides. No manual z-index on Dialog, Sheet, Drawer, AlertDialog, DropdownMenu, Popover, Tooltip or HoverCard.
- Status color comes from a Badge variant, `text-destructive`, or a new token (`--success`). Never `text-emerald-600`.

**Forms.**
```tsx
// Incorrect
<div className="space-y-2"><Label>Email</Label><Input /></div>
<div className="relative"><Input className="pr-10" /><Button className="absolute right-1" /></div>
{opts.map(o => <Button variant={v === o ? "default" : "outline"} onClick={() => setV(o)}>{o}</Button>)}

// Correct
<FieldGroup>
  <Field data-invalid={!!err}>
    <FieldLabel htmlFor="email">Email</FieldLabel>
    <Input id="email" aria-invalid={!!err} />
    <FieldError>{err}</FieldError>
  </Field>
</FieldGroup>
<InputGroup><InputGroupInput /><InputGroupAddon align="inline-end"><Button size="icon-sm" /></InputGroupAddon></InputGroup>
<ToggleGroup type="single" value={v} onValueChange={setV}>{opts.map(o => <ToggleGroupItem key={o} value={o}>{o}</ToggleGroupItem>)}</ToggleGroup>
```
- Control choice: text → `Input`; fixed list → `Select`; searchable → `Combobox`; no-JS → `NativeSelect`; boolean → `Switch` (settings) or `Checkbox` (forms); one of few → `RadioGroup`; 2–5 option toggle → `ToggleGroup`; OTP → `InputOTP`; multi-line → `Textarea`.
- Settings rows: `Field orientation="horizontal"`. Hidden label: `FieldLabel className="sr-only"`.
- Related checkboxes/radios/switches: `FieldSet` + `FieldLegend variant="label"` + `FieldDescription` + `FieldGroup className="gap-3"`.
- Invalid: `data-invalid` on `Field` **and** `aria-invalid` on the control. Disabled: `data-disabled` on `Field` **and** `disabled` on the control.
- Labelled ToggleGroup: `<Field orientation="horizontal"><FieldTitle id="theme">Theme</FieldTitle><ToggleGroup aria-labelledby="theme" spacing={2}>…`.

**Composition.**
```tsx
// Incorrect
<SelectContent><SelectItem value="a">A</SelectItem></SelectContent>
<Button isLoading>Save</Button>  <hr />  <div className="animate-pulse h-4" />  <DialogContent>{/* no title */}</DialogContent>

// Correct
<SelectContent><SelectGroup><SelectItem value="a">A</SelectItem></SelectGroup></SelectContent>
<Button disabled><Spinner data-icon="inline-start" />Saving…</Button>  <Separator />  <Skeleton className="h-4" />
<DialogContent><DialogTitle className="sr-only">Edit member</DialogTitle>…</DialogContent>
```
- Items live in their Group: Select, DropdownMenu, Menubar, ContextMenu, Command; `MessageScrollerItem` in `MessageScrollerContent`; same-sender `Message`s in `MessageGroup`; `Bubble`s in `BubbleGroup`; `Attachment` rows in `AttachmentGroup`.
- Custom triggers: `asChild` (radix) or `render` (base). Dialog, Sheet and Drawer always have a Title.
- Full Card: `CardHeader`/`CardTitle`/`CardDescription`/`CardContent`/`CardFooter`. `TabsTrigger` inside `TabsList`. `Avatar` always has `AvatarFallback`.
- Callout → `Alert` + `AlertTitle` + `AlertDescription`. Empty → `Empty` (`EmptyHeader` > `EmptyMedia variant="icon"`, `EmptyTitle`, `EmptyDescription`; CTA in `EmptyContent`). Styled spans → `Badge`.
- Toast: base → `toast.add({ title })` from `@/components/ui/toast`; radix/React Aria → Sonner (`harden.md` appendix).
- Overlay choice: focused task with input → `Dialog`; destructive confirm → `AlertDialog`; side details or filters → `Sheet`; mobile bottom panel → `Drawer`; hover info → `HoverCard`; small click content → `Popover`.

**Icons.**
```tsx
// Incorrect
import { Check } from "lucide-react"            // project uses tabler
<Button><Check className="size-4 mr-2" />Save</Button>   <Item icon="check" />

// Correct
import { IconCheck } from "@tabler/icons-react"  // from info.iconLibrary
<Button><IconCheck data-icon="inline-start" />Save</Button>   <Item icon={IconCheck} />
```
No sizing classes on icons inside Button, DropdownMenuItem, Alert or Sidebar parts unless asked.

### 1.3 Base vs Radix

| Concern | radix | base |
|---|---|---|
| Composition | `asChild` | `render={<X />}`; non-button render needs `nativeButton={false}` |
| Select | inline JSX, `<SelectValue placeholder>`, `position` | `items` prop on root, placeholder as `{ value: null }` item, `alignItemWithTrigger`; supports `multiple`, object values |
| ToggleGroup | `type="single" \| "multiple"`, string value | `multiple` boolean, `defaultValue` always an array (`["daily"]`); controlled `value={[v]} onValueChange={(v) => set(v[0])}` |
| Slider | always an array (`[50]`) | single thumb is a number (`50`); ranges are arrays; cast `v as number[]` |
| Accordion | `type="single" collapsible defaultValue="item-1"` | no `type`; `multiple` boolean; array `defaultValue` |
| Toast | Sonner | `toast.add` |

### 1.4 Component selection

| Need | Component |
|---|---|
| Action | `Button` variant |
| Inputs | Input, Select, Combobox, Switch, Checkbox, RadioGroup, Textarea, InputOTP, Slider |
| 2–5 option toggle | ToggleGroup |
| Data display | Table, Card, Badge, Avatar, Item |
| Navigation | Sidebar, NavigationMenu, Breadcrumb, Tabs, Pagination |
| Overlays | Dialog, Sheet, Drawer, AlertDialog |
| Feedback | toast/Sonner, Alert, Progress, Skeleton, Spinner |
| Command palette | Command inside Dialog |
| Charts | Chart (Recharts) |
| Layout | Card, Separator, Resizable, ScrollArea, Accordion, Collapsible |
| Empty | Empty |
| Menus | DropdownMenu, ContextMenu, Menubar |
| Info | Tooltip, HoverCard, Popover |
| Chat | MessageScroller, Message, Bubble, Attachment, Marker, Trace (§4) |

### 1.5 CLI and preset workflow

1. Context from `info --json`. 2. Check installed components (`resolvedPaths.ui`); never import uninstalled ones or re-add installed ones. 3. `search`. 4. `docs <component>` and fetch the URLs **every time** you create, fix, debug or use a component; `view` for uninstalled items. 5. `add`, previewing with `--dry-run` / `--diff`. 6. Rewrite hardcoded `@/components/ui/...` imports in third-party registry files to the project alias. 7. Review added files for missing subcomponents, rule violations and icon imports (swap to `iconLibrary`). 8. If the user didn't name a registry ("add a login block"), ask. Never default.

```bash
npx shadcn@latest init --name my-app --preset base-nova --template vite   # next|start|react-router|astro|laravel; --monorepo
npx shadcn@latest init --defaults                       # next + nova
npx shadcn@latest apply a2r6bw [--only theme,font]     # icon is not supported by --only
npx shadcn@latest preset decode|url|open a2r6bw ; npx shadcn@latest preset resolve --json
npx shadcn@latest add button card dialog | @magicui/shimmer-button | owner/repo/item | --all
npx shadcn@latest add button --dry-run | --diff button.tsx
npx shadcn@latest search @shadcn -q "sidebar" -t ui     # -t ui|block|hook
npx shadcn@latest docs button dialog select ; npx shadcn@latest view @shadcn/button
npx shadcn@latest mcp init
```

- **Switching presets: ask first.** Overwrite (`apply <code>`), partial (`apply <code> --only theme,font`), merge (`init --preset <code> --force --no-reinstall`, then per-component `--dry-run`/`--diff`), or skip. `apply` needs `components.json` and preserves the base. Never decode preset codes or build preset URLs by hand.
- **Updating upstream with local edits:** `add <c> --dry-run`, then `add <c> --diff <file>`; merge by hand. **Never `--overwrite` without explicit approval.** Never fetch raw GitHub files.
- Styles: `vega`, `nova`, `maia`, `lyra`, `mira`, `luma`, `rhea`, `sera` (§2.1).
- Registry authoring: root `registry.json` needs `name`, `homepage`, `items[]`; item `name`, `type`, `files`, `dependencies`, `registryDependencies` (bare = official, `@ns/item`, `owner/repo/item[#ref]`), `cssVars`; `registry:file` and `registry:page` need a `target`.

### 1.6 `info` fields that change what you write

| Field | Use |
|---|---|
| `aliases` | real import prefix; never hardcode `@/` |
| `isRSC` | add `"use client"` for state, effects, handlers, browser APIs |
| `tailwindVersion` | v4 → `@theme inline`; v3 → `tailwind.config.js` |
| `tailwindCssFile` | the only CSS file you edit; never create another |
| `style`, `base` | style family; `radix` vs `base` (§1.3) |
| `iconLibrary` | icon imports |
| `resolvedPaths`, `framework`, `packageManager`, `preset` | where files live, runner for non-shadcn installs |

### 1.7 Theming

Pipeline: CSS variables in `:root` and `.dark` → Tailwind utilities → components. Pairs: `background`, `card`, `popover`, `primary`, `secondary`, `muted`, `accent`, `destructive`, each with `-foreground`; plus `--border`, `--input`, `--ring`, `--chart-1..5`, `--sidebar-*`. Dark mode is a `.dark` class on root (`next-themes`: `<ThemeProvider attribute="class" defaultTheme="system" enableSystem>`).

```css
:root { --warning: oklch(0.84 0.16 84); --warning-foreground: oklch(0.28 0.07 46); }
.dark { --warning: oklch(0.41 0.11 46); --warning-foreground: oklch(0.99 0.02 95); }
@theme inline { --color-warning: var(--warning); --color-warning-foreground: var(--warning-foreground); }
/* v3: colors: { warning: "oklch(var(--warning) / <alpha-value>)" } */
```
Then a cva variant: `warning: "bg-warning text-warning-foreground"` (hover tint inside the gated hover variant). Higher-level patterns get a wrapper (`ConfirmDialog` over `AlertDialog`). `--radius` is global (`rounded-lg` = `var(--radius)`, `rounded-md` = `calc(var(--radius) - 2px)`); for a brand scale pin `--radius-sm…4xl` (§2.3).

### 1.8 House adjustments to stock shadcn

Apply these once per project; they enforce the Floor (SKILL.md §3).

```css
/* Hover only on fine pointers (Tailwind v4's default is (hover: hover) alone) */
@custom-variant hover (@media (hover: hover) and (pointer: fine) { &:hover { @slot; } });
```
- **Focus rings appear instantly.** Stock inputs and selects ship `transition-[color,box-shadow]`, which animates the ring. Change it to `transition-colors` (or remove).
- **No `transition-all`.** Stock Button ships it; replace with `transition-[color,background-color,border-color,transform]`.
- **Press feedback:** add `enabled:active:scale-[0.97] duration-150 ease-(--ease-out)` to the Button base, never on 100+/day controls (SKILL.md §5).
- **Spacing:** Tailwind classes on the house scale are `0.5 1 2 3 4 6 8 10 12 16 20 24 30 36` (2…144px). `gap-5`, `gap-7`, `gap-14` and friends are off-scale.
- **z-index:** leave shadcn overlays alone; custom layers use the house z-scale.
- **Shimmer / spinners:** `linear` is fine; under `prefers-reduced-motion: reduce` stop the sweep and keep the static text.

---

## 2. DESIGN.md → themed shadcn app

Inputs: an optional preset code (pass through, never decode); the DESIGN.md (required; copy it to the app root, and if macOS privacy blocks the read, ask the user to copy it in, never guess); template `vite` (`next` for Next.js/App Router/SSR); app name kebab-cased from the brand; base `base` (`radix` only if asked).

### 2.1 Scaffold and pick a style by geometry

`init --preset <code> --template vite --name <app> --no-monorepo -y`, or without a code `init --base base --preset <style> …`. Init without `--preset` or `--base` blocks on a prompt even with `-y`. Then `info --json`. The theme gets replaced, so pick the style for its **geometry**:

| Style | Character | Pick when the DESIGN.md has |
|---|---|---|
| vega | classic shadcn | neutral SaaS geometry |
| nova | compact, reduced padding | 32–36px controls, 6–8px radius, hairlines over shadows |
| maia | soft, rounded, generous | 40px+ controls, 12px+ card radius, editorial pacing |
| lyra | boxy, sharp | 0–4px radius, hard edges, mono/technical type |
| mira | dense | data UIs, 28–32px controls, tight tables |
| luma | rounded, soft elevation | pill buttons, layered soft shadows, macOS feel |
| rhea | luma, compact | luma softness at product density |
| sera | editorial, typographic | serif display, magazine hierarchy |

State it in one line: "Picked maia: 40px controls, 12–16px cards, generous padding."

### 2.2 Install and read

- `add --all -y`, then count from disk (`ls <ui>/*.tsx | wc -l`), never from memory. Wrap the app in `TooltipProvider` and mount the Toaster. Typecheck: Vite `npx tsc -p tsconfig.app.json --noEmit` (root `tsc` checks nothing); Next `npx tsc --noEmit`.
- Read the DESIGN.md: frontmatter (`colors`, `typography`, `rounded`, `spacing`, `components`) and prose (Overview, Colors, Typography, Layout, Elevation, Shapes, Components, Do's and Don'ts, Responsive, Known Gaps), or prose with `{colors.x}` refs. Grep headings first; read frontmatter, Do's/Don'ts, Iteration Guide and Known Gaps in full.
- Extract one table: canvas/ink; action color and its states (hover, active, pressed, disabled, on-primary); secondary button; surfaces; lines; focus (outline vs ring, color, width, offset, alpha); type families + substitutes; type scale (size, weight, leading, tracking); radius scale; elevation tiers verbatim; control heights; hard rules.

### 2.3 Map tokens (edit only `tailwindCssFile`)

The DESIGN.md is the project's system (SKILL.md §0), so keep its hex values verbatim. Tokens you *derive* (dark surfaces, hover tints, missing states) are written in OKLCH.

| DESIGN.md role | Token |
|---|---|
| canvas | `--background`, usually `--popover` too |
| ink | `--foreground`, `--card-foreground`, `--popover-foreground` |
| CTA | `--primary` / `--primary-foreground`; hover/pressed → new `--primary-active` |
| secondary button | `--secondary` |
| soft band | `--muted` |
| descriptive text | `--muted-foreground` |
| ghost hover, active tab, menu highlight | `--accent` |
| card fill | `--card` |
| hairline | `--border`, `--input` |
| focus | `--ring` |
| error | `--destructive` |
| illustration accents | `--chart-1..5` + named tokens |
| nav rail | `--sidebar-*` |

Extra tokens go in `:root` (and `.dark`) plus `@theme inline { --color-x: var(--x) }`.

```css
@import "@fontsource-variable/inter";            /* DESIGN.md-named substitutes, at the top */
@import "@fontsource-variable/fraunces";

@theme inline {
  /* Radius: base-nova uses sm accents · md menu/select/command items · lg buttons, inputs, tabs, popovers, menus
     · xl cards, dialogs, command · 2xl toasts, large containers · 4xl pill badges */
  --radius-sm: 4px; --radius-md: 6px; --radius-lg: 8px; --radius-xl: 12px;
  --radius-2xl: 16px; --radius-3xl: 16px; --radius-4xl: 9999px;
  --font-sans: "Inter Variable", system-ui, sans-serif;
  --font-heading: "Fraunces Variable", Georgia, serif;   /* styles CardTitle/DialogTitle/SheetTitle: must read well at 16px */
  --font-mono: ui-monospace, "SF Mono", Menlo, monospace;
}
@theme {  /* elevation in @theme, NOT @theme inline; copy multi-layer shadows verbatim */
  --shadow-xs: 0 0 #0000;   /* hairline-only systems: inputs and outline buttons use xs */
  --shadow-sm: 0 1px 3px rgb(20 20 19 / 0.08);
  --shadow-lg: 0 1px 3px rgb(20 20 19 / 0.08), 0 8px 24px rgb(20 20 19 / 0.06);
}
/* Type scale: one @utility per token. NOT --text-* keys: cn() reads unknown text-* as a color and drops it next to text-muted-foreground */
@utility type-display { font-family: var(--font-heading); font-size: clamp(34px, 3vw + 16px, 48px); line-height: 1.1; letter-spacing: -0.021em; font-weight: 500; text-wrap: balance; }
@utility type-caption { font-size: 12px; line-height: 1.4; font-weight: 500; }   /* sentence case, no tracking */
/* Focus as a solid outline (unlayered, so it beats utilities) */
:is(button, a[href], [role="button"], [role="tab"], [role="checkbox"], [role="radio"], [role="switch"], [role="slider"]):focus-visible {
  outline: 2px solid var(--ring); outline-offset: 2px; --tw-ring-shadow: 0 0 #0000;
}
```
- If the DESIGN.md has no elevation, use the house elevation (SKILL.md §4). Verify radius coverage with `grep -ohE "rounded-(\[[^ \"]+\]|[a-z0-9]+)" <ui>/*.tsx | sort | uniq -c`; cap tiny buttons with `rounded-[min(var(--radius-md),6px)]`.
- Convert px tracking to em (`-1px / 48px = -0.021em`). Respect "no tracking below size X" and "display never bold".
- **Uppercase captions:** a DESIGN.md caption spec like "12px / 500 / 1.5px tracking / uppercase" collides with the Floor. Build `type-caption` in sentence case without tracking, list it as a deviation, and ask before switching to the spec's version.
- A ring-with-alpha focus spec ("3px coral at 15%") replaces `ring-ring/50` on field components; add `input, textarea, select, [role="combobox"]` to the outline selector if fields also get the outline. Either way the ring stays ≥3:1 and appears instantly.
- Dark: DESIGN.md dark surfaces go in `.dark`. `className="dark"` on any section renders a dark band (`@custom-variant dark (&:is(.dark *))`). Light-only spec → derive dark and tell the user.

### 2.4 Recipes → cva variants, heights, states

Keep variant names: primary → Button `default`; tinted secondary → `secondary`; canvas + border → `outline`; ghost and nav links → `ghost`; text link → `link`; circular icon → `icon*` sizes + `rounded-full`; badges → Badge variants; card → Card (fill, ring, `--card-spacing`); text input → Input, InputGroup, Select, Textarea; category tabs → Tabs.

**Control heights move together.** The base-nova default `h-8` (32px) lives in button (`h-8`, icon `size-8`), input, input-group, select (`data-[size=default]:h-8`), native-select, combobox (`min-h-8`), toggle (`h-8 min-w-8`), input-otp (`size-8`), tabs (`group-data-horizontal/tabs:h-8`), menubar and command (`h-8!`). Change all of them or none: `grep -n "\bh-8\b\|size-8\b" <ui>/*.tsx`. With no height in the DESIGN.md, use 40px (`h-10`) for app UI.

States follow the spec exactly ("darken on press only" → `active:`, not `hover:`). Keep hover on ghost buttons and menu items even if the spec omits it, and say so. Turn each "Don't" into a final check (radius off-scale, accent on buttons, bold display).

### 2.5 Verify and report

- Typecheck; zero console errors; page loads at `scrollY === 0` (otherwise a component steals scroll on mount).
- No horizontal overflow at **375px and 1280px**: `document.documentElement.scrollWidth <= innerWidth`.
- Screenshot the hero, Foundations, one form and one dark band; compare canvas, CTA, display font, radius and focus ring to the spec. Toggle dark mode once.
- Every text pair ≥ **4.5:1** (UI components and focus ≥ 3:1). Report pairs that fail even when the DESIGN.md specified them.
- **Report:** app path, dev URL, component count from disk, style + reason, mappings (colors, fonts and substitutes, radius, shadows, focus), contrast failures, every edited component file and why, every deviation (derived dark, kept hover, substituted fonts, sentence-case captions).

**Traps:** inline `Command` scrolls to its first item on mount (control `value`, initial `"none"`) · `Sidebar` is fixed (`<SidebarProvider className="min-h-0">` + `collapsible="none"` on showcase pages) · `MessageFooter` belongs inside `MessageContent` · base `Select` shows the raw value without `items` · `Resizable` v4 uses `orientation="horizontal"`, `defaultSize="40%"` · `DirectionProvider` also needs `dir="rtl"` · leave bottom padding for NavigationMenu · header links rendered as Button need `nativeButton={false} render={<a />}` (base) or `asChild` (radix).

---

## 3. Design-system page

Distilled from Primer, Carbon, Atlassian, Pajamas, Cloudscape, Paste, Lightning, Material 3, Evergreen, Orbit and EightShapes.

**Principles:** foundations → components → compositions · roles, not palettes (each color is a surface + foreground pair with measured contrast; raw palette second) · typical case first, then variants by priority, shared axes (size, icon), then states · pin states so they show without interaction; one label style, never the state name inside the component · don't render every combination: primitives get a variant × state matrix, composed components get a typical example plus additions · real brand-voice copy, no lorem, including long, empty, error and loading · composition proves the system · render the DESIGN.md's Do's and Don'ts.

**Outline:** 1 Header (name, "Themed from DESIGN.md", anchors, light/dark toggle, CTA) · 2 Overview (hero recipe) · 3 Foundations (color roles, palette, type, spacing, radius, elevation, motion if specified, icons) · 4 Actions (Button, ButtonGroup, Toggle, ToggleGroup, Badge, Kbd) · 5 Inputs (Field, Input, InputGroup, Textarea, Select, NativeSelect, Combobox, Checkbox, RadioGroup, Switch, Slider, InputOTP, Calendar, Label) · 6 Navigation (Tabs, Breadcrumb, Pagination, NavigationMenu, Menubar, Sidebar) · 7 Data display (Card, Table, Chart, Item, Avatar, Accordion, Collapsible, Carousel, ScrollArea, Resizable, AspectRatio, Separator) · 8 Feedback (Alert, toast, Progress, Spinner, Skeleton, Empty) · 9 Overlays (Dialog, AlertDialog, Sheet, Drawer, Popover, HoverCard, Tooltip, DropdownMenu, ContextMenu, Command) · 10 Conversation · 11 Do/Don't · 12 Recipes + example screen · 13 Footer.

**Foundations**
- Color roles (required pairs): background/foreground, card, popover, primary, secondary, muted, accent, destructive on background, and every extra text token on background. A `ColorPair` measures contrast live and flags anything under 4.5:1 ("a brand primary at 3:1 behind 14px labels is a real finding").
- Palette swatches grouped as in the DESIGN.md, labeled "Warm stone, `surface-warm`". Include border, input, ring, chart-1…5, sidebar-*.
- Type specimen table: Token | Spec (size / leading / weight / tracking) | Use | Specimen (one line of real copy). Order display → title → body → label/caption → code. Follow with one body-size paragraph containing a link and inline code.
- Spacing: the house scale as labeled bars, then the semantic ladder: inside a component → between related items → between groups → between sections.
- Radius: the same box at each step, labeled `md, 8px: buttons, inputs`. Elevation: flat (canvas, bands) → hairline (cards) → soft (popovers, menus) → deep (dialogs). Motion: a token / ms / use table, only if defined. Icons: one row per size in foreground and muted.

**Tiers and block anatomy.** Primitives: variant × state matrix, size ladder, icon row. Fields: typical, then description, error, disabled, required. Composed: typical, then one element added at a time, then edge cases. Overlays: a real labeled trigger each. Shell: framed, `collapsible="none"`. Conversation: one realistic thread, then variant rows. Each block: name + when to use → typical → variants in priority order with a one-line "when" → size ladder → icon (leading, trailing, icon-only) → states → edge cases. Wide components span the full row.

**Pinned states.** Same gate as the Floor, plus a `data-preview` hook:
```css
@custom-variant hover { @media (hover: hover) and (pointer: fine) { &:hover { @slot; } } &[data-preview~="hover"] { @slot; } }
@custom-variant focus-visible (&:focus-visible, &[data-preview~="focus"]);
@custom-variant active (&:active, &[data-preview~="active"]);
```
`StateMatrix` columns: default, hover, focus, active, disabled; plus `loading` (Spinner + disabled) for Button and `invalid` (`aria-invalid`) for fields. If the spec says a state doesn't change, the column stays identical. Column labels sit above the row, never inside the component.

**Edge cases:** long text (Button label, Badge, Item title, Table cell, Breadcrumb truncation) · empty (Table → Empty, Combobox no matches, empty Command) · error (FieldError, destructive Alert, Attachment `state="error"`) · loading (Spinner, Skeleton, Progress, toast loading) · many items (AvatarGroup count, Pagination ellipsis, ScrollArea) · missing image (AvatarFallback) · disabled (`data-disabled`) · RTL (`DirectionProvider dir="rtl"`). Pull values from the catalog in `harden.md` §1.8.

**Do/Don't pairs.** Pick the 2–4 easiest-to-break rules; render each pair in `grid gap-4 md:grid-cols-2` with a one-line caption ("One primary action per group."). Mark the Don't with `text-destructive`, not a new color.

**Recipes** (`hero-band`, `feature-card`, `pricing-tier-card`, `callout-card`, `footer`) are the spec's acceptance tests: if one needs a raw value, the token mapping is incomplete. Example screen: a settings page (FieldGroup, Switch, Select, Button) or a list view (Table, Badge, Pagination, Empty).

**Coverage.** Showcase files live in `src/showcase/<section>.tsx` (Vite) or `app/design-system/` (Next). Drive the uncovered list to zero:
```bash
UI=src/components/ui; SHOW=src/showcase   # Next: UI=components/ui; SHOW=app/design-system
for f in $UI/*.tsx; do n=$(basename "$f" .tsx); grep -rqE "ui/$n['\"]" "$SHOW" || echo "uncovered: $n"; done
```

**Checklist:** every component renders · color pairs show contrast, every pair under 4.5:1 noted · type specimen covers all tokens · radius and elevation ladders name their users · primitives have state matrices (Button and fields include loading and invalid) · edge cases · do/don't pairs · recipes + example screen · theme toggle and a scoped dark band if relevant · no lorem, no state names inside components, no tracked uppercase captions.

---

## 4. AI chat UI

**Decide first.** If the project has shadcn's chat primitives installed (`message-scroller`, `message`, `bubble`, `trace` in `resolvedPaths.ui`), use §4.1. Otherwise use AI Elements (§4.2). Never mix both in one thread. Both agree on: auto-scroll with a jump-to-latest button, the user's message anchoring the scroll, collapsible tool and reasoning rows, explicit tool approval, and a shimmer for "thinking". Show what the AI did (tool names, sources, diffs); no sparkle or wand icons as AI shorthand (the Floor).

### 4.1 shadcn chat primitives

`npx shadcn@latest add message-scroller message bubble attachment marker trace code-block` (same names for base and radix).

```tsx
<MessageScrollerProvider autoScroll>
  <MessageScroller>
    <MessageScrollerViewport>
      <MessageScrollerContent>
        {messages.map((m) => (
          <MessageScrollerItem key={m.id} messageId={m.id} scrollAnchor={m.role === "user"}>
            <Message align={m.role === "user" ? "end" : "start"}>
              <MessageAvatar><Avatar><AvatarFallback>{initials(m.author)}</AvatarFallback></Avatar></MessageAvatar>
              <MessageContent>
                <MessageHeader>{m.author}</MessageHeader>
                <Bubble variant={m.role === "user" ? "default" : "ghost"}><BubbleContent>{m.text}</BubbleContent></Bubble>
                <MessageFooter>{time}</MessageFooter>   {/* inside MessageContent */}
              </MessageContent>
            </Message>
          </MessageScrollerItem>
        ))}
      </MessageScrollerContent>
    </MessageScrollerViewport>
    <MessageScrollerButton />
  </MessageScroller>
</MessageScrollerProvider>
```
- Never a raw overflow div, `ScrollArea`, `useStickToBottom`, `ResizeObserver` or scrollTop math. `defaultScrollPosition="end" | "last-anchor"`; for SSR "end" inject the inline scroll script with `suppressHydrationWarning` (+ CSP `nonce`).
- Bubble variants: default, secondary, muted, tinted, outline, ghost, destructive; `BubbleReactions side align`. Never a styled `bg-primary rounded-2xl` div.
- Attachment `state`: idle, uploading, processing (both shimmer the title), error, done; `size` default/sm/xs; `orientation`; parts `AttachmentMedia variant="icon"|"image"`, `AttachmentContent/Title/Description`, `AttachmentActions/Action`.
- Marker for system notes and dividers: `variant` default, separator (centered label with rules), border. Not Separator + span.
- **Trace** for assistant activity: `TraceLine` = `TraceIcon` + `TraceTitle` + optional `TraceDescription` (count, duration, diff stat). `status` = `"running" | "error" | "pending"` or omitted; wording Running / Ran / Failed to run. Map SDK states yourself; never pass one through, never dim with `opacity-*`. Behavior wraps the line (`TraceCollapsible` > `TraceCollapsibleTrigger` > `TraceLine`; or `<a>`; or `SheetTrigger`); the line is never itself the trigger. A tool row stays a plain line until output arrives, then becomes collapsible. One `Trace` per run of parts; a text part or a pending approval ends the run. A pending approval renders outside Trace as an `Item` with Approve (default) / Deny (outline) in `ItemActions`. Trailing icon only for a link (arrow-up-right + sr-only "(opens in a new tab)") or an overlay (chevron-right).
- CodeBlock for code, terminal, tool I/O, logs, stack traces and diffs (never raw `<pre>`): `wrap` for tool output, `lineNumbers` for source, `max-h-*` to cap, `variant="destructive"` for errors, `CodeBlockLine variant="added"|"removed"|"highlighted"`; in a Trace row `size="sm" wrap max-h-64`. Highlight with `useHighlightedCode` (client) or `await highlightCode()` (RSC); don't highlight tool output.

| AI Elements concept | shadcn build |
|---|---|
| Tool, Reasoning, Chain of Thought, Task, Plan, Sources, Subagent | Trace |
| Checkpoint | Marker separator + Button calling `setMessages` |
| Confirmation | Item + ItemActions |
| PromptInput | InputGroup (textarea + attachment chip + submit) |
| Suggestion | Button calling `sendMessage({ text })` |
| ModelSelector | DropdownMenu + DropdownMenuRadioGroup (second one for effort) → `body.model`, `body.effort` |
| InlineCitation | HoverCard on a short Button from a `source-url` part |
| Context | HoverCard + Progress from message metadata `usage` |
| File tree | nested Collapsibles, files as Buttons |

### 4.2 AI Elements

A registry on top of shadcn; components are copied into `@/components/ai-elements/` as source. **Prerequisites:** Node 18+, Next.js with the AI SDK, shadcn (auto-installed if missing). Optional: Vercel AI Gateway with `AI_GATEWAY_API_KEY` in `.env.local`.

```bash
npx ai-elements@latest add conversation message prompt-input tool confirmation reasoning   # or pnpm dlx / bunx --bun
```
```tsx
"use client";
import { useChat } from "@ai-sdk/react";
import { Conversation, ConversationContent, ConversationScrollButton } from "@/components/ai-elements/conversation";
import { Message, MessageContent, MessageResponse } from "@/components/ai-elements/message";

const { messages, sendMessage, status, addToolApprovalResponse } = useChat();
<Conversation>
  <ConversationContent>
    {messages.map(({ id, role, parts }) => (
      <Message from={role} key={id}>
        <MessageContent>
          {parts.map((part, i) => part.type === "text" ? <MessageResponse key={`${id}-${i}`}>{part.text}</MessageResponse> : null)}
        </MessageContent>
      </Message>
    ))}
  </ConversationContent>
  <ConversationScrollButton />
</Conversation>
```
Server route: `streamText({ model, messages: await convertToModelMessages(messages), tools })` → `return result.toUIMessageStreamResponse()`, with `export const maxDuration = 30`. User messages sit on a filled bubble; assistant messages run full-width and flat. Customize by editing the copied source.

**Tool states** (`ToolHeader state` → label; `getStatusBadge(state)`; `type ToolPart = ToolUIPart | DynamicToolUIPart`):

| State | Label |
|---|---|
| `input-streaming` | Pending |
| `input-available` | Running |
| `approval-requested` | Awaiting approval |
| `approval-responded` | Responded |
| `output-available` | Completed (auto-opens) |
| `output-error` | Error |
| `output-denied` | Denied |

Every state also changes the badge icon or text, not just its color (the Floor).

**Approval flow.** The server tool sets `requireApproval: true`; the client renders:
```tsx
{tool.approval && (
  <Confirmation approval={tool.approval} state={tool.state}>
    <ConfirmationRequest>This tool wants to delete <code>{path}</code>. Approve?</ConfirmationRequest>
    <ConfirmationAccepted><CheckIcon /> You approved this run</ConfirmationAccepted>
    <ConfirmationRejected><XIcon /> You rejected this run</ConfirmationRejected>
    <ConfirmationActions>
      <ConfirmationAction variant="outline" onClick={() => addToolApprovalResponse({ id: tool.approval!.id, approved: false })}>Reject</ConfirmationAction>
      <ConfirmationAction onClick={() => addToolApprovalResponse({ id: tool.approval!.id, approved: true })}>Approve</ConfirmationAction>
    </ConfirmationActions>
  </Confirmation>
)}
```

**PromptInput props:** `onSubmit(message, event)`, `accept`, `multiple`, `globalDrop`, `syncHiddenInput`, `maxFiles`, `maxFileSize` (bytes), `onError`. Behavior: auto-resizing textarea, Enter submits, Shift+Enter is a newline, drag-and-drop attachments, submit icon follows `status`, optional `PromptInputProvider` for lifted state. 16px text on mobile (the Floor).

**Troubleshooting:** unstyled → shadcn not set up for Tailwind 4 / `globals.css` missing the imports · CLI added nothing → run from the project root, check `components.json`, use `@latest` · stuck in light mode → toggle the class shadcn expects on `<html>` · "module not found" → `tsconfig.json` `"paths": { "@/*": ["./*"] }`.

**Catalog (pick, don't install all)**

| Group | Components |
|---|---|
| Chat core | Conversation (auto-scroll, scroll button, Markdown download) · Message (branching, GFM, actions) · PromptInput · Suggestion · Attachments (grid, inline, list) · Shimmer · Checkpoint |
| Agent activity | Reasoning (auto-opens while streaming) · ChainOfThought · Tool · Confirmation · Task · Plan · Queue · Agent · Context (token ring, cost) · Sources · InlineCitation · ModelSelector (cmdk) · OpenIn |
| Code and dev | CodeBlock (Shiki) · Artifact · WebPreview · JSXPreview · Sandbox · Terminal (ANSI) · Snippet · StackTrace · TestResults · Commit · FileTree · EnvironmentVariables (masked) · PackageInfo · SchemaDisplay |
| Voice and media | Persona (Rive avatar) · SpeechInput · MicSelector · VoiceSelector · AudioPlayer · Transcription · Image |
| Canvas (React Flow) | Canvas · Node · Edge · Connection · Controls · Panel · Toolbar |

Motion inside chat follows SKILL.md §5: streaming text and Reasoning open/close happen many times per session, so keep them near-instant; Shimmer stops under reduced motion.

---

## 5. Library picks

Identify the **task**, not the library named. Check `package.json`: if a listed library is installed, use it; if a competitor is installed, flag it but don't churn. Recommend **one** library in one sentence and wire it only if asked. If the right answer is off this list, say so.

| Task | Pick |
|---|---|
| Unstyled accessible primitives (dialog, popover, menu, select) | Base UI (if shadcn is installed, its components already wrap Base UI or Radix: use them) |
| Command palette | cmdk (shadcn `Command`) |
| Toasts | Sonner (shadcn base: its `toast`) |
| OTP input | input-otp |
| Live tuning while designing | DialKit (`explore.md`) |
| Shipped in-app control panel / GUI | Leva |
| Springs, layout, enter/exit animation | Motion (simple hover or fade: plain CSS) |
| Animated numbers | NumberFlow |
| Animated text | torph |
| 3D globe | Cobe |
| Dynamic OG images | Satori |
| Syntax highlighting | Shiki |
| Real-time / streaming charts | Liveline |
| General charts | Recharts (shadcn `Chart`) |
| Drag and drop | dnd kit |
| Virtualized lists | Virtuoso |
| State | Zustand |
| Conditional className | clsx (via `cn()`) |
| Typed variants (size, intent, state) | cva |
| Theme switch without flash | next-themes |

| You see | Swap to |
|---|---|
| Hand-built toast stack | Sonner |
| `div` dropdown with manual focus handling | Base UI / shadcn `DropdownMenu` |
| Number animated by re-rendering every frame | NumberFlow |
| 1,000+ rows rendered at once | Virtuoso |
| `useState` passed through five levels | Zustand |
| Nested className ternaries | clsx for ad-hoc, cva for real variants |
