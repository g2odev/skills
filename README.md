# Skills

A collection of agent skills and instructions for building WordPress themes (Roots Sage, PHP 8.4) with Tailwind CSS, Alpine.js and GSAP.

## Contents

| Item | Description |
|---|---|
| [AGENTS.md](AGENTS.md) | Always-on agent instructions: tech stack (Tailwind, Alpine, GSAP, Sage/WordPress), rules, layout, responsive, styleguide, Gravity Forms, accessibility and image conventions. Link into the root of a project (see [Usage](#usage)). |
| [tailwindcss](tailwindcss/SKILL.md) | Tailwind CSS v4 conventions and best practices |
| [alpinejs](alpinejs/SKILL.md) | Alpine.js conventions and best practices |
| [gsap](gsap/SKILL.md) | GSAP animation conventions and best practices |
| [wordpress-theme](wordpress-theme/SKILL.md) | WordPress theme development with Sage, Blade and PHP 8.4 |
| [acf-elements](acf-elements/SKILL.md) | Make Blade elements manageable through ACF flexible content via local JSON |

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

The skill folders live in the root of the repo, so the repo itself can be mounted as a project's `.claude/skills` directory:

```
skills/
├── AGENTS.md
├── tailwindcss/SKILL.md
├── alpinejs/SKILL.md
├── gsap/SKILL.md
├── wordpress-theme/SKILL.md
└── acf-elements/SKILL.md
```

## Usage

Add this repo to a project as a git submodule on `.claude/skills` — the skills are then picked up directly and loaded on demand, without copying or symlinking them:

```sh
git submodule add git@github.com:g2odev/skills.git .claude/skills
```

- **Skills**: available as `.claude/skills/<name>/SKILL.md` as soon as the submodule is added
- **AGENTS.md**: arrives as `.claude/skills/AGENTS.md`; link it into the project root so it is loaded automatically on every request — `ln -s .claude/skills/AGENTS.md AGENTS.md` (and `ln -s AGENTS.md CLAUDE.md` for Claude Code)
- **Note**: the skill links in AGENTS.md point to `.claude/skills/…`, i.e. the submodule path as seen from the project root — they resolve in a project, not inside this repo
- **Cloning a project**: `git clone --recurse-submodules …`, or `git submodule update --init` in an existing clone
- **Updating**: `git submodule update --remote .claude/skills`, then commit the new submodule pointer

## License

[MIT](LICENSE)