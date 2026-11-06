# Showcase: faux product UI, demo loops, bento grids

Source: Stageshow by Benji Taylor (benji.org/stageshow, `benjitaylor/skills`, MIT), adapted to craft house values.

**Contents:** 0 Kit and tooling · 1 The one idea · 2 Five failures · 3 Workflow · 4 Craft numbers · 5 Framing · 6 Motion and story · 7 Cursor, drag, scroll, phones · 8 Bento grids · 9 Annotations · 10 Anti-slop style pairs · 11 Verify · 12 Fresh-eye review · 13 REHEARSAL.md

The Floor (SKILL.md §3) applies to everything here. The house values (SKILL.md §4) apply to every piece of **UI feedback inside a demo** (press, hover, popover, menu, toast).

**Deliberate exception, storytelling pace.** A showcase is an illustration, not a product surface. Its storytelling beats may run slower than the UI caps: arrivals from nowhere 0.8–1s, payoff holds 2s, loops 10–16s, in-scene reading-order stagger 40–80ms, and the storytelling curves in §6. Only the story slows. A button pressed by the demo cursor still responds with `scale(0.97)` 150ms `--ease-out`, a popover still opens in 200ms, a hover still changes color in 150ms.

---

## 0. Kit and tooling

Check first whether the Stageshow skill is installed next to this one, in your agent's skills folder (for example `~/.claude/skills/stageshow` or `~/.agents/skills/stageshow`).

**If it exists (full kit, gold examples, film scripts):**

| Step | Do |
|---|---|
| Serve gold | `cd <skills-dir>/stageshow && npm install && npm run dev` (Vite playground; the dev script prints the preview URL) |
| Film a gold still | `node scripts/film.mjs "<preview>/?capture=<gold>" --select '#example' --still --out gold.png` |
| Copy the kit | Copy `kit/` into the project (for example `src/showcase/kit/`) and import from `kit/index.ts`. React 18+, `lucide-react` for `Icon`. Never hand-draw a window, device or bubble the kit already has. |
| Copy the gold | Copy the closest `examples/<gold>/` folder beside your component, fix the kit import path, keep its structure. |
| Quick loop | Optionally copy into `examples/<yours>/` inside the stageshow folder so `?capture=<yours>` films it, then move it into the host when it passes and delete the copy. |
| Verify | `scripts/film.mjs` (§11), `scripts/layout.mjs` for host widths, `scripts/jumps.mjs`, `scripts/class-clash.mjs` for unscoped class names. |
| Teaching pairs | `teaching/*.tsx` and the playground `?example=contrasts`: bad/good pairs. Bad halves are never starting points. |

**If it does not exist:** build with plain React and CSS following every rule in this file (stage with `overflow: hidden`, measured targets, mounted exits), or offer to install it: `npx skills add benjitaylor/skills --skill stageshow`.

Kit inventory (names to look for in `kit/index.ts`):
- **Stage:** design size, scale-to-fit, `compact`, scene state, playback, look, cursor, `poster`; the script receives `Act`. **Act:** `set`, `wait`, `move`, `click`, `type` (35ms/char), `key` (shows keycaps), `drag`, `scroll`, `caret`, `snap`, `stagger(items, fn, gap=60)`, `mark(name)`, `rect(name)`. Start a `move` without awaiting to run it alongside other beats.
- **Frames:** `Window`, `Phone` + `LockScreen` + `Notification`, `Keyboard`, `Thread` + `Bubble`, `Stamp`, `Receipt`. **Motion:** `Collapse`, `Roll`, `Swap`, `Count`, `Connector`, `Fly`, `Follow`, `States`, `Stream`, `Popover`, `Scroll`, `Camera`, `Expand`, `Slot`.
- **Parts:** `Bar`, `Text`, `Row`, `Avatar`, `Pin`, `Icon`, `Menu`, `Field`, `Segmented`, `Toggle`, `Tabs`, `Badge`, `Chip`, `BrandMark`, `Code`, `Chart` (restyle, never reshape geometry).
- CSS lives in `@layer stageshow`. Restyle with a plain class, never `!important`. `a.click()` draws a press and `a.set()` changes the scene; this is authored illustration, not browser automation.

Script shape:
```ts
await a.click({ at: "share" });
await a.set({ open: true });
await a.wait(2000);              // payoff hold
await a.click({ at: "close" });  // visible cause for the reset
await a.set({ open: false });
await a.move({ x: 600, y: 300 }); // back where it began
```

---

## 1. The one idea

> It reads as a real product, simplified to what the viewer needs, framed with intent, and every motion has a cause.

Simplify the words, not the context. Ask for something small: one interaction, one product, one size ("a small looping demo for the homepage hero: someone assigns an issue to a teammate"). Expect a few feedback rounds.

The bar is the gold examples. Picture the exacting designer who screenshots the one wrong thing and asks "what is this?": a dot off its avatar, a unit on its own line, a grabber under the island, copy touching a card edge.

---

## 2. Five failures

| Failure | Looks like | Fix |
|---|---|---|
| **Too much** | Real text everywhere, packed rows; reads as generated | Real words only on the story (3–7 strings), placeholder bars for the rest |
| **Wireframe** | Two gray blobs and a button | A full, structured screen in quiet grays with product anatomy |
| **Boxed** | Whole window shrunk into a card with even margins; a pasted screenshot | Crop to the panel the story happens in; bleed off an edge |
| **Flat** | Everything the same weight, every group in the same bordered card | One leading object; heaviest marks on the story; quieten the rest |
| **Bland** | Gray window plus cursor whatever the brief's look or energy | Map the product's tokens; a styled brief (kawaii, paper, poster color) gets its style in full |

Not calm, unfinished: an empty corner, a void under a list, rows spread to fill height, a window floating on a plate.

---

## 3. Workflow

Plan in text first: where it sits, the focal object, the action, the payoff, what to leave out.

### 3.1 Medium and size
Still, or animation (loop / once when in view / on hover). Design at the shown size, shown 1:1.

| Placement | Design size |
|---|---|
| Hero beside copy | ~720×340 |
| Full-width hero | 900–1100 wide |
| Feature card | ~400×300 |
| Lone card in a 680px column | 500–560 wide, so text reads 15–20px |
| Minimum for any demo | ~320 wide (bento tiles and satellites may be smaller) |
| Phone column | a separate `compact` design at 358 in a 390 viewport, e.g. `compact={{ below: 560, width: 358, height: 320 }}`. Redesign, never squeeze. |

### 3.2 Look and token mapping
Pick theme (light / dark / blueprint / glass), palette (accent / mono) and fidelity (minimal = all bars; standard = words where meaningful; realistic = real content everywhere). Default: light, accent, standard. Use realistic at hero scale, for a named real product, or for a still whose subject is a list or page.

Map the product's tokens into the kit (procedure works for any faux UI):
1. Find tokens: `:root` vars, Tailwind `theme.extend` colors/borderRadius/fontFamily, `theme.ts` or tokens JSON. None? Read computed styles of a real button, input, menu and card.
2. Map:

| Product | Kit token |
|---|---|
| primary | `--ss-accent` (+ `-2`, `-soft`, `-text`) |
| page bg / card bg | `--ss-canvas` / `--ss-surface` (+ `-chrome`, `-well`, `-raised`) |
| text scale | 3 text tokens: strong / text / faint |
| borders | `--ss-line`, `--ss-line-faint` |
| font / mono | `--ss-font` / `--ss-mono` |
| button & input radius / menu item / popover & card | `--ss-radius-md` / `-sm` / `-lg` (kit defaults 8 / 6 / 10, window 12, pill 999) |
| input height / menu row | `--ss-control` (34) / `--ss-row` (26) |
| placeholder inks | `--ss-ink-1..5` (`greys="cool"` if the product's grays are cool) |

3. Write a look file and use it by default: `.look-acme .ss-stage { --ss-accent: …; --ss-font: …; --ss-radius-md: 6px; --ss-control: 32px }` (or `tokens={{…}}`). Values come from the project's token block, never new literals.
4. Check: the still beside a real screenshot at the same scale (accent, gray temperature, radii, type).

A look changes tokens only; keep the product's motion habits. Public site: take values, never assets (no unlicensed fonts or images). The developer wants their own components: import them inside the stage with `data-target` and `rect()`, fixed props, never call live services.

### 3.3 Story
Pick a shape (§6.1), 3–5 beats plus a payoff, 10–16s. No cursor move you can't explain. Data never rewinds in view without a cause. Richer story vs calmer frame: choose the frame.

### 3.4 Start from gold
Film the closest gold (§3.8). Copy its folder; keep stage size, crop and bleed, row and type sizes, spacing, beats and kit parts. Read §4 fully. Replace look and content with the brief's product; a copy that only changes the name is not done. No gold for the story: pick the layout from §5.2 and copy that example, then borrow motion from the example with the closest story. Frame it (§5), film one still at both widths beside the gold's. Emptier, flatter or more boxed than gold: fix the frame before any motion.

### 3.5 Screen
Count real strings (3–7; a budget for words, not elements). Use kit tokens, `Bar`, `Text`, `Row`. Fit every control's text before it moves. Put `data-target` / `data-cursor` on cursor targets. Measure with `rect()`; never hard-code coordinates.

### 3.6 Choreograph
Keep appearing things mounted and toggle them so exits animate. Animate the relationship (a new row carries the rows below; a resizing surface never stretches its text). Beats overlap, groups stagger, the payoff holds, the loop ends where it began or resets with a visible cause.

### 3.7 Rehearse
Set `poster` first (§6.2). Film the loop: contact sheet at shown width and at 390; step every press, menu, morph and the loop seam. Fix the largest visible fault first: framing > hierarchy > motion > polish. Re-film. Compare with gold using §10 and the teaching pairs. `film.mjs --gate` ending "rehearsal passed" is necessary, not sufficient. Then the fresh-eye review (§12) and `REHEARSAL.md` beside the component (§13).

### 3.8 Gold examples (need → example)

| Need | Gold |
|---|---|
| Still / feature card | `static-card` |
| Dark still | `dark-bento` |
| Animated hero (5 beats, named targets, annotation, terminal satellite) | `hero-demo` |
| Choreography without cursor | `payout-flow` |
| Content changing in place | `choreography` |
| AI answer (question, Thinking, streaming with height, sources last) | `ai-chat`; cards `ai-cards` |
| Form validation (error on causing field, cleared by fixing, button confirms on itself) | `form-error` |
| Drag reorder / freeform drag | `reorder` / `freeform-drag` |
| Chart scrub | `chart-scrub` |
| Table edit (total rolls a beat later) | `table-edit` |
| Loading (skeleton at content geometry, rows resolve top to bottom) | `loading` |
| Dark hero | `dark-hero` |
| System story (consequence lands elsewhere) | `subscriptions` |
| Phones | `phone-touch`, `dynamic-island`, `messages`, `device-pair` |
| Cursors | `cursors`, `cursor-morph`, `cursor-resize` |
| Bento | `bento`, `ruled-bento`, `staggered-bento`, `hero-grid` |
| Layout specimens | `close-up`, `many-of-one`, `fade-crop`, `panel-annotation`, `overlap`, `stacked-cards`, `opposite-corners`, `tilted-stack`, `corner-chip`, `monochrome`, `window-row`, `story`, `calendar`, `upload`, `share`, `swipe`, `presence`, `annotations` |
| Product studies (borrow interaction, never identity) | `figma`, `linear`, `slack`, `raycast`, `finder(-glass)`, `spotify`, `notion`, `google-docs`, `gmail`, `google-calendar`, `powerpoint`, `excel`, `wikipedia`, `telegram`, `imessage` |
| Catalogs, not compositions | `chrome`, `phones`, `controls`, `parts`, `kit-extras`, `glass`, `backgrounds`, `brand-marks`, `looks` |

---

## 4. Craft numbers (the still frame)

**Unit note.** The 4–6px stack, 6px mark clearance and 7px bar gap below are **measured ink distances** (glyph edge to glyph edge), not CSS spacing tokens. Produce them with on-scale CSS (e.g. 1.3 line-height + 2px gap); keep every `gap`/`padding` on the house 4px scale.

### 4.1 Restraint
- Real strings: title, the thing acted on, typed text, the result. 3–7. Everything else is `Bar`.
- Word budget ≠ element budget: a calendar still needs a populated week.
- Where the story touches a list: 2–3 real rows and at most as many bars. Never bars where the eye lands.
- Small surfaces (popover, card, tile): a row is ONE line (avatar, name, value). No gray bar under a real name.
- Label and value match: both words or both bars.
- Named avatars get their own color; gray discs are filler.
- Keep domain-signature content (album art, thumbnails, avatars).
- Output as subject (log, diff, terminal): real lines, short and few. A terminal is a character grid: spinner and tick share one `1ch` cell.
- ONE number gets to be big. Placeholder widths vary (60, 45, 70, 52%).
- Say it once. No masthead or monogram for a fictional name; product name only when the brief leans on it and the real product would show it.
- Judge the silhouette at thumbnail size first. Compose objects, not an inventory of cards. Space follows the story: the payoff gets room, no empty band mid-window. Dots, halos, badges and focus rings sit fully inside the content edge. Nothing outsized.
- Density follows the story; empty space can be hierarchy; enlarge content before adding anatomy.

### 4.2 Type
- Fit the text before fixing the box.
- Faux UI text 11–15px at design size, scaled by the stage; never below 10px as rendered (check at 390). Page copy around the stage follows house type (body ≥16px).
- Small text line-height ≥1.3, with insets on all sides.
- One family (product's or system). Monospace only for code, IDs and aligned numbers.
- Sentence case. Hierarchy from spacing plus one weight step. No real headings: a faux title is a `div`, never `h1–h6` (the demo lives inside someone else's page outline).

### 4.3 Spacing and alignment

| Rule | Value |
|---|---|
| Words inset from window / card / panel / hovered-row edge | **12px** (film flags <8px as `tight inset`) |
| Control clearance: button from field, line or card edge above | **≥8px** (`tight control`) |
| Title over subtitle, ink between lines | **4–6px** (<3 `tight stack`, >10 `loose stack`) |
| Dot / badge / ring clearance from neighbors | **6px** clear (within 5px = `tight mark`) |
| Row padding | 12px (8px dense). A one-line row 3× taller than its words = `fat row` |
| Bars in the same line box as replaced text | 6px and 5px bars want 7px between |
| Text to neighbor | fixed gap 8–12px, never leftover `space-between` |
| **No near-tangents** | Edges 1–3px apart read as accident: align exactly or separate by ≥ the gutter, never <8px |
| First heading below a toolbar rule | at least the side inset |

- Bars may bleed; words never.
- Center a label on its letters: `text-box: trim-both cap alphabetic`.
- Nested shapes pad evenly; outer radius = inner + padding (Floor). Carry the inset round the bottom.
- Whitespace groups: a title sits closer to its metadata than to the next record. Align the reading edge, then the visible ink. Center on the visible face (undo tracking with negative `margin-right`).
- A popup under a field draws its ring inset so the sides align.
- Controls hug content; never `flex: 1` with content at one end (`lopsided`).
- Fix overflow by fitting content, not widening. Columns share one grid template with fixed tracks. A dot, badge or ring is not a column: put it in the gutter or on the avatar, 6px clear. An unread dot hangs in the margin so the row never moves.
- Centered amount: center the digits; the currency sign is absolutely placed at `right: 100%`. A rolling amount rolls only digits.
- Links get no hover surface; controls have a visible surface at rest. Hover and selection fills are inset and rounded (negative margin on the list, not the row).
- Clone real menus from computed styles. A trigger with an open popover keeps its pressed look (`data-open`).

### 4.4 Icons and logos
- Imagery earns its place; never invented illustration.
- Icons from a set, never hand-drawn. Order: the product's own pack; Apple context → kit `Icon` filled / SF-style; Android → Material Symbols; else Phosphor, Iconoir, Radix, Tabler, Heroicons. Lucide deliberately, never reflexively.
- One family, one stroke weight (house: 1.5 at 400, 2 at 500–600), one size 14–16px per scene.
- **Small icons filled at ≤14px** and inside tinted tiles (outlines go wiry). Matches house rule.
- Even glyphs in even boxes (odd in even snaps = `half-pixel glyph`). Fixed icon boxes `flex: none`.
- Icons are one opaque color; fade via SVG `opacity`; no duotone. Never mix filled and outline in one scene (`mixed icons`).
- Emoji are content (a reaction), never icons.
- Other brands via `BrandMark` in original proportions and colors. Real company name + fake initial tile = never (`invented mark`). Fictional product: neutral initial tile or plain-text wordmark, never a letter in a colored square.
- Play triangle shifted toward its point by 1/6 of its width.
- AI features: no sparkle, wand or star, no purple as the AI color. Show what the AI does. The product's own sparkle icon is fine.

### 4.5 Color and looks
- One neutral family; five inks plus one accent to start. Status colors mean status.
- No redundant tonal borders: try fill alone first. Quiet is not faint. Gradient or glow serves the story or goes.
- Hero is never outlined with an accent ring plus halo (reads as a phantom focus ring). Lift with surface, raised shadow or an accent label.
- A filled button keeps its color on hover (one step of its own color, never a gray swap).
- Mono palette: mark the acted-on thing with a solid step in surface and weight, not a translucent wash.
- **Nothing translucent overlaps** (facepiles, stacked chips): opaque, or it reads as `onion skin`.
- Dark themes: depth from a lighter surface, not shadow (satellites on `--ss-raised`). A light doc over a dark workspace stays readable light.
- Light inks are tuned for white cards on a gray canvas; step up a tone on pure white. Status colors never pass through mud.
- Build once, film every look: a light pass does not prove dark.

### 4.6 Chrome and devices
- Never draw platform chrome from memory: use the kit or a real capture (Floor). Window chrome 38px. `barSize="tall"` (64px, 32px controls) only for a real icon+label toolbar.
- Browser chrome only when the browser is part of the picture. Quiet browser windows center content and use a rounded, centered URL pill.
- Columns run the full window height. `sidebar` holds the traffic lights (default width 180, content heading at 16px inset).
- macOS 26 Finder: solid white window, only the floating sidebar is glass (`sidebarStyle="floating"`: inset 8px, 24-radius window), gray glyphs, selected row on a gray pill. `lightsSize="large"`: 12px lights, 8px apart.
- iPhone messaging keeps Messages conventions: blue sent, gray received, plus + mic, reactions on the bubble.
- Phones from the kit; never shrink the iOS clock. Status bar and home indicator come from `Phone`, never hand-drawn. Nothing under the island. Phone lit from the first frame.
- Notification: one bold title over the message, real app icon, true iOS size, banner width = screen minus 18pt each side, body wraps to 3 lines max.
- Phone background has a job: `LockScreen` for notification and island stories; never invented home-screen icons.
- A phone may straddle a window but never cover the amount it explains.
- Several windows: pick the active one; push the others back with quieter contrast and shadow. Platform conventions (Messages blue, traffic lights, iOS banner size) are chrome, not placeholders.

### 4.7 Page of demos
One figure language per page: one ground, one gutter, one `--ss-shadow-hero`, one radius set. Fewer, stronger demos. A comparison pair shows a different demo, not the same one twice.

Recipes: scope CSS under the demo root class (`@keyframes` outside). Host prose styles exclude the stage: `:not(:where(.ss-stage *))`. Check a fresh load at each width and a resize between.

---

## 5. Framing

### 5.1 Rules
1. One hero, at most 1–2 satellites (receipt, phone, detail card).
2. Keep what identifies the product: title, real chrome, rows that give the action a reason. Logo and nav optional.
3. The medium picks the first layout: corner bleed for a hero beside copy, pinned bottom bleed for a card or tile, phone pinned from the top.
4. **Crop only filler.** Payoff, action, cursor targets and real words stay whole. A list, diff or terminal may run one line off a bled bottom. A sliced button, chip or shadow is the usual fault; remove a slice-prone placeholder action instead.
5. Crop 10–40% of the hero off stage. The hero fills ≥2/3 of the stage width and runs off an edge. Shapes readable when cut run off by ≥1/3.
6. The stage masks everything (`overflow: hidden`), one rounded corner per edge. Fades only for sliding tracks, stacks, charts and a window-foot fade.
7. Satellites straddle the hero edge about half outside. Fully outside: a 1px leader ending in a dot.
8. Depth: `--ss-shadow-satellite` on the satellite, `--ss-shadow-ground` on a lone surface, a hairline on light edges. Shadow on the outlined shape, never on a wrapper.
9. Start flat and frontal. Earn tilt. Perspective only if asked.
10. **One gutter** (hero inset from stage edge), measured in the final crop. All edge distances match, except a straddling satellite.
11. Full gutter or well past it on every edge. Crossing by <24px or stopping a few px short is a mistake.
12. A window keeps device shape: whole ≥1.25:1; bled, the visible part ≥ square. Never squash.
13. What the story opens stays in frame, a gutter from every edge, every frame.
14. Colored stage: hero inset 8–20% from the sides, runs off the bottom.
15. No dead corners, at rest or anywhere in the loop.
16. Anchored UI (popover, menu, toast) is not a satellite.
17. Two satellites = two kinds (a card + a small system object), never two same-width cards on one edge (`twin satellites`). A pointing-only second object becomes an annotation with a connector.
18. A satellite covers chrome, not content.
19. Related surfaces get different jobs and weights; the result goes in the hero's quiet margin.
20. A small illustration may need more air.

The page is the ground: "white page" means a transparent stage, no plate. `unframed` only when the developer asks (whole, centered, even margin ≥ shadow depth). Never box the stage in another ground. Moving a window keeps the bleed; recheck the crop. Small formats crop harder: a 400px card shows about a third of a dashboard at full size.

Shadow tokens: `--ss-shadow-hero` (one per page), `-window` (quieter default), `-satellite` (tight; wide blur smears on white), `-ground` (hairline + long faint fall), `-float` (popovers), `-card`, `-lift` (carried thing). These live inside the stage; host page surfaces use house elevation.

Fades: 32px edge mask on tracks (`--ss-fade: 32px`; kit default 64px); last 25–30% of a stack or chart; `.ss-fade-b`, `.ss-fade-r`. Window-foot fade is a mask, not a color overlay; only the foot, only over filler, at most one window per scene.

Phones in frame: a satellite phone starts at 40% of the window's visible width and ≥220px; run it off the bottom rather than shrink it. Draw at ≥340 wide (~0.85 true size) when it shows app text. A top-cropped phone loses status bar and nav together. In a 400px host the phone is cropped (pinned a gutter from one edge, off the other, screen 1:1, stage a bit taller than wide), never whole.

### 5.2 Layout recipes

| Layout | Hero / satellite geometry | Use | Example |
|---|---|---|---|
| Pinned bottom bleed | Inset 8–12% left/right/top, 15–35% past bottom | Feature cards, bento | `static-card` |
| Corner bleed | Inset 10% top+left, 15–35% past right+bottom; satellite straddles left or bottom | Hero beside copy | `hero-demo` |
| Stage with gutters | 8–20% each side on a colored stage, off bottom | Brand moments | `raycast` |
| Satellite focus | Hero cropped hard; satellite 30–40% of hero width carries the story | Detail-led stories | `share` |
| Wide hero | 900–1100px window at 1:1, off bottom/sides, realistic fidelity | Full-width hero | `linear` |
| Row of windows | 3–4 same height, outer ones off both sides | Range of surfaces | `window-row` |
| Device pair | Desktop inset top/left ~75–80% wide off bottom; phone straddles right edge ~1/3 outside, top in upper half, present from frame one | Cross-device | `device-pair` |
| Many of one | Grid of one thing off every edge; one tile in focus by depth, only it has words | Scale, collections | `many-of-one` |
| Fade crop | Window inset top/sides, fading over the last third | Long content | `fade-crop` |
| Close-up | Whole window 1.3–1.6× incl. chrome, off bottom | Small detail at reading size | `close-up` |
| Panel over product | Panel straddles product's right edge + one neutral leader to what it changed | Settings → effect | `panel-annotation` |
| Overlapping windows | Smaller chromeless card in front, edge on the window's sidebar line, covers only filler | Two related views | `overlap` |
| Stacked cards | Two components, same width and left edge, 12px apart, lower one off bottom | Component pair | `stacked-cards` |
| Opposite corners | Copy bottom-left, product panel hung off top-right | Text-led tiles | `opposite-corners` |
| Tilted stack | 3 same-size cards at −6°, 4°, −2°, offset 12–20px; straightest on top, only it has words | Collections | `tilted-stack` |
| Corner chip | Pill centered on the window corner, half outside, over chrome | Status, badge | `corner-chip` |
| Monochrome row | 3 windows grayscale, faded foot | Quiet range | `monochrome` |

---

## 6. Motion and story

### 6.1 Story shapes
- **How it feels to use:** the cursor does one satisfying thing; the result lands under its hand.
- **A flow in an app:** a few screens as one continuous object, payoff last. No bolted-on second device.
- **What the system does:** the consequence lands elsewhere; the link is visible (connector, card flying into a list). Usually no cursor.

Beauty decides: calmer, fuller, better framed beats more story.

### 6.2 Opening, poster, payoff
- Open on a finished screen that works as a still (a thread opens with a real message, not bars).
- Short history sits at the TOP of a pane, never pinned to the bottom over a void.
- `poster` = the frame most people see (thumbnail, reduced motion, screenshot). The beat that tells the story alone, never mid-change (no digit mid-roll, no label mid-morph), result left on screen. Test: cover the motion and ask what happened.
- Payoff visible at shown size (a 16px avatar at the edge is not a payoff) and arrives FULL (put the first lines in before `Collapse` opens).
- A satellite arrives at the beat it pays off, carrying its payload, or is present from frame one. Never parked empty.
- A connector draws over ~1s after both ends are placed, routes around cards, ends on a dot.

### 6.3 Loops
- Hero loop 10–16s, 3–5 beats to the payoff. **Payoff holds 2s.** Nothing sits still >3s. No dead air at the seam: the next action starts within ~1s.
- The reset removes what the loop added as ONE thing (reply and reaction leave together); the loop keeps a complete product picture.
- Every reset has a cause or is invisible, in this order of preference: a seen cause (undo with keycaps, Clear, Close) > exit around an anchor > snap while hidden (`act.snap`).
- Exit around an anchor (illustration reset only) = one reverse stagger, 40–80ms apart, everything gone within 0.5s, never all at once. Product UI exits stay simultaneous and fast (house values).
- Never play navigation backwards: leave the way the product leaves, or cut while covered. Data never rewinds in view.
- An accumulating story is a conveyor. A second scene must add something new. A looping video starts on a calm frame. Film the seam at 60ms intervals.

### 6.4 Pace and easing (storytelling only; see the exception at the top)

| Job | Curve / timing |
|---|---|
| Travel across the stage | `cubic-bezier(0.45, 0, 0.1, 1)`: departure, clear travel, soft arrival |
| Arrival from nowhere | `cubic-bezier(0.25, 0.8, 0.25, 1)` over 0.8–1s |
| Phone in-app response (UIKit-quick) | `cubic-bezier(0.3, 0.75, 0.1, 1)` over 0.5s |
| Page thrown | `cubic-bezier(0.25, 1, 0.5, 1)` over 0.5s |
| Grows while traveling (`Fly`) | ≥600ms with a long settle; a quick ease-in-out lurches |
| Distant-hue color change | ~200ms (passes through mud if slower) |
| In-scene reading-order stagger | 40–80ms (`act.stagger` default 60) |
| Grid tiles stagger | 80–120ms (house marketing stagger) |
| Page swap inside one window | keep chrome, crossfade body 200ms with an 8px rise; never slide whole pages sideways |
| **UI feedback inside the demo** | **House values**: press `scale(0.97)` 150ms `--ease-out`, popover 200ms, hover 150ms `ease` |

- Judge distance and overlap, not duration. Durations and easings by job; one easing everywhere flattens hierarchy.
- Feedback starts on input. Related responses overlap: the setting leads, the label follows, then dependents; confirmation follows its cause.
- A small expressive icon may overshoot locally (delight); cards and text settle without bounce.
- Holding a result means stop: no pulse, no recount. One object enters whole.

### 6.5 Changing text and values
- **Every change moves**: a submenu slides, text rolls, a surface morphs. Never a one-frame replacement.
- Change only what changed: `Roll` animates the changed span and moves the rest by measured width; never roll unchanged words.
- Decide a control's footprint before its label transition: `States` holds the largest; `Roll variant="fade"` resizes. Button states share one slot (one grid cell); check shortest, longest and pending labels.
- Right-pinned values change against that edge (`Roll align="end"`). Pin a changing label to its icon (it grows away from the chevron).
- Numbers roll with `Count`: tabular digits, last digit first, unchanged digits still, no per-frame lerp blur, stable slot, unit in its own box; hold the completed value. A total changes a beat after its edit (`delay`).
- Typed text becomes a chip in place. The result follows press feedback (state changes after the click completes).
- An object keeps identity: plus → close rotates the same strokes.
- A spinner turns before it is visible and through its fade-out. `Check`: pending arc → on (one mark, never a spinner stacked over a check).
- Settle geometry before chasing text shift; watch for glyph shimmer when a layer finishes (keep the transform layer; never `will-change` over the whole scene).
- `Stream` writes words in their final place (no reflow).

### 6.6 Appear, travel, leave
- **Nothing pops**: everything has an entrance and stays mounted until its exit finishes. No holes, no blank frames.
- Neighbors make room continuously. A new row carries the rows below (animate the containing track); it enters whole, and an exiting row leaves before its space closes.
- A resizing surface moves size, position and content as one; never stretches text; retargets from current geometry when interrupted (never re-key).
- Replacing content keeps the container and alignment. Switching versions morphs or crossfades with height eased.
- Solid things (windows, phones, cards) never fade through: they arrive by moving, opaque. Cards, toasts, pills and reactions arrive whole from just above (`Collapse reveal="drop"`); clip only text rows.
- Content after container (the pin number appears after the circle is nearly full).
- A popup is sized and scrolled before its first frame. An open surface holds its anchor (reserve height for the longest list; else `anchor drift`). Overlays close where they were (`{open, at}`).
- A dashed edge never changes size (fade a dashed `::before`); border style doesn't transition. A moving edge never slices text.
- **One object, one trip** ending in its final place: open the room with `Collapse`, measure the slot, then `Fly` from the source's own rect to the landing element's own rect (never a padded wrapper; 4px off shows).
- The flyer changes to its resting look during the last third. Landings decelerate over the last third: no bounce, no ring pulse. After motion, nothing moves.
- A moving selection ring uses `Follow` (keeps momentum when retargeted).
- Popover (kit): scales a touch from the side nearest the trigger, contents fade, never resizes while opening, 4px padding. Closing freezes children: make an in-menu change in one `set`, close in the next.

---

## 7. Cursor, drag, scroll, phones

**Cursor**
- No cursor = no stand-in: never walk a focus ring or spotlight through the scene.
- Elements choose the cursor kind via `data-cursor` (ibeam, hand, open, crosshair, resize-ns/ew/nwse/nesw); it switches mid-path as it crosses an edge, by morphing. Native app buttons keep the arrow.
- The cursor **never covers the words it clicks**: aim inside the target, off its text (`x: 0.85, y: 0.6`). It stays where the press happened.
- Every move travels to the next action, follows a held object, or clears the way. It rests on empty surface near the next move, never on an unused control. Stage fades a resting cursor after 4s.
- I-beam and crosshair are white with a dark edge on a dark stage. Keyboard stories show keys: `act.key('cmd+k')`.

**Drag and selection**
- A mouse drag is one continuous motion (drag-then-throw is a finger flick). It starts where the thing is; an external file enters already held with a fixed grab offset.
- Pickup belongs to the press (set `lifted` in `onMove`); a long press shows press, hold, lift.
- Carried card: `scale(1.03)`, deeper shadow (`-lift`), 1–2° tilt of the whole card including words, `will-change: transform` held throughout, settles flat on drop, no selection outline in the air.
- Held item rides the cursor exactly, no transition. Grab offset from pickup fractions with a shared `transform-origin`.
- Constrained controls follow their mechanism (`{ path }`). Drop targets never travel; the slot jumps cell to cell. Between lists use `Slot` (dashed mark at full height, only fades).
- Text selection runs left → right from a word boundary and stays under the I-beam every frame (`a.drag` + `a.caret(name, at)` in `onMove`; gate fails `selection drift`; never `caretRangeFromPoint`).

**Scroll and camera**
- Nothing scrolls unless the pointer is on it; the cursor moves over the area first. Start with whole rows. Scroll back before the loop ends, not at the seam.
- Camera needs a reason (payoff too small at shown size). Frame the wide shot as a good still first. One push-in per story, in and out; edges feather while zoomed; the cursor holds still while the view moves.

**Phones**
- Act where people do: tap the notification to open the app. Long-press actions read invented. Touch cursor `{ style: 'touch' }`.
- Swipe = flick: a third of the screen in ~0.25s, at the finger's speed (no pause, no ease-in, no snap). A swiped row reveals its action as the row's own back (same height, edge to edge, clipped by the list).
- Reaction = `Bubble` tapback: real emoji in a small white pill on the bubble's lower corner, ringed in the ground. `Bubble` rises from its tail corner; `source` forms out of the composer (field clears in the same update); typing dots grow into the message (one object, never a swap).
- Dynamic Island (platform-faithful): animate the black shape's box and radius together, content on a separate unfiltered layer; 500ms underdamped growth, 380ms shrink delayed 120ms, press squeeze to 0.965; filter blur 2.6, alpha matrix 9 −4.

---

## 8. Bento grids

**Rules**
1. Only ~1/3–1/2 of tiles show UI.
2. ONE hero tile (2 columns or 2 rows).
3. A UI tile shows ONE component at real size or larger, not a window. Window chrome at most once per grid (on the hero).
4. Neighbors contrast like a checkerboard: never two window crops side by side.
5. Stat tile: one real number at 48–72px, a short lead ("Up to"), a one-line label, at most one colored glyph. Real figures only (never "10x"; a missing figure is a visible slot). No stat pinned to the bottom over a void. Number + unit on one line (`nowrap`, or move the unit to the lead).
6. Copy is short and OUTSIDE the picture: a 1–4 word title, or one sentence + at most one supporting sentence. Run-in ("**Stay in sync.** Your files stay up to date.") or a large title alone. Tile copy is page type 17–24px, never 12px UI scaled down.
7. One frame, varied contents: shared radius, gap, padding and copy position. Rounded tiles on a gap, OR a ruled grid with 1px rules. Gap = inner padding or a clear fraction of it. Even outer gutter unless the page sets it. Left-led UI aligns to the copy gutter.
8. Use the tile edge deliberately: an inner fixed-width stage must not slice a window and leave a blank strip.
9. Say each thing once across the set: no repeated strings, avatars, marks or chrome.
10. Compose grounds together: a quiet grid shares one surface plus one photo / accent / inverted tile as punctuation; expressive grids get rhythm from lightness, temperature or saturation.
11. Motion: one place leads. Supporting tiles do one small thing once on scroll-in, staggered 80–120ms in reading order, or on hover (gated by house hover rules).
12. One Stage per tile at 1:1, copy as page HTML outside the stage. Never one Stage around the whole grid (a 1000px grid in a 630px column draws 15px titles at 9.5px).
13. Narrow widths recompose: an ~550px article column may hold two tiles (~270 each); a phone stacks, hero first. Important text ≥10px at a 320px host. Every width is its own design; never center a picture about as wide as its tile.

**Tile kinds:** One component · Hero UI · Stat · Statement (large type) · Object (card, chip, isometric drawing) · Diagram (grid, heatmap, nodes) · Collection (logos or app icons bleeding off) · Photo (real licensed photo only; never an SVG/CSS-drawn "photo"; substitute a purely graphic pattern or smooth curves, still).

Five-tile default: one hero UI, 1–2 components, one stat or statement, one object / diagram / collection.

**Layouts:** Hero and strip (2/3 + 1/3 row, then three equal; default for 5–6) · Staggered pair (two columns, tall/short alternating, offset) · Even grid 3×2 (only if every neighbor differs in kind) · Big square (4 columns, 2×2 hero) · Ruled (one rounded container, 1px rules, no gaps, mixed spans; technical pages).

**Plan before building**, then check rules 1, 2, 3, 4 and 9 against the table:

| Tile | Span | Kind | The one thing | Words |
|---|---|---|---|---|
| 1 | 2×2 | Hero UI | Assigning an issue lands it in a teammate's inbox | "Assign to Maya" |
| 2 | 1×1 | Stat | [real figure from the user] | "Up to … faster triage" |

---

## 9. Annotations

- An annotation explains ONE relationship. Design the product first; add annotations after. Leave the line out if the layout already explains it.
- Leader: 1px (1.5px at hero scale) quiet ink; leaves the note's edge; lands 4–6px short of the object ending in a 4px dot (or an edge with a gap); straight or one bend; never through surfaces; never crossing a rounded border.
- Record link (dotted) joins two views of the same thing. A collection joins inputs at one junction, then delivers after all arrive.
- Direction shows transfer: request and response start from different ends; status changes after the line lands. Solid request vs dashed acknowledgement; a shared trunk is drawn once.
- Moving dots only while data flows: fixed route, only the phase animates; pause offscreen; static poster under reduced motion.
- Node pill: white, fine rim, tight shadow. Inline badge on the line (`POST`, `200 OK`) in mono, not tracked caps. One relationship = one color.
- Build route geometry, stroke pattern, reveal and phase separately (reveal with a mask so dots don't stretch).
- Attach via measured `rect()` named targets, to the facing edge; check the final segment direction (`route="h"`). `rect()` refreshes on commit / transition end, not per frame: put object and annotation in the same moving coordinate space (`Follow` wrapper).
- `Connector`: dots 3px radius accent + 2px white rim + subtle shadow; `arrow`, `label` (arrives after the line), `route="step"` (several lines into one target, ~80px between columns) or `"curve"`.
- Leave room for the arrowhead and label inside the frame clip. A narrow host stacks; labels stay normal size.
- On a page of demos, mark a bad example in the page's own annotation style.

---

## 10. Anti-slop style pairs (failure → correction)

| Failure | Correction |
|---|---|
| Purple slop: violet-to-pink gradient ground, gradient text, tinted cards | Product neutrals + one accent |
| Tiny text, 9–10px | 13–14px names, 11–12px metadata, fewer rows |
| Over-detailed rows: branch, hash, author, duration, badge on every row | Detail only on the story row |
| Neon glow template | One soft neutral shadow |
| TRACKED CAPITALS on headings, labels, buttons | Sentence case |
| Invented icons in rainbow tiles; filled beside outlined | One pack, one stroke; color only for status |
| AI shorthand: sparkles, wands, gradient "AI" chips | Say what happens |
| Onion skin: translucent icon ink darkening at overlaps | One opaque color |

**Motion pairs**

| Failure | Correction |
|---|---|
| Instant label replacement | Stable button, clipped label slot, `Roll` |
| Result without press feedback | Small press response first (house press value) |
| New row shoves neighbors | Animate the containing track or reserve space |
| Plus → close loses identity | Rotate the same strokes |
| Exit skipped | Keep mounted until the exit finishes |
| Interrupted motion restarts | Retarget from current geometry; don't re-key |
| Landing jumps at handoff | Measure the element via `rect()`, not the padded wrapper |
| Loop erases work | Visible cause (Clear) |
| Added icon shifts neighbor | Open width + gap together |
| Surface and content resize separately | One coordinated geometry |
| Drag from the wrong point | Keep the grab offset |
| Spinner starts after appearing | Rotate before visible |
| Text shifts as it settles | Move the label by transform in a stable grid |
| Label hits a hard edge | Soften the moving boundary locally |
| Scrolling reveals a harsh crop | Whole rows; fade only hiding edges |
| Dependent changes start together | Offset the starts while movements overlap |

"Matching endpoint screenshots prove nothing about the frames between."

**Detail pairs:** chart draws left → right, area follows, end dot lands when the line arrives · balance a top bar (one control + an empty side = unfinished) · choose copy that fits (no truncated chips or wrapped names unless the brief fixes them).

**Making a comparison:** change ONE thing; same content, layout, endpoints and timing; label the exact difference; readable at normal speed.

---

## 11. Verify

Answer from frames, never from code. Prove the still before choreography: render opening and payoff at both widths beside gold at shown size. Can a viewer find the focal object and what changes before reading branding or copy?

### 11.1 film.mjs (headless Chrome via playwright-core; contact sheet at true size)

| Need | Command flags |
|---|---|
| Loop contact sheet | `--every 1000 --count 18` |
| Still / poster | `--still` |
| Phone width | `--width 390` |
| Step a beat | `act.mark('drop')` in the script, then `--at drop --every 50 --count 24` (never guess delays) |
| Zoom a region | `--zoom x,y,w,h` or `--zoom target:<name>[,pad]` |
| Hover / dark / reduced motion / control states | `--hover` / `--dark` / `--reduced` / `--states` |
| Slow motion | `--slow 6 --every 40` |
| Report | `--report <json>`, `--strict` (exit 1 on candidates), `--gate` |

Run at least: `node scripts/film.mjs '<url>' --select '.your-demo' --gate`, the same with `--width 390`, and with `--states`. Host controls: `node scripts/layout.mjs "<url>" --select .pg --width 390 --report … --strict`. Long loops: `jumps.mjs --ms 30000`. Coverage gaps >1.6× interval or intervals >40ms = inconclusive (exit 2). "No issues detected" is not approval. Without the scripts: Playwright screenshots on a fixed interval, plus the DevTools Animations panel at 10% speed.

**Gate lines** (all must be 0; clearing one by hiding or marking it is a bug): framing · cut by the crop · tight stack / loose stack · tight mark · tight inset · skeleton ladder · onion skin · kit override · unit wrapped · clipped glyph · loose list · fat row · empty pane · emoji icon · anchor drift · uneven padding · mixed icons · tight control · invented mark · uneven inset · uneven gutters · avatar alignment · spills its slot · selection drift.

**Candidate lines** (judge each): touching (<5px) · tight lines · lopsided · uneven · near-tangent · half-pixel glyph · squeezed · covers · dashed edge resizes · handoff shift · settle snap · fly size · hard edge · appears then moves · near-empty frames · lands on a transform · steps in its box · clipped · unclipped · double corner · appears with no entrance / vanishes with no exit (film each at 50ms) · shrunk, not laid out for phone · small text <10px at widths <600. Mark intentional steps with `data-snap`.

### 11.2 Checklist

- [ ] **Gate:** passes at wide, 390 and `--states`.
- [ ] **Framing:** one gutter + bleed; nothing with words cut; the ground equals the page named in the brief; no empty corner; panes full; phone and island rules.
- [ ] **Hierarchy and density:** exactly one stated difference vs gold; the subject has real words; label/value match; named avatars colored; realistic lists hold ≤ as many placeholders as real rows; heaviest mark on the story; each bento tile a specific moment; stat + unit on one line.
- [ ] **Motion:** each beat filmed at 40–80ms; nothing appears or leaves without entrance/exit; swipe, reaction, terminal and number rules; the loop ends where it began; UI feedback at house values.
- [ ] **Polish:** 4× zoom on anything hand-built; no translucent icons or onion skin; 12px inset / 4–6px stack / 6px mark clearance / 8px control clearance; nothing <10px on phone.
- [ ] **Reduced motion:** shows the complete poster; toggling during playback works.
- [ ] **Live pacing:** rAF log vs transform; pause, offscreen and hidden tab stop work; resume never skips or replays accumulation.
- [ ] **Host:** fresh load at every width; no full-page wrapper in the component; every look filmed; typecheck passes.
- [ ] Tools can't sign off feel: watch at speed, slowed, interrupted, and after the final edit.

### 11.3 You see → you do

| You see | You do |
|---|---|
| Void under a list | More product, or crop the pane; never spread rows |
| Fat row | Padding 12px (8px dense) + more rows |
| Loose list | Tighten the pitch |
| Whole window floating | Bleed off two edges, or fill |
| Gray bars where the eye lands | Real words |
| Label + bar | Both words, or drop it |
| Unit wrapped | `nowrap`, resize, or move the unit |
| Small or whole phone in a narrow host | Crop it |
| Grabber under the island | Start the sheet below the status bar |
| Anchor drift | Pin by the opening edge + reserve height |
| Emoji used as icon | A glyph from the pack |
| Clipped glyph | Fit the viewBox |
| Mark sitting on words | Move it to the gutter |
| A gate line you think is intentional | Justify it in REHEARSAL.md with the frame |
| Ten runs green and nothing changes | Stop; get a fresh eye |

---

## 12. Fresh-eye review brief

Spawn a subagent with no transcript (SKILL.md §2.7). Give it: this file, the frames (contact sheets at shown width and 390, poster, stepped beats), the URL, the gold still, and a one-line brief in the user's own words. Do NOT give it the plan, your reasons or REHEARSAL.md.

Brief to paste:
```
You are reviewing a faux product UI illustration you did not build.
Brief: "<one line, the user's words>". Gold reference: <gold.png>. Frames: <paths>. URL: <url>.
1. First question: does this composition answer this product and this page, or could it be any product in its category?
2. Start from the brief's own words (e.g. "white page" makes a tinted plate finding #1).
3. Answer the §11.2 checklist from frames. Film missing frames yourself. A predicted failure or pass is not a finding.
4. State the one difference vs gold.
5. Rank findings framing > hierarchy/density > motion > polish, by how much each spoils the picture, not by ease of fix.
   Each: what you would screenshot, which frame, one-line fix.
6. Re-judge every line marked "intentional". Don't fix anything. Don't soften.
```

---

## 13. REHEARSAL.md template

Write beside the component.

```markdown
# Rehearsal: <component>

## Gold
Copied: <example>. Kept: <stage size, crop, row/type sizes, beats>. Changed: <the one difference, look, content>.

## Beats
1. <beat> (t = 0.0s)
2. <beat>
3. <beat>
Payoff: <what is on screen, held 2s>. Loop: <n>s. Reset cause: <visible cause | exit around anchor | snap while hidden>.
Poster: <beat shown as the still>.

## Captured
Widths: <shown>, 390. Looks: <light/dark/...>. States: <rest/hover/pressed/focused>. Reduced motion: <yes/no>.

## Film
| Gate/candidate line | Frame | Fix or why it stays |
|---|---|---|

## Review
| Fresh-eye finding | Response |
|---|---|

## Assumptions
## Unresolved
## Not checked
```
