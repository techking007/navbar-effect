# Navbar Effect Adapters

Port the [spec.md](spec.md) physics and DOM contract. Do not invent a different interaction model.

## Hard rule

Keep every dropdown menu mounted. Toggle opacity and `pointer-events`. Measure the active node and animate the shared size shell. Remounting on menu change kills the width/height morph.

## React / Next (default)

Match the reference split:

- `nav-hover` primitives: `NavChevron`, `DropdownHighlight` (`"use client"`)
- Header: hover zone, `openMenu` / `heldMenu`, post-layout measure, shared flyout

Use `useLayoutEffect` for pill box and panel measure. Use `useEffect` + `requestAnimationFrame` for the chevron spring. Use `cn()` and Tailwind v4 tokens when the project already has them.

Fallback file names if the project has no header yet:

- `src/components/nav-hover.tsx`
- `src/components/header-nav.tsx`

## Vue

- Chevron: watch `open`, run the same rAF spring against a `ref` on `<polyline>`
- Highlight: `watch(activeIndex)` + `nextTick` (or `flush: 'post'`) to measure
- Panel: `watch([openMenu, heldMenu, hoverIndex])` then read `offsetLeft` / `offsetWidth` / `offsetHeight`
- Keep stacked menus in one `v-for`; bind `:style` opacity / `translateX(delta * 16px)`
- Do not use `v-if` on individual menus

## Svelte

- Chevron: `$effect` on `open` with the same spring
- Highlight: `$effect.pre` or `tick()` then measure
- Panel: bind `bind:this` on each trigger `li` and menu node
- `{#each}` all menus; never `{#if}` the inactive ones out of the DOM

## Vanilla / any other framework

One shared panel element. Algorithm:

1. On trigger enter: set open id, clear close timer, reset highlight index
2. After layout: `panel.style.transform = translateX(${trigger.offsetLeft}px)`; set size shell to the active menu's offset size
3. Each menu stays as an absolutely positioned child; set opacity and `translateX((index - activeIndex) * 16px)`
4. Pill: query `[data-dropdown-link]`, copy its rect relative to the highlight root
5. On zone leave: `80` ms later clear open id

CSS transitions live on the wrappers listed in the spec. JS only writes the target numbers.

## CSS-only is not enough

`:hover` on a single submenu cannot morph a shared panel across triggers, cannot spring the chevron points, and cannot snap-then-ease the highlight pill. Use a small amount of JS.

## Tokens without Tailwind

| Tailwind in the reference | Portable equivalent |
|---------------------------|---------------------|
| `bg-primary/[0.04]` | `color-mix(in oklch, var(--foreground) 4%, transparent)` or 4% of the project's text color |
| `bg-primary/[0.03]` | 3% of the same |
| `bg-card` | project surface / popover background |
| `text-primary` / `text-primary/50` | project foreground / 50% |
| `rounded-xl` | `12px` |
| `rounded-md` | `6px` |
| `rounded-lg` | `8px` |
| `shadow-xl shadow-black/10` | `0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)` |

Keep the millisecond values and the cubic-bezier exactly as in the spec.
