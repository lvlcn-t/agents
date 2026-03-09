# AGENTS.md

Guidelines for agentic coding assistants working in this repository.

## What this repo is

opencode configuration: MCP servers, custom AI agents, and slash commands.
The primary content is Markdown — agent system prompts in `agents/` and
slash command definitions in `command/`. There is no application code,
build pipeline, or test suite.

## Linting

Markdown is the only lintable artifact. The linter is
[markdownlint-cli2](https://github.com/DavidAnson/markdownlint-cli2).

```bash
# Lint all Markdown files (uses .markdownlint-cli2.yaml automatically)
npx markdownlint-cli2@0.21.0 "**/*.md"

# Lint a single file
npx markdownlint-cli2@0.21.0 path/to/file.md

# Run via pre-commit (requires pre-commit installed)
pre-commit run markdownlint-cli2 --all-files
```

There are no unit tests. CI runs the lint command above on every push and
pull request (see `.github/workflows/markdownlint.yml`).

## Repository structure

```text
opencode.json        → MCP servers, providers, built-in agent model assignments
agents/              → Custom agent system prompts (one file per agent)
command/             → Slash commands (invoked as /command-name in opencode)
docs/                → How-to guides for maintainers
agent-frontmatter.cjs → Custom markdownlint rule (do not rename)
.markdownlint-cli2.yaml → Root lint config (applies repo-wide)
```

Subdirectory overrides live alongside the files they govern:

- `agents/.markdownlint-cli2.yaml` — disables MD041 (files start with
  YAML frontmatter, not a heading)
- `command/.markdownlint-cli2.yaml` — same override for command files

## Markdown style rules

These are enforced by `.markdownlint-cli2.yaml`. Violations fail CI.

| Rule  | Requirement                                                 |
| ----- | ----------------------------------------------------------- |
| MD003 | ATX-style headings only (`##`, never underline style)       |
| MD004 | Unordered lists use `-` (not `*` or `+`)                    |
| MD007 | List indent is 2 spaces                                     |
| MD013 | 80-character line limit for headings and prose; code blocks |
|       | and tables are exempt                                       |
| MD022 | Blank lines required before and after every heading         |
| MD029 | Ordered lists must use sequential numbers (`1. 2. 3.`)      |
| MD033 | No inline HTML                                              |
| MD034 | No bare URLs — use `[text](url)` or reference links         |
| MD040 | Fenced code blocks must declare a language                  |
| MD041 | First line must be a top-level heading (waived in `agents/` |
|       | and `command/` where frontmatter comes first)               |
| MD046 | Code blocks must use fences, not indentation                |

### Additional conventions

- **Single H1** per document.
- **Reference links** for long or repeated URLs — place them at the
  bottom of the file.
- **Informative link text** — never "click here" or bare "link".
- **Callouts** — use only cross-platform alert types:
  `[!NOTE]`, `[!TIP]`, `[!IMPORTANT]`, `[!WARNING]`, `[!CAUTION]`.
  Do not use Obsidian-only types (`[!abstract]`, `[!info]`, etc.).
- **80-char source limit** — wrap prose at 80 columns in the source
  file. This is `strict: true`; headings are included.

## Agent files (`agents/*.md`)

Every file in `agents/` must open with a YAML frontmatter block before
any Markdown content. Three fields are required — the custom lint rule
`agent-frontmatter-fields` enforces this at line 1.

```markdown
---
name: my-agent
description: One sentence: what this agent does and when to invoke it.
model: github-copilot/claude-sonnet-4.6
---

System prompt body starts here.
```

### Required frontmatter fields

| Field         | Notes                                                    |
| ------------- | -------------------------------------------------------- |
| `name`        | Must match the filename stem exactly                     |
| `description` | Shown in the UI; one sentence, include when to invoke it |
| `model`       | See model table below                                    |

### Optional frontmatter fields

| Field        | Notes                                           |
| ------------ | ----------------------------------------------- |
| `color`      | Hex color, e.g. `"#22c55e"` (opencode UI label) |
| `hidden`     | `true` hides from UI; invoke only via `@name`   |
| `mode`       | `subagent`; pair with `hidden: true`            |
| `permission` | Tool permission overrides (see below)           |

### Model choices

| Model                              | Use for                                   |
| ---------------------------------- | ----------------------------------------- |
| `github-copilot/claude-sonnet-4.6` | General purpose — default for most agents |
| `azure-anthropic/claude-opus-4-6`  | Deep reasoning: security, architecture    |
| `github-copilot/gpt-5.3-codex`     | Code generation and transformation        |

### Permissions

Agents inherit global tool settings unless overridden via `permission`:

```yaml
permission:
  edit: deny            # deny a tool entirely
  write: allow          # allow unconditionally
  webfetch: allow
  bash:
    "*": ask            # ask for any bash command ...
    "git diff": allow   # ... except these specific patterns
    "git log*": allow
  task:
    security-review: allow  # allow delegating to a named sub-agent
```

Agents that delegate to sub-agents must explicitly grant
`task: <agent-name>: allow` for each sub-agent.

## Slash command files (`command/*.md`)

Command files also use YAML frontmatter. MD041 is disabled here too.
Required fields: `description` and `agent`.

```markdown
---
description: Short description shown in the command palette
agent: review
---

Instructions passed to the agent when this command is invoked.
```

## Writing agent system prompts

- **One agent, one responsibility.** If you find yourself writing
  "and also…", consider whether the second responsibility belongs in
  a separate sub-agent.
- **Delegate explicitly.** Use `@agent-name` in the prompt body to
  signal where sub-agents should be called.
- **Be specific about when to invoke.** The `description` field is
  passed to the model as context — write it for the model, not just
  the human.

## See also

- [Adding a new agent](docs/adding-an-agent.md)
- [Markdown linting](docs/linting.md)
- [opencode agent configuration](https://opencode.ai/docs/agents)
- [markdownlint rules reference][ml-rules]

[ml-rules]: https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md
