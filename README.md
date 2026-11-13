# Skills

A collection of agent skills and instructions for building WordPress themes (Roots Sage, PHP 8.4) with Tailwind CSS, Alpine.js and GSAP.

## Contents

| Item | Description |
|---|---|
| [AGENTS.md](AGENTS.md) | Always-on agent instructions: tech stack (Tailwind, Alpine, GSAP, Sage/WordPress), rules, layout, responsive, styleguide, Gravity Forms, accessibility and image conventions. Loaded in a project through `CLAUDE.md` (see [Usage](#usage)). |
| [tailwindcss](skills/tailwindcss/SKILL.md) | Tailwind CSS v4 conventions and best practices |
| [alpinejs](skills/alpinejs/SKILL.md) | Alpine.js conventions and best practices |
| [gsap](skills/gsap/SKILL.md) | GSAP animation conventions and best practices |
| [wordpress-theme](skills/wordpress-theme/SKILL.md) | WordPress theme development with Sage, Blade and PHP 8.4 |
| [acf-elements](skills/acf-elements/SKILL.md) | Make Blade elements manageable through ACF flexible content via local JSON |

## Structure

Each skill lives in its own folder containing a `SKILL.md` file with instructions and any supporting files. Every `SKILL.md` follows the same outline, so they stay predictable to read and to extend:

| Section | Contains |
|---|---|
| `## Contents` | Table of contents for in-file navigation |
| `## Documentation` | Links to the official docs |
| `## When to use` | The situations that should trigger this skill |
| `## Core principles` | The few ideas everything else follows from |
| `## Conventions` | The actual rules, grouped in `###` subsections |
| `## Anti-patterns` | Avoid/Do-instead table |
| _(optional)_ | Extra material between Conventions and Anti-patterns (e.g. `## Templates` in acf-elements) and/or a closing section: `## Tooling`, `## Working with …` or `## Related skills` |

The repo is laid out like a project's `.claude` directory, so it can be mounted there as a whole:

```
.claude/                 ← this repo
├── CLAUDE.md            imports AGENTS.md (`@AGENTS.md`)
├── AGENTS.md
└── skills/
    ├── tailwindcss/SKILL.md
    ├── alpinejs/SKILL.md
    ├── gsap/SKILL.md
    ├── wordpress-theme/SKILL.md
    └── acf-elements/SKILL.md
```

## Usage

Add this repo to a project as a git submodule on `.claude` — no copying and no symlinks:

```sh
git submodule add git@github.com:g2odev/skills.git .claude
```

- **Skills**: available as `.claude/skills/<name>/SKILL.md` and loaded on demand
- **Instructions**: Claude Code loads `.claude/CLAUDE.md` automatically on every request; it imports `AGENTS.md`, which holds the actual rules. The skill links in AGENTS.md are relative (`skills/…`), so they resolve both in a project and inside this repo
- **Project settings**: `.claude/settings.local.json` is ignored by this repo, so a project's local Claude settings never end up here. The project must not track a `.claude` directory of its own — the submodule takes that path
- **Cloning a project**: `git clone --recurse-submodules …`, or `git submodule update --init` in an existing clone
- **Updating**: `git submodule update --remote .claude`, then commit the new submodule pointer

## License

[MIT](LICENSE)