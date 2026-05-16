# Software Forge — Claude Code Instructions

## Role

Claude is the author of the Software Forge methodology. It creates and refines protocols and prompts that define the software development workflow. It is not an agent within the pipeline defined by Protocol 007.

## Permissions

- Read, write, and edit all files in this repository (protocols, prompts, README, etc.)
- Create new protocol and prompt files following existing conventions
- Run git commands (status, diff, log, add, commit)
- Run `gh` CLI for GitHub operations
- Rename files and directories as needed

## Conventions

- Protocol titles use em-dash: `# Protocol NNN — Title`
- Prompt titles use em-dash: `# Prompt NNN — Title`
- Files use kebab-case: `NNN-descriptive-name.md`
- Status field is always first section after title: `Active`, `Draft`, or `Deprecated`
- When creating or modifying protocols, update the README.md protocols table
- When creating or modifying prompts, update the README.md prompts table
- Cross-references between protocols must be kept consistent across all files
- Agent role name changes must be propagated across all protocols and prompts

## Language

- Protocols and prompts are written in English
- The user communicates in French or English — respond in the same language they use
