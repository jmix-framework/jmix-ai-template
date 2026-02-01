---
name: liquibase
description: Creating database changelogs for Jmix entities — table creation, foreign keys, indexes, and rollback strategies.
---

# Liquibase (Database Migrations)

## When to Use
- When creating a new entity and need database table
- When adding columns, indexes, or foreign keys
- When modifying database schema

## Changelog Template
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-latest.xsd">

    <changeSet id="020-customer-1" author="yourproject">
        <createTable tableName="CUSTOMER">
            <column name="ID" type="${uuid.type}">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="VERSION" type="int">
                <constraints nullable="false"/>
            </column>
            <column name="NAME" type="varchar(255)">
                <constraints nullable="false"/>
            </column>
            <column name="EMAIL" type="varchar(255)"/>
        </createTable>
        <rollback>
            <dropTable tableName="CUSTOMER"/>
        </rollback>
    </changeSet>
</databaseChangeLog>
```

## Changeset ID Convention
- **Format:** `NNN-entity-N` (e.g., `020-customer-1`) or `YYYYMMDD-HHMM-topic`
- **Author:** Use project name (NOT generic `dev`)
- **Never modify applied changesets**

## Column Types

| Java Type | Liquibase Type |
|-----------|----------------|
| UUID | `${uuid.type}` |
| String | `varchar(n)` |
| Integer | `int` |
| Long | `bigint` |
| BigDecimal | `decimal(p,s)` |
| Boolean | `boolean` |
| LocalDate | `date` |
| LocalDateTime | `timestamp` |
| Enum | `varchar(50)` |

## Required Columns
```xml
<column name="ID" type="${uuid.type}">
    <constraints primaryKey="true" nullable="false"/>
</column>
<column name="VERSION" type="int">
    <constraints nullable="false"/>
</column>
```

## Foreign Keys
```xml
<addForeignKeyConstraint
    baseTableName="ORDER_"
    baseColumnNames="CUSTOMER_ID"
    constraintName="FK_ORDER_CUSTOMER"
    referencedTableName="CUSTOMER"
    referencedColumnNames="ID"/>
```

## Include in changelog.xml
```xml
<include file="020-customer.xml" relativeToChangelogFile="true"/>
```

## Contexts
```xml
<changeSet id="..." author="..." context="dev">
    <!-- Only runs in dev environment -->
</changeSet>
```

## ⚠️ Jmix Studio Note

**Jmix Studio applies Liquibase BEFORE application starts!**

When user clicks "Run" in IDE:
1. Studio detects changelog changes
2. Applies them to database automatically
3. Then starts the application

If you have changelog errors, app won't start. Always verify:
- Changelog syntax is correct
- `changelog.xml` includes your new file

### Common Errors
- `Table already exists` → clean DB: `rm -rf .jmix/hsqldb/*`
- `Duplicate changeset ID` → use `NNN-entity-N` format
- `Invalid type UUID` → use `${uuid.type}`

## Forbidden
- `type="UUID"` (use `${uuid.type}`)
- `id="1"` without prefix (collision risk)
- `author="dev"` (use meaningful name)
- Missing VERSION column
- Modifying applied changesets
