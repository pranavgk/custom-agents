# Custom AI Agents

A collection of custom AI agents for code review, analysis, and automation — designed for use with GitHub Copilot CLI, Claude Code, or any agent framework that supports markdown prompt files.

## 🔍 Adversarial Code Review Pipeline

A 5-agent adversarial system that produces high-precision, high-recall code reviews by having agents debate each other.

```
Input (ADO PR or raw diff)
       │
       ▼
  ┌─────────┐
  │ 🦅 HAWK  │  Find ALL possible issues (maximize recall)
  └────┬────┘
       │
       ▼
  ┌──────────┐
  │ 🔬 SKEPTIC│  Challenge each finding (eliminate false positives)
  └────┬─────┘
       │
       ▼
  ┌─────────┐
  │ ⚖️ JUDGE │  Final adjudication (ground truth)
  └────┬────┘
       │
       ▼
  ┌─────────┐
  │ 🔧 FIXER │  Apply fixes for confirmed issues
  └─────────┘

  Coordinated by: 🎯 ORCHESTRATOR
```

### Agents

| Agent | File | Purpose | Reward Focus |
|-------|------|---------|--------------|
| **The Hawk** | [`adversarial-review/hawk-reviewer.md`](adversarial-review/hawk-reviewer.md) | Exhaustive issue finder | +15 security finds, +10 real bugs, -8 misread code |
| **The Skeptic** | [`adversarial-review/skeptic-reviewer.md`](adversarial-review/skeptic-reviewer.md) | False positive eliminator | +10 correct disproof, -20 dismissing real security issue |
| **The Judge** | [`adversarial-review/judge-reviewer.md`](adversarial-review/judge-reviewer.md) | Final arbiter + own findings | -15 missed real issue, -10 surviving false positive |
| **The Orchestrator** | [`adversarial-review/review-orchestrator.md`](adversarial-review/review-orchestrator.md) | Pipeline coordinator | Feeds full context through Hawk→Skeptic→Judge |
| **The Fixer** | [`adversarial-review/review-fixer.md`](adversarial-review/review-fixer.md) | Automated issue resolver | -20 introducing new bug, +10 correct fix |

### How the Reward System Works

Each agent has a **self-evaluation rubric** embedded in its prompt. Before emitting output, the agent scores itself against explicit criteria. This isn't runtime ML — it's prompt-engineered scoring that shapes reasoning behavior:

- **The Hawk** is incentivized to find real issues but penalized for vague findings
- **The Skeptic** is penalized MORE for dismissing real issues than for failing to disprove false positives
- **The Judge** is penalized for both false positives AND missed real issues, forcing careful adjudication

### Quick Start

#### 1. Set up MCP Servers

Copy `config/mcp.json.example` to your project's `.vscode/mcp.json` and fill in your ADO details:

```json
{
  "mcp": {
    "servers": {
      "azure-devops": {
        "command": "npx",
        "args": ["-y", "@tiberriver256/mcp-server-azure-devops"],
        "env": {
          "AZURE_DEVOPS_ORG_URL": "https://dev.azure.com/your-org",
          "AZURE_DEVOPS_AUTH_METHOD": "azure-identity",
          "AZURE_DEVOPS_DEFAULT_PROJECT": "your-project"
        }
      }
    }
  }
}
```

#### 2. Install Agents Globally

Copy the agent files to your global Copilot config:

```powershell
# Windows
Copy-Item adversarial-review\*.md ~\.copilot\agents\

# macOS/Linux
cp adversarial-review/*.md ~/.copilot/agents/
```

#### 3. Use

```
@review-orchestrator Review PR 12345 in repo MyRepo
```

Or review a local diff:

```
@review-orchestrator Review the current staged changes
```

### Prerequisites

- [Azure DevOps MCP Server](https://github.com/tiberriver256/mcp-server-azure-devops): `npm install -g @tiberriver256/mcp-server-azure-devops`
- Azure CLI (`az login`) for azure-identity auth, or set a PAT
- Node.js 16+

## Adding New Agents

Create a new folder under the repo root for each agent category (e.g., `security/`, `documentation/`, `testing/`). Each agent is a markdown file with YAML frontmatter:

```markdown
---
name: my-agent
description: What this agent does
tools:
  - azure-devops
  - fetch
---

# Agent Name

Your prompt here...
```

## License

MIT
