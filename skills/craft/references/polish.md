# Polish

Sources: better-ui and make-interfaces-feel-better (Jakub Krehel), Better Interface, spacing-audit (Tran Mau Tri Tam), Emil Kowalski's UI cheatsheet and emil-design-eng.

**Contents:** 1 Ground rules, 2 Surfaces (radii, shadows, outlines, hit areas), 3 Optical alignment, 4 Icons, 5 Press feedback, 6 Type rendering, 7 Spacing audit, 8 Layout grouping, 9 Rendering gotchas, 10 Small details, 11 Before you finish

Motion recipes (enter, exit, stagger, springs, gestures) live in `motion.md`. This file keeps only press feedback and the icon crossfade, because they are polish staples.

## 1. Ground rules

- Polish applies once the interaction is sound. A polish finding never outranks a broken interaction (see `review.md` for domain order).
- Express every fix in the project's styling system (Tailwind, plain CSS, CSS Modules, CSS-in-JS). Never add a second styling system for a polish fix.
- Values here are **exact**, not ranges. `cubic-bezier(0.2, 0, 0, 1)` is not `cubic-bezier(0.4, 0, 0.2, 1)`. The only by-eye exceptions are optical nudges and the concentric-radius padding cutoff.
- A deliberate, consistent project convention (for example "no shadows anywhere") is a preference, not a finding. The same detail done two different ways is a finding.
- Everything here sits on top of the Floor (SKILL.md §3) and the house values (SKILL.md §4). Where a project token exists, use the token.

## 2. Surfaces

### Concentric radii

`outer radius = inner radius + padding + border width`. Mismatched nested radii are the most common reason a surface "feels off".

| Outer padding | Inner radius | Outer radius | Tailwind |
|---|---|---|---|
| 4px | 8px | 12px | `rounded-xl p-1` > `rounded-lg` |
| 8px | 8px | 16px | `rounded-2xl p-2` > `rounded-lg` |
| 8px | 12px | 20px | `rounded-[20px] p-2` > `rounded-xl` |
| 12px | 12px | 24px | `rounded-3xl p-3` > `rounded-xl` |

```css
.card       { border-radius: 20px; /* 12 + 8 */ padding: 8px; }
.card-inner { border-radius: 12px; }
```

- Padding past 24px, or deliberately asymmetric padding: treat the layers as separate surfaces and keep each component's own radius token.
- Inner radius never drops below 2px. If the math yields less, use 2px or square the inner element.
- One radius per nesting level across the app. No card inside a card (Floor).

### Shadows for elevation, borders for structure

Replace borders whose only job is depth with the house elevation token. Keep real borders on dividers, separators, table cells, selected states and form inputs (inputs need 3:1 non-text contrast).

```css
:root {
  --shadow-border:
    0 0 0 1px oklch(0 0 0 / 0.06),
    0 1px 2px -1px oklch(0 0 0 / 0.06),
    0 2px 4px 0 oklch(0 0 0 / 0.04);
  --shadow-border-hover:
    0 0 0 1px oklch(0 0 0 / 0.08),
    0 1px 2px -1px oklch(0 0 0 / 0.08),
    0 2px 4px 0 oklch(0 0 0 / 0.06);
}
.dark {
  --shadow-border: 0 0 0 1px oklch(1 0 0 / 0.08);       /* depth shadows vanish on dark */
  --shadow-border-hover: 0 0 0 1px oklch(1 0 0 / 0.13);
}
.card {
  position: relative;
  border: 1px solid transparent;   /* forced-colors drops box-shadow; this keeps an edge and the box size */
  box-shadow: var(--shadow-border);
}
/* Hover lift: fade a pseudo-element, never transition box-shadow itself */
.card::after {
  content: ""; position: absolute; inset: -1px; border-radius: inherit;
  box-shadow: var(--shadow-border-hover); opacity: 0; pointer-events: none;
  transition: opacity 150ms ease;
}
@media (hover: hover) and (pointer: fine) {
  .card:hover::after { opacity: 1; }
}
```

- Layers: 1px ring, lift, ambient depth. Alpha shadows adapt to any background; solid borders do not. Use the token on cards, bordered-style buttons, dropdowns, popovers, modals and anything on varied backgrounds.
- Never convert dividers (`border-b`, `border-t`, side borders). Never combine a visible border with a wide shadow. Tailwind 4: define both values in `@theme` and use `shadow-border`.

### Image outlines

Content images get a 1px inset outline so their edge holds against any surface.

```css
img.content { outline: 1px solid oklch(0 0 0 / 0.1); outline-offset: -1px; }
.dark img.content { outline-color: oklch(1 0 0 / 0.1); }
```

Tailwind: `outline outline-1 -outline-offset-1 outline-black/10 dark:outline-white/10`.

- Pure black or white alpha only. Never slate, zinc, neutral, gray, `#111827`, the project's near-black or the accent: a tinted outline reads as dirt on the image edge.
- `outline`, not `border`: no layout shift, and the `-1px` offset hugs the radius.
- Skip transparent artwork (logos, illustrations, cutouts).

### Hit areas

Floor minimum is 24×24. Aim for 44×44 on touch and 40px controls in app UI. Grow the target, not the visual, with a pseudo-element:

```css
.checkbox { position: relative; width: 20px; height: 20px; }
.checkbox::after {
  content: ""; position: absolute; top: 50%; left: 50%;
  translate: -50% -50%; width: 44px; height: 44px;
}
```

Tailwind: `relative size-5 after:absolute after:top-1/2 after:left-1/2 after:size-11 after:-translate-1/2`. If the extended area would overlap another control, shrink it to the largest size that does not collide. Label and control share one hit target (no dead strip between a checkbox and its text).

## 3. Optical alignment

Geometric centering is wrong whenever the shape is asymmetric. Fix by eye, then lock the value.

| Case | Fix | Tailwind |
|---|---|---|
| Button with icon | Icon-side padding = text-side padding − 2px (`16px` text side, `14px` icon side) | trailing icon: `ps-4 pe-3.5`; leading icon: `ps-3.5 pe-4` |
| Play triangle | `translate: 2px 0` toward the point. Physical: never mirror in RTL | `translate-x-[2px]` |
| Stars, arrows, carets, chevrons | Fix the SVG `viewBox` or path first; fallback is a 1px wrapper nudge | `translate-x-px` |
| Icon next to cap-height text | Size the icon 1em–1.25em of the adjacent font size and center on cap height, not line box | `size-[1.125em]` |

These 2px and 1px nudges are optical offsets. The spacing audit (§7) leaves them alone.

## 4. Icons

### Library, size, color

- One icon library per surface. Never mix libraries on one screen. Never emoji as icons (Floor).
- Always SVG, always `currentColor`. Strip hardcoded `fill="#666"` / `stroke="#333"` on import.
- Use the library's native grids: 16, 20, 24. Test at the smallest size it renders (often 16px); prefer a simplified glyph at small sizes.

### Stroke matches the adjacent text weight

| Adjacent text | Rendered stroke |
|---|---|
| 400 weight, 14–16px | 1.5px |
| 500–600 weight | 2px |
| 700, or an emphasized standalone icon | 2.5px |

A 24px-grid icon drawn at 16px scales its stroke by 16/24, so `stroke-width="2"` paints about 1.33px. Fix with Lucide's `absoluteStrokeWidth` or `vector-effect: non-scaling-stroke`. One stroke weight per set. If the library has no stroke variants, keep its native stroke and use size or color for emphasis.

### One SVG, recolored per state

```css
.icon-btn { color: var(--icon-muted); }
@media (hover: hover) and (pointer: fine) { .icon-btn:hover { color: var(--icon-strong); } }
.icon-btn[aria-pressed="true"] { color: var(--icon-active); }
.icon-btn:disabled { opacity: 0.4; }
```

- Outline is the default (toolbars, list rows, inline). Filled means selected or active (active tab, saved bookmark, liked heart), and also for glyphs at 14px or smaller. Outline and fill are never interchangeable.
- An active tab icon swaps instantly. Navigation is high-frequency.

### RTL mirroring

| Mirror under `dir="rtl"` | Never mirror |
|---|---|
| Back/forward arrows, nav chevrons, "send", text alignment, list and indent glyphs, speaker waves | Logos, checkmarks, physical objects (clocks, cups, pencils), media playback controls |

One mechanism per element: `[dir="rtl"] .icon-directional { scale: -1 1; }` or Tailwind `rtl:-scale-x-100`. Analyze composite icons part by part.

### Icon crossfade

For infrequent state swaps only: copy → copied, play ↔ pause, like → liked, loading → success, a contextual toolbar appearing. Values are exact: scale **0.25 → 1**, opacity **0 → 1**, blur **4px → 0**.

Do **not** animate: hover-revealed row actions (tens a day, frequency gate: instant or a ≤150ms opacity fade), tab or nav icon swaps, decorative or always-visible icons.

Motion (use `motion/react` if `motion` is installed, `framer-motion` if that is; follow neighboring imports if both; never add a dependency for this):

```tsx
import { AnimatePresence, motion } from "motion/react";

<AnimatePresence initial={false} mode="popLayout">
  <motion.span
    key={copied ? "copied" : "copy"}
    initial={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
    animate={{ opacity: 1, scale: 1, filter: "blur(0px)" }}
    exit={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
    transition={{ type: "spring", duration: 0.3, bounce: 0 }}
  >
    {copied ? <CheckIcon aria-hidden /> : <CopyIcon aria-hidden />}
  </motion.span>
</AnimatePresence>
```

No library: both icons stay in the DOM in one grid cell.

```tsx
const swap = "[grid-area:1/1] transition-[opacity,filter,scale] duration-300 ease-[cubic-bezier(0.2,0,0,1)] motion-reduce:scale-100 motion-reduce:blur-[0px]";
const on = "scale-100 opacity-100 blur-[0px]";
const off = "scale-[0.25] opacity-0 blur-[4px]";

<button aria-label={copied ? "Copied" : "Copy"} className="grid place-items-center size-8">
  <CopyIcon aria-hidden className={cn(swap, copied ? off : on)} />
  <CheckIcon aria-hidden className={cn(swap, copied ? on : off)} />
</button>
```

- `initial={false}` stops the swap from playing on first render. Never put it around an intentional entrance (a staggered hero). Check on a full refresh.
- Reduced motion keeps the opacity crossfade and drops scale and blur. With `<MotionConfig reducedMotion="user">` blur still runs, so branch on `useReducedMotion()` for `filter`.
- The `aria-label` change is the static cue; the animation is never the only signal.
- Tailwind 4 `scale-*` writes the `scale` property; Tailwind 3 writes `transform`. Transition the property the class actually sets.

## 5. Press feedback

One value across the app: `scale(0.97)`, 150ms, `--ease-out`, never on disabled controls.

```css
.btn { transition: scale 150ms var(--ease-out); }
.btn:not(:disabled):active { scale: 0.97; }
```

```html
<!-- Tailwind 4 -->
<button class="transition-[scale] duration-150 ease-(--ease-out) enabled:active:scale-[0.97]">
<!-- Tailwind 3 -->
<button class="transition-transform duration-150 ease-[cubic-bezier(0.23,1,0.32,1)] enabled:active:scale-[0.97]">
```

```tsx
<motion.button
  whileTap={disabled ? undefined : { scale: 0.97 }}
  transition={{ duration: 0.15, ease: [0.23, 1, 0.32, 1] }}
/>
```

- Give the shared Button a `static` prop (or variant) that drops the scale for controls where it would look wrong (full-width rows, large cards).
- Links styled as buttons have no `:disabled`: gate with `[aria-disabled="true"]` instead.
- Any other press value in the codebase (`active:scale-95`, `scale(0.9)`, `whileTap={{ scale: 0.96 }}`) is a finding.

## 6. Type rendering

```css
html { -webkit-font-smoothing: antialiased; -moz-osx-font-smoothing: grayscale; } /* root only, never per element */
h1, h2, h3, h4 { text-wrap: balance; }
p, li, figcaption, blockquote { text-wrap: pretty; }
.prose { max-width: 65ch; }
.display { letter-spacing: -0.02em; line-height: 1.05; }
.live-number { font-variant-numeric: tabular-nums; }
.badge, .label { white-space: nowrap; }
.url, .id { overflow-wrap: break-word; }
```

Tailwind: `antialiased` on `<html>`, `text-balance`, `text-pretty`, `max-w-[65ch]`, `tracking-[-0.02em]`, `tabular-nums`, `whitespace-nowrap`, `break-words`.

| Rule | Detail |
|---|---|
| `text-wrap: balance` | Headings and titles. Only works up to 6 lines (Chromium) or 10 (Firefox); on longer text it is silently ignored. |
| `text-wrap: pretty` | Paragraphs, descriptions, captions, list items, card text. Neither value on long-form (10+ lines), code or `pre`. |
| `tabular-nums` | Counters, timers, live prices, numeric table columns, animated numbers, dashboards. Not static display numbers, phone numbers, zip codes or version strings. Inter's `1` becomes wider and centered: check it. |
| Tracking | Display sizes −0.02em. Body at default tracking. No tracked all-caps labels (Floor). |
| Line height | Unitless. Display 1.0–1.15, body 1.5–1.65, anything wrapping to 3+ lines ≥1.4. Never `line-height: 24px`. |
| Measure | Body 45–75ch, default `65ch` (about 560–680px at 16px). |
| Weight | Below 18px stay at ≥400. Weights under 300 only at 28px+. |
| Truncation | One line: `overflow: hidden; text-overflow: ellipsis; white-space: nowrap`. Multi-line: `line-clamp`. The full value stays reachable (tooltip or expanded view). |
| Inputs | 16px on mobile (`text-base sm:text-sm`). Never `maximum-scale=1`. |
| Families | Font choice belongs to the project or `direction.md`. Rendering rules never override the chosen face. |

## 7. Spacing audit

Run it when spacing "looks inconsistent", before handoff, or when asked to check or standardize spacing. If the project has its own spacing scale (tokens, Tailwind theme), audit against that. The house scale applies when it has none.

### The scale

House scale (SKILL.md §4): **0 · 2 · 4 · 8 · 12 · 16 · 24 · 32 · 40 · 48 · 64 · 80 · 96 · 120 · 144**.

| Role | Steps | Used for |
|---|---|---|
| Hairline / optical | 2 | Hairlines and optical offsets only. Never a layout gap. |
| Fine (inside a component) | 4, 8, 12 | Icon-to-label, chip padding, tight stacks |
| Structural | 8, 16, 24, 32, 48, 64 | Card padding, section gaps, layout margins |
| Page rhythm | 40, 80, 96, 120, 144 | Section spacing on pages |

**20 is not on the house scale.** spacing-audit lists 20 as a fine step but omits it from its own allowed values; the house resolves that by excluding it. 56 is excluded too. Snap 20 to 16 or 24, and 56 to 48 or 64.

Tailwind (4px base): `0.5`=2 · `1`=4 · `2`=8 · `3`=12 · `4`=16 · `6`=24 · `8`=32 · `10`=40 · `12`=48 · `16`=64 · `20`=80 · `24`=96 · `30`=120 · `36`=144. Off-scale: `1.5` `2.5` `3.5` `5` `7` `9` `11` `14` and any arbitrary `[Npx]` not on the scale.

### Grading

| Grade | Condition | Example |
|---|---|---|
| **Error** | Value not on the scale (includes every non-multiple of 4, and 20, 28, 36, 44, 56, 72, 88) | `gap: 10px`, `p-5`, `mt-[18px]` |
| **Error** | 2px used as a layout gap | `gap-0.5` between cards |
| **Warning** | On the scale, a multiple of 4 but not 8, used at layout level (section gap, card padding, layout margin) | section `gap-3` (12px): "consider an 8pt step" |
| OK | Fine steps (4, 12) inside a component | icon-to-label `gap-1` |

### What to check, what to leave

Check padding (all four sides), gap / item spacing, gaps between sections and grouped blocks, and margins around content. Convert rem to px at 16px.

Leave untouched, and list with the reason:
- Values bound to a token or variable (note it, never overwrite).
- 1px borders and hairlines.
- Optical-alignment offsets (`pe-3.5` beside an icon, a 2px play nudge, `translate-x-px`).
- Hug/fill-driven or computed values.
- Hit-area pseudo-element sizes (`after:size-11` is a size, not spacing).

### Snapping rules

1. Snap each error to the nearest allowed step: 6→4/8, 10→8/12, 14→12/16, 18→16, 20→16/24, 28→24/32, 36→32/40, 44→40/48, 56→48/64, 72→64/80.
2. Exactly halfway: round toward the surrounding rhythm of that group.
3. Consistency within a group beats per-element correctness: three cards at 14, 16, 16 all become 16.
4. Fine gaps (icon-to-label and similar) round to 4 or 8, never to a larger step.
5. A whole group uniformly off-grid on purpose (a deliberate 6pt system): flag it once and ask before mass-changing.
6. Prefer `gap-*` over `space-x-*` / `space-y-*` when touching a flex or grid container.

### Procedure

1. Inventory. In code, start with:
   ```bash
   rg -n '\b-?(p|m)[trblxyse]?-(\[[^\]]+\]|[0-9.]+)|\b(gap|space)-([xy]-)?(\[[^\]]+\]|[0-9.]+)' src
   rg -n '(padding|margin|gap|row-gap|column-gap)[a-z-]*\s*:' --glob '*.{css,scss,module.css}' src
   ```
   In Figma, walk the selected frame top-down (whole page if nothing is selected).
2. Classify each value: leave (token, hairline, optical, computed), error, warning or OK.
3. **Report before you apply.** Summary first: total values checked; every change as `before → after`, grouped by component or frame (`Card padding: 14 → 16`); untouched values with the reason.
4. Apply only after the summary, and in the same turn only if the user asked for fixes.
5. End with a one-line rhythm note: "Layout now on an 8pt rhythm, fine spacing on 4pt."

Never resize elements or reflow content to hit a number, never invent a new scale, and never touch radius, stroke or type during a spacing audit unless asked.

## 8. Layout grouping

- **Group with space first**, a background shape second, a separator line last. A separator where spacing would do: remove it and double the gap.
- **Between groups ≥ 2× within a group.** 8px inside a group means 16px or more between groups. Space above a heading ≥ 2× the space below it.
- One spacing step per level of subordination; 16px is a good default step.
- Without a density system, start at 12px between adjacent bordered or filled controls and 24px clearance around borderless text or icon-only controls. Compact tools may go tighter if hit areas do not overlap.
- Controls look like controls: a background shape, a border or a consistent placement zone. Never style a control like the static text next to it.
- Align to shared edges. Use logical properties (`padding-inline-start`, `margin-inline-end`, Tailwind `ps-*`/`pe-*`/`ms-*`/`me-*`). Physical left/right only for physical geometry.
- Order by importance: most important near the top and on the leading edge.
- Plan for growth: no fixed widths or heights on text containers, rows wrap, critical actions never sit where resizing or scrolling clips them. Overflow rules are in the Floor.
- Breakpoints come from content (house stack point 60rem, single column 40rem), never 768/1024 by habit. Prefer container queries for components.

## 9. Rendering gotchas

### Suppress transitions on theme switch

Color transitions fire across the whole tree when the theme flips, so the page smears. Kill transitions for one frame:

```ts
export function withoutTransitions(apply: () => void) {
  const style = document.createElement("style");
  style.textContent = "*,*::before,*::after{transition:none!important}";
  document.head.appendChild(style);
  apply();
  void document.body.offsetHeight; // force a synchronous style flush
  requestAnimationFrame(() => requestAnimationFrame(() => style.remove()));
}

matchMedia("(prefers-color-scheme: dark)")
  .addEventListener("change", () => withoutTransitions(syncThemeFromSystem));
// The in-app toggle calls withoutTransitions(() => setTheme(next)) too.
```

With `next-themes`: `<ThemeProvider disableTransitionOnChange>`.

### `will-change` makes a containing block

- Add it only after you see a first-frame stutter, and name the property (`will-change: scale` when animating `scale`). Worth it for `transform`/`translate`/`scale`/`rotate`, `opacity`, `filter`; `clip-path` rarely. Never `all`, never `top`/`left`/`width`/`height`/`background`/`border`/`color`.
- **Gotcha:** `will-change: transform` or `filter` (like `transform`, `filter` and `contain: paint` themselves) makes the element the containing block for `position: fixed` descendants and creates a stacking context. A fixed modal or tooltip inside it stops tracking the viewport and can sink under siblings.
- Fix: portal overlays to `body`, remove `will-change` when the animation ends, or move it to an element with no fixed descendants.

### Contain scroll in overlays

Dialogs, drawers, menus, popovers and side panels with `overflow: auto` get `overscroll-behavior: contain` (Tailwind `overscroll-contain`) so scrolling to the end does not scroll the page behind.

### Hover gating

All hover styles sit inside `@media (hover: hover) and (pointer: fine)`. Tailwind 4's built-in `hover:` checks only `(hover: hover)`; Tailwind 3's checks nothing. Redefine it once:

```css
@custom-variant hover {
  @media (hover: hover) and (pointer: fine) {
    &:hover { @slot; }
  }
}
```

Where a control draws its own pressed state, add `-webkit-tap-highlight-color: transparent`.

## 10. Small details

| Symptom | Fix |
|---|---|
| "340 days ago" | Relative time under 7 days, absolute date after. Full timestamp in `<time dateTime title>`. |
| "1 members", "You have 1 new messages" | `Intl.PluralRules` selecting a full message per category. Never concatenate fragments around a variable. |
| Gradient fade shows a visible edge | Eased gradient: extra stops along an ease curve, not two stops. |
| Avatar squished to an oval | `flex-shrink: 0` on the avatar (Floor), `min-width: 0` on the text sibling. |

```ts
const rtf = new Intl.RelativeTimeFormat(locale, { numeric: "auto" });
const abs = new Intl.DateTimeFormat(locale, { dateStyle: "medium" });
export function when(date: Date, now = new Date()) {
  const s = (date.getTime() - now.getTime()) / 1000;
  const a = Math.abs(s);
  if (a >= 7 * 86400) return abs.format(date);
  if (a >= 86400) return rtf.format(Math.round(s / 86400), "day");
  if (a >= 3600) return rtf.format(Math.round(s / 3600), "hour");
  if (a >= 60) return rtf.format(Math.round(s / 60), "minute");
  return rtf.format(0, "second"); // "now"
}

const plural = new Intl.PluralRules(locale);
const members = { one: (n: string) => `${n} member`, other: (n: string) => `${n} members` };
export const memberCount = (n: number) =>
  (members[plural.select(n) as "one" | "other"] ?? members.other)(new Intl.NumberFormat(locale).format(n));
```

With an i18n library, use its ICU plural form instead: `{count, plural, one {# member} other {# members}}`.

```css
/* Eased fade-out edge on a scroll area (mask alpha follows an ease-in-out curve) */
.fade-bottom {
  mask-image: linear-gradient(to bottom,
    oklch(0 0 0 / 1) 0%, oklch(0 0 0 / 0.951) 15.5%, oklch(0 0 0 / 0.825) 29%,
    oklch(0 0 0 / 0.648) 41.2%, oklch(0 0 0 / 0.45) 52.9%, oklch(0 0 0 / 0.259) 64.7%,
    oklch(0 0 0 / 0.104) 77.5%, oklch(0 0 0 / 0.013) 91.9%, oklch(0 0 0 / 0) 100%);
}
.avatar { flex-shrink: 0; width: 32px; aspect-ratio: 1; border-radius: 9999px; object-fit: cover; }
```

For a color scrim instead of a mask, use the same stops with `oklch(from var(--surface) l c h / <alpha>)`.

## 11. Before you finish

Grep for each detection, apply the fix, then verify by looking (SKILL.md §2 step 6).

| Detection | Fix |
|---|---|
| Same radius on a padded parent and its child | Outer = inner + padding (+ border) |
| `0 0 0 1px` ring shadow with no border | Add `border: 1px solid transparent` |
| Border plus wide shadow on one element | Pick one: the elevation token, or the border |
| `transition: box-shadow` / `transition-[box-shadow]` | Hover shadow on a pseudo-element, fade its opacity |
| `outline-slate-*`, `outline-zinc-*`, hex outline on `img` | `outline-black/10 dark:outline-white/10`, offset −1px |
| `transition: all`, `transition-all` | Name the properties: `transition-[scale,opacity]` |
| `animation:` on `:hover`, `.open` or a toggled class | Use a transition (interruptible) |
| `active:scale-95`, `scale(0.9)`, `whileTap` ≠ 0.97 | `scale(0.97)` 150ms `--ease-out` |
| `:active` scale without a disabled guard | `:not(:disabled):active` / `enabled:active:` |
| `bounce:` > 0 outside a momentum gesture or rare delight | `bounce: 0` (house springs) |
| Base rule `opacity: 0` revealed only by an animation | Hidden state in the `from` keyframe + `animation-fill-mode: both` |
| `translate`/`scale`/`filter`/`motion.*` without a reduced-motion path | Gate it; keep the opacity crossfade |
| Color transitions + a theme toggle with no suppression | `withoutTransitions()` / `disableTransitionOnChange` |
| `will-change: all` or on a non-animating element; a `will-change`/`transform` ancestor over a `position: fixed` child | Name the property or delete it; portal the overlay to `body` |
| Hex `fill`/`stroke` inside an icon | `currentColor` |
| Mixed icon libraries or stroke widths on one surface | One library, stroke per §4 table |
| Directional icon not mirrored in RTL (or a play icon that is) | `rtl:-scale-x-100` on directional only |
| Icon crossfade on hover-revealed actions or tab icons | Remove: instant swap |
| Ungated `:hover`, Tailwind 3 `hover:`, Tailwind 4 default `hover:` | `(hover: hover) and (pointer: fine)` variant |
| `overflow: auto` in dialog/drawer/menu without `overscroll-behavior` | `overscroll-behavior: contain` |
| Changing numbers in proportional digits; no `antialiased` on the root | `tabular-nums`; `antialiased` on `<html>` |
| No `text-balance` on headings / `text-pretty` on short body; display at default tracking; paragraphs past 75ch | Add them; `-0.02em`; `max-width: 65ch` |
| Off-scale padding, margin or gap | Spacing audit (§7) |
| Separator line where spacing would group | Remove the line, double the gap |
| `margin-left`/`padding-right` in localizable UI | Logical properties |
| Avatar/icon in a flex row without `flex-shrink: 0`; "N days ago" past a week; string-built plurals; two-stop fades | §10 fixes |
