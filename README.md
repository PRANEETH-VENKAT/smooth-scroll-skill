# smooth-scroll

A Claude skill for adding smooth scrolling to websites using **Lenis** or **GSAP ScrollSmoother**. It picks the right library for the project, sets it up (plain HTML or React), and runs a checklist for common scroll bugs such as broken `position: fixed`, anchor links, modals, and nested scroll areas.

## Contents

```
smooth-scroll/
├── SKILL.md                         # Main instructions and library decision
└── references/
    ├── lenis.md                     # Lenis setup and patterns
    └── gsap-scrollsmoother.md       # ScrollSmoother setup and patterns
```

## Install

**Claude.ai / Claude app:** zip the `smooth-scroll` folder (so `SKILL.md` sits inside it), then upload it under Settings > Capabilities > Skills.

```bash
zip -r smooth-scroll.skill smooth-scroll
```

**Claude Code:** copy the folder into your skills directory.

```bash
cp -r smooth-scroll ~/.claude/skills/        # personal
cp -r smooth-scroll .claude/skills/          # per project
```

## Usage

Ask Claude things like "make my site scroll smoothly", "add parallax on scroll", or "my sticky header breaks with Lenis". The skill triggers automatically.

## License

MIT. See [LICENSE](LICENSE).
