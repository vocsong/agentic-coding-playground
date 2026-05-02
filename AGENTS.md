# Agent Instructions

## Skill creation workflow

When the user asks to create a skill, follow this requirements gathering workflow before writing anything:

1. **Goal** — What problem does the skill solve? Who is it for?
2. **Inputs** — What does the user provide? What context does the skill need?
3. **Outputs** — What does the skill produce? What does success look like?
4. **Steps** — Walk through the skill step-by-step. What tools/commands are needed at each step?
5. **Edge cases** — What can go wrong? What are the boundaries?

Ask clarifying questions iteratively until all details are clear. Only then write the skill.

## Publishing a skill

- Skills are published to `skills/<name>/`
- Each skill must include `skills.md` with the full explanation, structure, and workflow
- Do not invent placeholder skills — only create a skill when the requirements are fully understood
