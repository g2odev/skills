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

Each skill lives in its own folder containing a `SKILL.md` file with instructions and any supporting files. The layout follows [GoogleChrome/modern-web-guidance](https://github.com/GoogleChrome/modern-web-guidance): a description that says when to load the skill, a compact `SKILL.md`, detail in `references/` and a checklist to verify the result.

**Frontmatter description.** This is the only part an agent always sees, so it decides whether the skill gets loaded. It has three parts: one sentence on what the skill covers, `Trigger on:` with the concrete keywords, function names and file names that should load it, and `Do NOT trigger for:` naming the neighbouring skill that owns the adjacent topic. Keep it a single-quoted one-liner under 1024 characters, without apostrophes.

**Outline.** Every `SKILL.md` follows the same outline, so they stay predictable to read and to extend:

| Section | Contains |
|---|---|
| `## Contents` | Table of contents for in-file navigation |
| `## Documentation` | Links to the official docs |
| `## When to use` | The situations that should trigger this skill |
| `## Core principles` | The few ideas everything else follows from |
| `## Conventions` | The actual rules, grouped in `###` subsections |
| `## Reference files` | _(only when the skill has `references/`)_ Table mapping each topic to its reference file |
| `## Anti-patterns` | Avoid/Do-instead table |
| `## Output checklist` | Checkbox list the agent verifies before delivering |
| _(optional)_ | A closing section: `## Tooling`, `## Working with …` or `## Related skills` |

**Reference files.** A `SKILL.md` holds the rules; long templates and examples that are only needed for part of the work move to `references/<topic>.md`, so they are read on demand instead of on every load. Split once a `SKILL.md` grows past roughly 300 lines — a small skill stays one file. In a split skill, `SKILL.md` keeps the rule itself in one or two sentences and links to the reference for the full example; each reference file opens with its own table of contents. Links are relative, so they resolve both in a project and inside this repo.

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
    └── acf-elements/
        ├── SKILL.md
        └── references/
            ├── templates.md           JSON and Blade templates
            ├── field-patterns.md      buttons, groups, typed elements, populated fields
            └── field-descriptions.md  default instructions and placeholders per field type
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