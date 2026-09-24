# skills

Claude skills for use with Claude Code and the Agent SDK.

## Skills

- [`issue-flow`](skills/issue-flow/SKILL.md): idea to shipped code via a GitHub issue, an optional design doc, and stacked PRs.
- [`ai-writing-improver`](skills/ai-writing-improver/SKILL.md): style rules for editing prose so it reads as dense, human writing instead of typical AI output.
- [`mermaid-diagram`](skills/mermaid-diagram/SKILL.md): rules for small Mermaid diagrams that sit under prose: capped boxes and arrows, caller-to-callee arrows, no arrow labels, consistent names, and zoomed-in views of one overall map.

## Usage

Copy a skill directory into `~/.claude/skills/` (personal) or a project's `.claude/skills/` (project-scoped), or point Claude Code at this repo directly.

## License

[MIT](LICENSE)
