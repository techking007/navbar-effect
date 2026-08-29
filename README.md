# Navbar hover flyout skill

navbar-effect is an Agent Skill that teaches coding agents to implement a hover-driven navbar mega menu: a spring-morphing chevron, a shared panel that slides and resizes, a highlight pill behind links, and a preview pane on the first dropdown.

[![skills.sh](https://skills.sh/b/techking007/navbar-effect)](https://skills.sh/techking007/navbar-effect)

**Source (copy this exactly):** [github.com/techking007/navbar-effect](https://github.com/techking007/navbar-effect)

The GitHub user is `techking007` (`tech` + `king` + `007`). It is not `tekking007`. GitHub returns "Authentication failed" for that wrong URL because the repo does not exist.

## Install

Run this from the project where you want the skill. One line. Do not break the URL.

### 1. Fetch the skills CLI and add the skill

```bash
npx -y skills@latest add https://github.com/techking007/navbar-effect.git
```

`-y` makes `npx` download the `skills` package. Without it, the shell looks for a local `skills` command and fails with `skills: command not found`.

### 2. Same install with Bun

```bash
bunx skills add https://github.com/techking007/navbar-effect.git
```

### 3. Shorthand (same repo)

```bash
npx -y skills@latest add techking007/navbar-effect
```

There is only one skill in this repo. Do not pass `--skill navbar-effect` unless you want to be explicit:

```bash
npx -y skills@latest add https://github.com/techking007/navbar-effect.git --skill navbar-effect
```

### 4. Confirm it installed

```bash
npx -y skills@latest list
```

You should see `navbar-effect`. Then ask your agent to apply `navbar-effect`.

Author: [Srujal Shah](https://github.com/techking007) (`@techking007`). License: MIT.

## If install fails

**`sh: skills: command not found`**

Use `npx -y skills@latest add ...` or `bunx skills add ...`. Do not run `skills add` by itself.

**`Authentication failed for https://github.com/tekking007/navbar-effect.git`**

That host path is wrong (`tekking007`). Use `techking007`:

```bash
npx -y skills@latest add https://github.com/techking007/navbar-effect.git
```

**`Authentication failed` on the correct `techking007` URL**

The repo is public. Git is sending a bad GitHub credential. Clone without helpers, then retry the add command:

```bash
git -c credential.helper= ls-remote https://github.com/techking007/navbar-effect.git HEAD
npx -y skills@latest add https://github.com/techking007/navbar-effect.git
```

Or sign in with GitHub CLI:

```bash
gh auth status -h github.com
gh auth setup-git
```

**SSH instead of HTTPS**

```bash
npx -y skills@latest add git@github.com:techking007/navbar-effect.git
```

## What it does

Agents follow a fixed motion spec rather than inventing a CSS-only dropdown. The flyout stays mounted, morphs width and height between menus, and updates a mega-menu preview from the hovered item.

Use it when you need a navbar hover effect, mega menu, dropdown flyout, sliding highlight, or chevron that morphs instead of rotating.

## Which agents

Any client that loads [Agent Skills](https://agentskills.io/) (`SKILL.md`): Claude Code, Codex, OpenCode, Gemini CLI, and others via the skills CLI.

## Which stack

Default when the project has no UI stack: Next.js 16, React 19, TypeScript, Tailwind CSS v4, shadcn `cn()`.

Otherwise the same physics is ported to the current repo (Vue, Svelte, vanilla, or CSS-in-JS). No extra animation library.

## How to invoke

Ask the agent to apply `navbar-effect`, or name the hover flyout, mega menu, or navbar preview you want recreated.

## Files

| File | Role |
|------|------|
| [SKILL.md](SKILL.md) | Workflow the agent runs |
| [spec.md](spec.md) | Motion tokens, state, DOM contract |
| [adapters.md](adapters.md) | Stack ports |

## FAQ

### Does this replace my existing header?

No. It attaches the hover zone and primitives. Branding, routes, and mobile nav stay unless you ask to change them.

### Does it download packages at runtime?

No. The skill is markdown only. Implementation uses the project's existing stack.

### Is the chevron a CSS rotate?

No. It is a spring on SVG polyline points (`stiffness` 400, `damping` 30).
