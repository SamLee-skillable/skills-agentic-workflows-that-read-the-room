# Project guidance for Claude Code

## GitHub Agentic Workflows (gh-aw)

This repo is set up to author and run [GitHub Agentic Workflows](https://github.com/github/gh-aw) (gh-aw) using Claude Code — no GitHub Copilot required.

### Two layers

- **Authoring (local):** Use the `.claude/` skills and agent to design, create, debug, and upgrade workflow files:
  - `.claude/agents/agentic-workflows.md` — dispatcher agent that routes to the right gh-aw prompt
  - `.claude/skills/agentic-workflows/SKILL.md` — router skill
  - `.claude/skills/agentic-workflow-designer/SKILL.md` — interview-driven workflow designer
- **Execution (CI):** Workflows run in GitHub Actions. Set `engine: claude` in each workflow's frontmatter to run on Claude instead of the default Copilot engine, and add an `ANTHROPIC_API_KEY` repo secret.

### Prerequisites

The `github-agentic-workflows` MCP server (configured in `.mcp.json`) and the `gh aw` commands require the gh-aw CLI extension:

```bash
gh extension install github/gh-aw
```

### Key commands

```bash
gh aw init                      # initialize repo for agentic workflows
gh aw compile [workflow-name]   # generate the .lock.yml file
gh aw run <workflow-name>       # trigger a workflow on demand (preferred over gh workflow run)
gh aw logs [workflow-name]      # inspect runs
gh aw fix --write               # apply upgrade codemods
```

### Conventions

- Workflow source lives in `.github/workflows/*.md`; compiled `.lock.yml` files are generated (see `.gitattributes`) — edit the `.md`, never the `.lock.yml` directly.
- Keep agent-job permissions read-only; perform all writes via gh-aw `safe-outputs`.
- Reference docs under `.github/aw/*.md` (created by `gh aw init`) for syntax, safe outputs, networking, and patterns.
