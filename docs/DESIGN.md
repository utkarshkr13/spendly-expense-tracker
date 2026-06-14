# Nivo — Design System

> Derived from Apple HIG, iOS 26 spirit. Figma MCP unavailable on first two runs — tokens defined manually. Future runs should pull from Figma variables when available.

## Brand

| Token | Value |
|---|---|
| Brand primary | `#6366f1` (indigo-500) |
| Brand gradient | `from-indigo-600 to-violet-600` |
| Brand name | **Nivo** (never rename) |
| Currency | ₹ Indian Rupee, `toLocaleString("en-IN")` |

## Typography

Follows SF Pro / system-ui stack:
```css
font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
```

| Role | Size | Weight | Class |
|---|---|---|---|
| Page title | 20px | 600 | `text-xl font-semibold` |
| Section header | 14px | 600 | `text-sm font-semibold` |
| Stat value | 30px | 600 | `text-3xl font-semibold tracking-tight` |
| Body | 14px | 400 | `text-sm` |
| Caption / meta | 12px | 400 | `text-xs text-slate-400` |
| Micro badge | 10px | 400 | `text-[10px]` |

## Color Tokens

### Category Colors
```js
Food:          #ef4444   (red-500)
Shopping:      #8b5cf6   (violet-500)
Transport:     #3b82f6   (blue-500)
Rent:          #0ea5e9   (sky-500)
Bills:         #f59e0b   (amber-500)
Entertainment: #ec4899   (pink-500)
Health:        #14b8a6   (teal-500)
Travel:        #22c55e   (green-500)
Income:        #10b981   (emerald-500)
Transfer:      #94a3b8   (slate-400)
Other:         #64748b   (slate-500)
```

### Semantic Tones (Stat cards)
```
rose    → text-rose-600    bg-rose-50    (spend, danger)
emerald → text-emerald-600 bg-emerald-50 (income, success)
indigo  → text-indigo-600  bg-indigo-50  (neutral highlight)
amber   → text-amber-600   bg-amber-50   (savings, warning)
slate   → text-slate-600   bg-slate-100  (counts, meta)
```

## Radius

| Component | Radius |
|---|---|
| Card | `rounded-2xl` (16px) |
| Nav item (active) | `rounded-xl` (12px) |
| Input / select | `rounded-lg` (8px) |
| Badge / chip | `rounded-full` |
| Mini badge | `rounded` (4px) |
| Logo | `rounded-xl` |

## Shadows (Updated v3 — Apple Depth Tokens)

| Component | Shadow |
|---|---|
| Card (default) | `.card-shadow` — `0 1px 3px rgba(0,0,0,0.05), 0 4px 14px rgba(0,0,0,0.05)` |
| Card (hover) | `0 2px 6px rgba(0,0,0,0.08), 0 8px 22px rgba(0,0,0,0.08)` |
| Auth card | `shadow-xl` |
| Active nav item | `shadow-sm` |
| Gradient hero card | `shadow-lg` |
| Modal overlay | `bg-black/30` |

The two-stop layered shadow (close ambient + diffuse drop) mirrors Apple's UIKit card shadow spec — renders depth without the harsh cutout look of single-stop shadows.

## Interaction — iOS Press State (New v3)

```css
.btn-press {
  transition: transform 0.1s ease, box-shadow 0.1s ease;
}
.btn-press:active {
  transform: scale(0.97);
}
```

Applied to: all `<button>` elements, clickable `Card` wrapper (when `onClick` prop provided), nav items.
Provides subtle tactile feedback matching iOS's "spring scale" press animation.

## Spacing

- Page padding: `p-5 lg:p-7`
- Card padding: `p-5` (standard), `p-4` (compact)
- Section gap: `space-y-5` or `space-y-6`
- Grid gap: `gap-4`

## Frosted Glass (iOS 26 Spirit)

```css
.glass-nav {
  background: rgba(255, 255, 255, 0.82);
  backdrop-filter: saturate(180%) blur(20px);
  -webkit-backdrop-filter: saturate(180%) blur(20px);
  border-color: rgba(203, 213, 225, 0.6); /* slate-300/60 */
}
```

Applied to: sidebar (`<aside>`) and sticky header (`<header>`).

## Motion

| Element | Motion | Class / CSS |
|---|---|---|
| Toggle switch | Thumb slide | `transition-all` |
| Nav item background | Color fade | `transition-colors` |
| Budget / bar chart fill | Width reveal | `transition-all duration-500` |
| Card on hover | Shadow depth | `transition: box-shadow 0.18s ease` (`.card-shadow`) |
| Button / card press | Scale down | `transition: transform 0.1s ease` (`.btn-press`) |
| Chart paths | ❌ (P2) | SVG SMIL planned |

## Empty States

- Table with no matches: centered `text-sm text-slate-400`
- "No data" in AreaSpark: centered placeholder
- Donut with no spending: "No spending data yet" text placeholder
- "Link another account": `border-dashed` card

## Accessibility

- Minimum tap target: 44×44px (iOS HIG). Nav buttons: `w-full py-2.5`.
- Focus rings: `focus:ring-2 focus:ring-indigo-200` on all inputs.
- Color not used as sole indicator — badges + text labels always accompany color.
- `aria-label` on all interactive elements.
- `aria-current="page"` on active nav item.
- `role="alert"` on error messages.

## Figma Reference

Figma MCP was unavailable on runs 1–2 (2026-06-14). When available, pull:
- Variable sets → update color tokens above
- Component library → align Card/Button shapes
- Screenshot of any new screen → derive spacing/type from it

[[NIVO_CONTEXT]] | [[AI_PLAN]] | [[ROADMAP]]
