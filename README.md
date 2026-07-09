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

## Structure

Each skill lives in its own folder containing a `SKILL.md` file with instructions and any supporting files. The layout mirrors a real project, so everything can be copied 1-on-1:

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
- **Note**: the skill links in AGENTS.md are relative to this repo (`tailwindcss/SKILL.md`); after copying to a project, update them to match the skills location there (e.g. `.github/skills/tailwindcss/SKILL.md`)

## License

[MIT](LICENSE)