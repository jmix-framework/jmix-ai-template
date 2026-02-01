# Jmix AI-Assisted Development Template

A template project with pre-configured AI coding assistant rules for Jmix 2.7.x applications.

## Prerequisites

- Java 17+
- Node.js 18+ (for MCP servers, optional)
- IntelliJ IDEA with Jmix Studio plugin

## Quick Start

```bash
./gradlew bootRun     # http://localhost:8080 (admin/admin)
./gradlew test        # Run tests
```

## Agentic AI Setup

### Junie (Recommended)

Recommended for IntelliJ IDEA users due to native IDE and Jmix Studio integration.

Junie reads `.junie/guidelines.md` by default. To use the universal `AGENTS.md` instead:

**Option A: Prompt prefix**

```
Read AGENTS.md first, then proceed with the task. <Your prompt here for task>
```

**Option B: Change default path**

```
Settings → Tools → Junie → Project Settings → Guidelines path → set AGENTS.md (instead of .junie/guidelines.md)
```

### Other Tools

| Tool           | Auto-reads                    | Notes                                        |
|----------------|-------------------------------|----------------------------------------------|
| Claude Code    | `CLAUDE.md`                   | Case-sensitive, only context file for vendor |
| Codex (OpenAI) | `AGENTS.md`                   | Works out of the box                         |
| Continue.dev   | `AGENTS.md`, `.continuerules` | Also, works out of the box                   |

If your tool uses a different config file and you don't want to always add pre-prompt about `AGENTS.md`, copy `AGENTS.md` content to that file (e.g., `CLAUDE.md`, `.cursorrules`) - or rename it to needing file.

## MCP Setup (Optional)

MCP servers enable AI to verify code via IDE and browser. Requires Node.js 18+.

**Step 1: Enable MCP Server in IntelliJ IDEA (required for JetBrains MCP)**
```
Settings → Tools → MCP Server → Enable MCP Server ✓
```

**Step 2: Configure MCP in your AI tool**

**Junie:**

`Settings → Tools → Junie → MCP Settings → Edit mcp.json:`
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    }
  }
}
```
Note: Junie runs inside IDEA, so JetBrains MCP is not needed. 

**Claude Code:**
```bash
claude mcp add jetbrains -- npx -y @jetbrains/mcp-proxy
claude mcp add playwright -- npx -y @playwright/mcp
```

**Codex (OpenAI):**
```bash
codex mcp add jetbrains -- npx -y @jetbrains/mcp-proxy
codex mcp add playwright -- npx -y @playwright/mcp
```

**Optional:**
- [Context7 MCP](https://github.com/upstash/context7-mcp) can be used for Jmix documentation search.


**Step 3: Verify MCP is working**

Ask your AI agent:
```
Do you have access to JetBrains MCP? Can you run get_file_problems()?
Do you have access to Playwright MCP? Can you take a screenshot?
```
If configured correctly, the agent will confirm available tools.

## Example Workflow

```
You: Create a Customer entity with name and email fields, and list/detail views

AI: 1. Reads AGENTS.md (auto or via prompt prefix)
    2. Reads ai-docs/skills/entities/SKILL.md, ai-docs/skills/views/SKILL.md
    3. Creates Customer.java with @JmixEntity, UUID, @Version, @InstanceName
    4. Creates Liquibase changelog, adds to changelog.xml
    5. Creates CustomerListView.java + XML, CustomerDetailView.java + XML
    6. Updates menu.xml
    7. Adds messages to ALL locale files
    8. Validates against ai-docs/rules/entities/RULE.md, ai-docs/rules/views/RULE.md
    9. Runs get_file_problems() via JetBrains MCP (if configured)
   10. Runs tests
   11. Opens browser via Playwright MCP, navigates to view, verifies UI works (if configured)
```

**After AI completes the task**, you can ask for additional verification:
```
Check all files for IDE errors using get_file_problems()
Run the app and verify Customer views work via Playwright
Run all tests and show me the results
```

The AI reads the relevant skill (`ai-docs/skills/{layer}/SKILL.md`) 
and applies rules (`ai-docs/rules/{layer}/RULE.md`) based on the task type (layer = entities or views, services, etc.).

## What's Inside

```
ai-docs/
├── skills/          # Step-by-step guides (entities, views, services...)
└── rules/           # Constraints applied by file path context
```

**Skills** — read when agent needs to perform a task (e.g., create entity)
**Rules** — auto-applied based on file path (e.g., `**/entity/**` triggers entity rules)

See [AGENTS.md](AGENTS.md) for the complete AI context file.

## Copy to Your Project

1. Copy `AGENTS.md` to your project root (or rename to your tool's config file)
2. Copy `ai-docs/` folder
