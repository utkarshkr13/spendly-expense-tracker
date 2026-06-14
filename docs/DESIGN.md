# Nivo — Design System

> Derived from Apple HIG, iOS 26 spirit. Figma MCP unavailable on runs 1–5 — tokens defined manually from HIG. Future runs should pull from Figma variables when available.

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
| Segmented control container | `rounded-xl` |
| Segmented control button | `rounded-lg` |
| Badge / chip | `rounded-full` |
| Mini badge | `rounded` (4px) |
| Logo | `rounded-xl` |

## Shadows (Apple Depth Tokens)

| Component | Shadow |
|---|---|
| Card (default) | `.card-shadow` — `0 1px 3px rgba(0,0,0,0.05), 0 4px 14px rgba(0,0,0,0.05)` |
| Card (hover) | `0 2px 6px rgba(0,0,0,0.08), 0 8px 22px rgba(0,0,0,0.08)` |
| Auth card | `shadow-xl` |
| Active nav item | `shadow-sm` |
| Gradient hero card | `shadow-lg` |
| Modal overlay | `bg-black/30` |
| Segmented active option | `shadow-sm ring-1 ring-black/[0.06]` |

## Interaction

### iOS Press State
```css
.btn-press { transition: transform 0.1s ease, box-shadow 0.1s ease; }
.btn-press:active { transform: scale(0.97); }
```

### iOS Segmented Control (New v5)
```css
.seg-btn { transition: background 0.15s ease, color 0.15s ease, box-shadow 0.15s ease; }
```

Container: `flex bg-slate-100 rounded-xl p-1 gap-1`  
Active option: `bg-white shadow-sm text-slate-800 ring-1 ring-black/[0.06]`  
Inactive: `text-slate-500 hover:text-slate-700`

Applied to: Transactions page type filter (All / Debits / Credits).  
Convention: use `SegmentedControl` for any 2–4 mutually exclusive toggle; prefer over `<select>` when options fit in one row.

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
  border-color: rgba(203, 213, 225, 0.6);
}
```

Applied to: sidebar (`<aside>`) and sticky header (`<header>`).

## Motion

| Element | Motion | Class / CSS |
|---|---|---|
| Toggle switch | Thumb slide | `transition-all` |
| Nav item background | Color fade | `transition-colors` |
| Budget / bar chart fill | Width reveal | `transition-all duration-500` |
| Card on hover | Shadow depth | `transition: box-shadow 0.18s ease` |
| Button / card press | Scale down | `transition: transform 0.1s ease` |
| Segmented control | Color + shadow | `.seg-btn` — `0.15s ease` |
| Savings ring | Dashoffset | `transition: stroke-dashoffset 0.5s ease` |

## Empty States

- Table with no matches: centered `text-sm text-slate-400`
- "No data" in AreaSpark: centered placeholder
- Donut with no spending: "No spending data yet" text placeholder
- "Link another account": `border-dashed` card

## Components

### SavingsGoalCard (v4)
- SVG ring: r=36, `strokeDasharray = 2πr`, `strokeDashoffset = circ*(1-pct/100)`
- Color: emerald (on-track), indigo (behind)
- Inline goal input: `border-b border-dashed border-indigo-300`

### SegmentedControl (v5)
- Props: `options: {value, label}[]`, `value`, `onChange`, `className`
- Each button: `aria-pressed`, `aria-label`
- No external deps — pure Tailwind + inline transition

## Accessibility

- Minimum tap target: 44×44px (iOS HIG). Nav buttons: `w-full py-2.5`.
- Focus rings: `focus:ring-2 focus:ring-indigo-200` on all inputs.
- Color not sole indicator — badges + text labels always accompany color.
- `aria-label` on all interactive elements.
- `aria-current="page"` on active nav item.
- `role="alert"` on error messages.
- `aria-pressed` + `aria-label` on SegmentedControl buttons.

## Figma Reference

Figma MCP unavailable on runs 1–5 (2026-06-14). When available, pull:
- Variable sets → update color tokens above
- Component library → align Card/Button shapes
- Screenshot of any new screen → derive spacing/type from it

[[NIVO_CONTEXT]] | [[AI_PLAN]] | [[ROADMAP]]
