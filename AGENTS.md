# AGENTS.md — Universal Agents Context File

## Agent Role
You are a Java developer expert in Jmix 2.x.x (Spring Boot + Vaadin Flow for simple).

## Project

- Type: Fullstack Jmix 2.7.x sample (Java 17, Spring Boot, Vaadin Flow)
- Database: HSQLDB (dev)
- Run: `./gradlew bootRun` (http://localhost:8080, admin/admin)

## Architecture

- Entity: `@JmixEntity`, UUID + `@JmixGeneratedValue`, `@Version`, `@InstanceName`
- Data: `DataManager` (NOT EntityManager)
- UI: Vaadin Flow Views (XML + Java), `StandardListView` / `StandardDetailView`
- Security: `@ResourceRole`, `@ViewPolicy`, `@MenuPolicy`
- DB: Liquibase changelog (UUID PK, VERSION, include in `changelog.xml`)

## Patterns

- DI: constructor injection only
- No Lombok on entities
- Business logic in services, not in views

## Dependency Injection

- **Views**: `@ViewComponent` for XML components (DataGrid, loaders, MessageBundle, DataContext)
- **Views**: `@Autowired` for Spring beans (DataManager, DialogWindows, Messages)
- **Services**: Constructor injection only

## When Asked to Create

### Entity

- Java class with UUID + Version + InstanceName
- Liquibase changelog + include in `changelog.xml`
- Messages in ALL locale files (`messages.properties`, `messages_*.properties`)

### View

- XML descriptor + Java controller
- Menu entry in `menu.xml`
- Messages for title/labels in ALL locale files

### Role

- `@ResourceRole` with entity/view/menu policies

## Validation Checklist

- Entity: UUID + Version + InstanceName present
- Changelog added to `changelog.xml`
- Messages added for all components (entity, enum, view titles, labels)
- View: XML + Java pair; menu updated
- Security: role covers entity/view/menu

## Development Workflow

After writing or modifying code, validate using this sequence:

1. **Write tests** — create/update tests for new functionality - see `ai-docs/skills/testing/SKILL.md`
2. **Check IDE problems** — if `idea-mcp` available: `get_file_problems(onlyErrors=false)` for all modified files,
   especially entities and views, enums and other jmix specific files
3. **Run tests** — `./gradlew test` to verify nothing is broken
4. **UI verification** (for views) — if `playwright-mcp` available and app is running:
    - Navigate to the view
    - Verify data displays correctly
    - Click or do things that should trigger UI logics was written
    - Test CRUD operations

**Note to user**: For full workflow support, configure MCP servers (JetBrains, Playwright).

## Forbidden

- Lombok on entities
- Field `@Autowired` (use constructor injection in services)
- EntityManager
- Business logic in views
- Edits in `frontend/generated/`
- **Hardcoded UI text** — ALL labels, titles, buttons MUST use `msg://` keys
- **Single-locale messages** — ALWAYS add to ALL locale files

## References

Detailed guides (read when performing specific tasks):

- Jmix overview: `ai-docs/skills/jmix-overview/SKILL.md`
- Entity creation: `ai-docs/skills/entities/SKILL.md`
- View creation: `ai-docs/skills/views/SKILL.md`
- Services: `ai-docs/skills/services/SKILL.md`
- Security: `ai-docs/skills/security/SKILL.md`
- Testing: `ai-docs/skills/testing/SKILL.md`
- Liquibase: `ai-docs/skills/liquibase/SKILL.md`
- DTOs: `ai-docs/skills/dto/SKILL.md`
- Enums: `ai-docs/skills/enums/SKILL.md`
- Fetch Plans: `ai-docs/skills/fetchplans/SKILL.md`
- **i18n (Messages)**: `ai-docs/skills/i18n/SKILL.md`

Context-specific rules (apply when editing files in matching paths):

- General (always): `ai-docs/rules/jmix-overview/RULE.md`
- Entities (`**/entity/**`): `ai-docs/rules/entities/RULE.md`
- Views (`**/view/**`): `ai-docs/rules/views/RULE.md`
- Services (`**/service/**`): `ai-docs/rules/services/RULE.md`
- Security (`**/security/**`): `ai-docs/rules/security/RULE.md`
- Testing (`**/test/**`): `ai-docs/rules/testing/RULE.md`
- Liquibase (`**/liquibase/**`): `ai-docs/rules/liquibase/RULE.md`
- DTOs (`**/dto/**`): `ai-docs/rules/dto/RULE.md`
- Enums: `ai-docs/rules/enums/RULE.md`
- Fetch Plans: `ai-docs/rules/fetchplans/RULE.md`
- **i18n (`**/*.properties`)**: `ai-docs/rules/i18n/RULE.md`

## Files by Vendor

- Cursor: `.cursorrules`, `ai-docs/rules/*/RULE.md`, `ai-docs/skills/*/SKILL.md`
- Claude: `CLAUDE.md`
- Codex/Agents: `AGENTS.md`
- Copilot: `.github/copilot-instructions.md`
- Continue.dev: `.continuerules`
- Junie: `.junie/guidelines.md` (or configure to use `AGENTS.md` or any other file)

## MCP (optional)

- If `jmix-rag-mcp-search` is available, use it for Jmix-specific questions instead of web search.
- If `idea-mcp` is available, use `get_file_problems(onlyErrors=false)` to check for warnings and errors in files.
