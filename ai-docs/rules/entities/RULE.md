---
description: Rules for creating JPA entities with Jmix annotations
globs: ["**/entity/**"]
alwaysApply: false
---

# Jmix Entity Rules

## Scope
Applies to `src/main/java/**/entity/`

## Required Annotations
- `@JmixEntity` + `@Entity` + `@Table` (both JmixEntity AND Entity!)
- ID: `UUID` with `@JmixGeneratedValue` (application-level, not DB)
- `@Version` field (Integer, not null)
- `@InstanceName` on display field or method

## @InstanceName
- On field (simple): `@InstanceName` on String field
- On method (computed): `@InstanceName` + `@DependsOnProperties({"field1", "field2"})`

## Relationships
- `@ManyToOne(fetch = LAZY)` — always LAZY!
- `@OneToMany(mappedBy = "...")`
- Composition: `@Composition` + `@OnDelete(CASCADE)`
- Delete policies: CASCADE, DENY, SET_NULL, UNLINK

## Optional Fields
- Audit: `@CreatedBy`, `@CreatedDate`, `@LastModifiedBy`, `@LastModifiedDate`
- Soft delete: `@DeletedBy`, `@DeletedDate`
- Embedded: `@Embedded` + `@EmbeddedParameters(nullAllowed=false)` + `@Embeddable` class
- Transient: `@JmixProperty(mandatory=true)` + `@Transient` on getter
- Multi-store: `@Store(name = "storeName")` + prefix in `@Entity(name = "store_EntityName")`

## Inheritance
- Use JPA `@Inheritance(strategy = InheritanceType.JOINED)`
- Add `@DiscriminatorColumn` + `@DiscriminatorValue`

## Validation
- `@NotNull`, `@Email`, `@Size`, `@Pattern`

## After Creating Entity
1. Create Liquibase changelog (UUID PK, VERSION column)
2. Add include to `changelog.xml`
3. Add messages to `messages_en.properties`

## Forbidden
- Lombok (`@Data`, `@Getter`, etc.) - any annotations forbidden
- `@GeneratedValue` (use `@JmixGeneratedValue`)
- `FetchType.EAGER` on relationships
- EntityManager for regular CRUD (use DataManager)
- Business logic in entities
