# Skills

A collection of agent skills and instructions for building WordPress themes (Roots Sage, PHP 8.4) with Tailwind CSS, Alpine.js and GSAP.

## Contents

| Item | Description |
|---|---|
| [AGENTS.md](AGENTS.md) | Always-on agent instructions: tech stack (Tailwind, Alpine, GSAP, Sage/WordPress), rules, layout, responsive, styleguide, Gravity Forms, accessibility and image conventions. Copy to the root of a project. |
| [tailwindcss](.github/skills/tailwindcss/SKILL.md) | Tailwind CSS v4 conventions and best practices |
| [alpinejs](.github/skills/alpinejs/SKILL.md) | Alpine.js conventions and best practices |
| [gsap](.github/skills/gsap/SKILL.md) | GSAP animation conventions and best practices |
| [wordpress-theme](.github/skills/wordpress-theme/SKILL.md) | WordPress theme development with Sage, Blade and PHP 8.4 |
| [acf-elements](.github/skills/acf-elements/SKILL.md) | Make Blade elements manageable through ACF flexible content via local JSON |

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

The layout mirrors a real project, so everything can be copied 1-on-1:

```
skills/
├── AGENTS.md
└── .github/
    └── skills/
        └── skill-name/
            └── SKILL.md
```

## Usage

- **AGENTS.md**: copy to the root of your project; it is loaded automatically on every request
- **Skills**: copy or symlink `.github/skills/` into your project — they are loaded on demand
- **Note**: the skill links in AGENTS.md point to `.github/skills/…` — the same layout as this repo, so copying AGENTS.md and `.github/skills/` together keeps them working; only update the links if the project stores its skills somewhere else

## License

[MIT](LICENSE)