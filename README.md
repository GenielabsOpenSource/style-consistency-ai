# style-consistency-visual-ai

A collection of agent **skills for preserving style consistency in visual generation** — keeping characters, art styles, and visual identity on-model across AI image and video generations.

## Skills

| Skill | Purpose |
|-------|---------|
| [`character-consistency`](skills/character-consistency/SKILL.md) | Maintain character identity and style accuracy across generations and edits with multimodal image models (Nano Banana, GPT Image, Flux Kontext, Seedream, etc.). Uses a ladder of approaches from a one-line corrective guideline up to a fine-tuned model, always starting at the cheapest rung that solves the problem. |

## Layout

```
skills/
└── <skill-name>/
    ├── SKILL.md          # entry point: description + core workflow
    └── references/       # progressive-disclosure detail loaded on demand
```

Each skill follows the [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills) convention: a `SKILL.md` with YAML frontmatter (`name`, `description`) plus optional reference files the agent reads only when needed.

## Using these skills

Point Claude Code (or any skills-aware agent) at this repo, or copy a skill directory into your agent's skills path. The `description` frontmatter controls when the skill auto-triggers.
