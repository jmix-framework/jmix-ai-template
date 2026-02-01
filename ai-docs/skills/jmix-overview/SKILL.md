---
name: jmix-overview
description: Understanding Jmix framework basics — what makes it different from plain Spring Boot and key concepts to know.
---

# Jmix Overview

## When to Use
- When you need to understand what Jmix is and how it differs from Spring Boot
- When you're new to the project and need context
- When deciding which Jmix component to use

## What is Jmix
High-productivity platform (Spring Boot + Vaadin Flow) for enterprise applications.
Jmix provides ready-to-use components for data access, UI, security, and more.

## Jmix vs Plain Spring Boot

| Aspect | Spring Boot | Jmix |
|--------|-------------|------|
| Entity | `@Entity` | `@JmixEntity` + `@Entity` |
| ID | Long / UUID | UUID + `@JmixGeneratedValue` |
| Data Access | JpaRepository / EntityManager | `DataManager`, `JmixJpaRepository` |
| UI | REST + SPA | Vaadin Flow (XML + Java) |
| Security | Manual / Spring Security | `@ResourceRole`, `@RowLevelRole` |
| DB Migrations | Flyway / manual | Liquibase (auto-generated) |

## Key Concepts

### Entities
- `@JmixEntity` + `@Entity` annotations together
- `@InstanceName` on display field (shown in UI references)
- `@Version` for optimistic locking
- UUID primary key with `@JmixGeneratedValue`

### Data Access
- **`DataManager`** — primary way, supports fetch plans, security-aware
- **`UnconstrainedDataManager`** — same as DataManager but bypasses security
- **`JmixJpaRepository`** — Spring Data style, but limited native query support
- **`EntityManager`** — standard JPA, use for native SQL queries when needed
- Fetch plans control lazy loading
- LoadContext for complex queries

### Views (UI)
- XML descriptor + Java controller pair
- `StandardListView<T>` for lists
- `StandardDetailView<T>` for edit forms
- Data containers bind data to UI

### Security
- `@ResourceRole` for entity/view/menu access
- `@RowLevelRole` for row-level filtering
- Policies: `@EntityPolicy`, `@ViewPolicy`, `@MenuPolicy`

### Database
- Liquibase for migrations
- UUID primary key, VERSION column
- Include changelogs in `changelog.xml`

## Project Structure
```
src/main/java/.../
├── entity/           # @JmixEntity classes
├── view/             # UI controllers
├── security/         # Roles
└── service/          # Business logic

src/main/resources/.../
├── view/             # XML descriptors
├── liquibase/        # DB changelogs
├── menu.xml          # App menu
└── messages_en.properties
```

## Development Workflow

### Verification
After writing code:
1. **Run application** (`./gradlew bootRun`) — Jmix Studio applies Liquibase before start
2. **Test in browser** (http://localhost:8080) — click through all functionality

### ⚠️ Notes
- Jmix Studio applies Liquibase BEFORE app starts — changelog errors block startup
- Views have both design-time and runtime errors — always test in browser

### MCP Tools (Optional)
If available:
- `idea-mcp` — check entities/views for missing i18n
- `jmix-rag-mcp-search` — search Jmix docs instead of web
