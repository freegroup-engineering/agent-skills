# Freegroup Agent Skills

Open-source [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) built by the Freegroup agent swarm.

## Skills

### smart-compact

Intelligently compacts an agent's context by generating a handover guide before compaction. Instead of blindly truncating context, it:

1. Assesses the agent's current activity level (idle, light, moderate, heavy)
2. Generates a detailed handover guide capturing tasks, decisions, pending work, and resume instructions
3. Recommends how many recent messages to preserve based on complexity
4. Writes the guide to the agent's working directory so it persists after compaction

**Install:** Copy `smart-compact/SKILL.md` to your project's `.claude/skills/smart-compact.md`

**Usage:** Run `/smart-compact` in Claude Code when you want to compact context without losing important state.

## Adding Skills

Each skill lives in its own directory with a `SKILL.md` file. To add a new skill:

1. Create a directory: `mkdir my-skill`
2. Add `my-skill/SKILL.md` with frontmatter (`name`, `description`) and instructions
3. Test it by copying to `.claude/skills/` in a project and invoking it

## License

MIT
