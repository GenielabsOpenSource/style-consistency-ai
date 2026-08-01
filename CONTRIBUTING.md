# Contributing to Style Consistency AI

Thank you for your interest in contributing! This project is community-driven and welcomes all skill levels.

## Ways to Contribute

### 🐛 Bug Reports
Found a skill instruction that misleads the agent, a broken provider snippet, or a workflow step that doesn't hold up in practice? Open an issue with:
- Which skill and section (file + heading)
- What you asked the agent to do
- What happened vs. what you expected
- The model/provider involved, if relevant

### ✨ New Skills & Improvements
Before opening a large PR, open an issue first to discuss. Some ideas we'd love to see:

- **The planned skills** — `background-consistency`, `object-consistency`, `video-consistency`
- **New provider references** for `model-integration` — Midjourney, Ideogram, Recraft, local ComfyUI / Stable Diffusion
- **Better prompting patterns** — delta-prompting techniques that hold identity on specific models
- **Before/after examples** — a real drift problem fixed by a rung of the ladder
- **Eval ideas** — ways to *measure* whether a generation is on-model

### 📖 Documentation
Documentation PRs are always welcome — fix typos, clarify instructions, add examples.

---

## Getting Started

```bash
git clone https://github.com/your-username/style-consistency-ai
cd style-consistency-ai
```

The skills are plain Markdown — no build step, no dependencies. To test a change, copy the skill into a skills-aware agent (e.g. `~/.claude/skills/` for Claude Code) and run a real generation task through it.

## Pull Requests

1. Fork the repo and create a branch from `main`
2. Keep each PR focused on one skill or one concern
3. Match the existing structure: `SKILL.md` frontmatter (`name`, `description`) + `references/` for on-demand detail
4. All PRs require maintainer approval before merge

## License

By contributing, you agree that your contributions will be licensed under the same [PolyForm Noncommercial 1.0.0](LICENSE) license that covers the project.
