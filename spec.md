# Navbar Effect Spec

Exact motion, state, and DOM contract. Do not paraphrase these constants.

## State

| Name | Type | Role |
|------|------|------|
| `openMenu` | `string \| null` | Currently hovered dropdown label. Null when the flyout is closed. |
| `heldMenu` | `string \| null` | Last opened dropdown. Updated whenever `openMenu` becomes non-null. Used so stacked menus and size stay correct while closing. |
| `hoverIndex` | `number` | Active `[data-dropdown-link]` in the open menu. `-1` when none. Reset to `-1` on every `openDropdown`. |
| `panelX` | `number` | Trigger `offsetLeft` of `openMenu ?? heldMenu`. |
| `panelSize` | `{ width, height }` | Measured `offsetWidth` / `offsetHeight` of the active menu node. Initial `{ width: 220, height: 180 }`. |
| `previewLabel` | `string` | Mega preview caption. Defaults to the first item label of the first dropdown. |
| `closeTimer` | timeout id | `80` ms delayed close. |

## Open / close

```
openDropdown(label):
  clear closeTimer
  openMenu = label
  hoverIndex = -1
  (heldMenu follows openMenu in an effect)

closeDropdown():
  clear closeTimer
  closeTimer = after 80ms:
    openMenu = null
    hoverIndex = -1
```

- Hover (not click) opens. Trigger `mouseenter` with items calls `openDropdown`. Trigger without items sets `openMenu = null`.
- `[data-nav-hover-zone]` `mouseleave` calls `closeDropdown`.
- Flyout wrapper `mouseenter` re-opens `heldMenu` so the pointer can cross the `pt-2` gap.
- Escape sets `openMenu = null`.

## Measurement

After layout (`useLayoutEffect` or equivalent), when `openMenu ?? heldMenu` is set:

- `panelX = triggerLi.offsetLeft`
- `panelSize = { width: menu.offsetWidth, height: menu.offsetHeight }`

Re-run when `openMenu`, `heldMenu`, or `hoverIndex` changes.

## Shared flyout (must stay mounted)

Never remount the panel on menu change. Structure:

1. Hover zone (`relative`, `data-nav-hover-zone`)
2. Trigger row (`ul` of labels; chevron only on items with children)
3. Positioner: `absolute top-full pt-2`, `translateX(panelX px)`
4. Enter shell: `origin-top-left`, opacity + scale
5. Size shell: `overflow-hidden rounded-xl`, animated `width` / `height`
6. Stack: `absolute top-0 left-0` children, one per dropdown. All stay in the DOM.

Active menu: `(openMenu ?? heldMenu) === label`.

```
delta = index - max(activeIndex, 0)
transform: translateX(delta * 16px)
```

First dropdown (`index === 0`) is the mega menu. Others are link-only (`w-[220px]`).

## Transition table

Easing token: `cubic-bezier(0.16, 1, 0.3, 1)`

| Surface | Property | Open | Closed / inactive |
|---------|----------|------|-------------------|
| Positioner | `transform` `translateX(panelX)` | `0.4s` easing | `none` |
| Positioner | `pointer-events` | `auto` | `none` |
| Enter shell | `opacity` | `1` | `0` |
| Enter shell | `transform` `scale` | `1` | `0.96` |
| Enter shell | transition | `opacity 0.2s` easing, `transform 0.2s` easing | same |
| Size shell | `width`, `height` | `0.4s` easing | `none` |
| Stacked menu | `opacity` | `1` | `0` |
| Stacked menu | `translateX(delta * 16px)` | `0.35s` easing | same |
| Stacked menu | `pointer-events` | `auto` if active | `none` |

## Mega preview

- Link column: `w-[220px] shrink-0 px-2 py-2`
- Preview column: `w-[480px] shrink-0 p-3`
- Preview tile: `bg-primary/[0.03]`, `min-h-[280px]`, `rounded-lg`, centered visual + `previewLabel`
- On mega item `mouseenter`: `previewLabel = item.label`
- Visual: use the current project's mark or a simple tile. Do not copy Relay / bot-mark assets.

Link rows: `h-[36px] py-1.5` without description; `py-3` with description. Description: `text-primary/40 text-[11px] leading-snug font-normal`.

## NavChevron

Morph polyline points. Do not rotate the SVG.

- Down: `4,6 8,10 12,6`
- Up: `4,10 8,6 12,10`
- Interpolation: `sideY = 6 + 4 * p`, `midY = 10 - 4 * p`, points `4,{sideY} 8,{midY} 12,{sideY}`
- `p` target: `1` open, `0` closed
- Spring: `stiffness = 400`, `damping = 30`
- `accel = stiffness * (target - p) - damping * v`
- `v += accel * dt`, `p += v * dt`
- `dt = min(0.032, (now - last) / 1000)`
- Stop when `abs(target - p) <= 0.002` and `abs(v) <= 0.002`; snap `p = target`, `v = 0`
- `prefers-reduced-motion: reduce`: snap `p = target` immediately, no rAF
- SVG: `viewBox="0 0 16 16"`, `fill="none"`, `stroke="currentColor"`, `strokeWidth="1.75"`, `strokeLinecap="round"`, `strokeLinejoin="round"`, `aria-hidden`

## DropdownHighlight

Sliding pill behind `[data-dropdown-link]`.

- Pill: `pointer-events-none absolute rounded-md`, fill `bg-primary/[0.04]` (or `color-mix` equivalent at 4% of foreground)
- Initial inline style: `opacity: 0; top: 0; left: 0; width: 0; height: 0`
- Track `armed` (boolean). Start `false`.
- Root `mousemove`: closest `[data-dropdown-link]` index, or `-1`
- Root `mouseleave`: index `-1`

When `activeIndex < 0`:

- After `80` ms: `transition: opacity 150ms ease`, `opacity = 0`, `armed = false`

When `activeIndex >= 0`:

- Measure link vs root `getBoundingClientRect`; set `top`, `left`, `width`, `height`
- If `armed`:
  - `transition: top 150ms cubic-bezier(0.16, 1, 0.3, 1), left 150ms cubic-bezier(0.16, 1, 0.3, 1), width 150ms cubic-bezier(0.16, 1, 0.3, 1), height 150ms cubic-bezier(0.16, 1, 0.3, 1), opacity 100ms ease`
  - `opacity = 1`
- If not `armed`:
  - `transition: none`, set box, force reflow (`getBoundingClientRect()`), then `transition: opacity 100ms ease`, `opacity = 1`
- Then `armed = true`

## Trigger chrome

- Trigger text: `text-sm font-medium`, `px-3 py-1.5`, `gap-1`
- Idle: `text-primary/50`; open: `text-primary`
- Chevron: `size-3` beside the label
- Panel card: `bg-card`, `rounded-xl`, `shadow-xl shadow-black/10`

## Hard rules

- No Framer Motion, GSAP, or extra animation libraries
- Do not remount menus on switch
- Do not rotate the chevron
- Do not copy Relay / Northwind / bot-mark content into other projects
