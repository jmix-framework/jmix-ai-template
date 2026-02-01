---
description: Rules for creating database changelogs for Jmix entities
globs: ["**/liquibase/**"]
alwaysApply: false
---

# Jmix Liquibase Rules

## Scope
Applies to `src/main/resources/**/liquibase/**`

## File Naming
- Pattern: `NNN-entity-name.xml` (e.g., `020-customer.xml`)
- Numbers control execution order
- Use descriptive names

## Changeset ID Convention
- Format: `NNN-entity-N` or `YYYYMMDD-HHMM-topic`
- Must be globally unique across all files
- Bad: `id="1"` (collision risk)
- Good: `id="020-customer-1"` or `id="20260112-1430-create-customer"`
- Author: use project name or username (not generic `dev`)

## Required Columns (Every Table)
```xml
<column name="ID" type="${uuid.type}">
    <constraints primaryKey="true" nullable="false"/>
</column>
<column name="VERSION" type="int">
    <constraints nullable="false"/>
</column>
```

## Audit Columns (If Entity Uses @CreatedBy etc.)
```xml
<column name="CREATED_BY" type="varchar(255)"/>
<column name="CREATED_DATE" type="timestamp"/>
<column name="LAST_MODIFIED_BY" type="varchar(255)"/>
<column name="LAST_MODIFIED_DATE" type="timestamp"/>
```

## Column Types
- UUID: `${uuid.type}` (cross-database)
- String: `varchar(n)`
- Integer: `int`
- Long: `bigint`
- BigDecimal: `decimal(p,s)`
- Boolean: `boolean`
- LocalDate: `date`
- LocalDateTime: `timestamp`
- Enum: `varchar(50)` or `int`

## Contexts
- Use `context` for environment-specific changes
- Test data: `context="dev"` or `context="test"`
- Exclude from prod: `context="!prod"`

## Rollback
- Always provide explicit `<rollback>` for DDL
- Create compensating changesets for production (never modify applied ones)

## Naming Conventions
- Tables: `UPPER_SNAKE_CASE`
- Columns: `UPPER_SNAKE_CASE`
- FK: `FK_TABLE_REFERENCE`
- Index: `IDX_TABLE_COLUMN`

## Reserved Words (Add Underscore)
- `ORDER` → `ORDER_`
- `USER` → `USER_`
- `GROUP` → `GROUP_`

## ⚠️ Jmix Studio Note

**Jmix Studio applies Liquibase BEFORE application starts!**

When user clicks "Run" in IDE:
1. Studio detects changelog changes
2. Applies them to database automatically
3. Then starts the application

If you have changelog errors, app won't start. Always verify:
- Changelog syntax is correct
- `changelog.xml` includes your new file

## Forbidden
- `type="UUID"` (use `${uuid.type}`)
- `id="1"` without file prefix (collision)
- `author="dev"` (use meaningful name)
- Auto-increment IDs (use UUID)
- Missing VERSION column
- Modifying applied changesets
