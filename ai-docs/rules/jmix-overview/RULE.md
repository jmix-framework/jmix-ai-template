---
description: General Jmix coding guidelines — applies to all code in Jmix project
alwaysApply: true
---

# Jmix General Rules

## Scope
Applies to all code in Jmix project

## Think Before Coding
- Understand what you're building before writing code
- Read existing code in the project first
- Follow patterns already established in codebase
- Don't overcomplicate — Jmix provides ready solutions

## Code Quality
- Write simple, readable code
- Use meaningful names (classes, methods, variables)
- Keep methods short and focused
- Don't repeat yourself — extract common logic

## Jmix Way
- Use `DataManager` for data access (not EntityManager for regular CRUD)
- Use Jmix annotations (`@JmixEntity`, `@InstanceName`, etc.)
- Use constructor injection for dependencies
- Keep business logic in services, not in views
- Use `_base` fetch plan by default, optimize later if needed

## Common Sense
- Don't load data you don't need
- Don't create abstractions for single use
- Don't add features not requested
- Test your code before committing
- If something seems wrong — ask, don't guess

## Forbidden
- **Lombok** is NOT recommended
- Business logic in views
- Editing `frontend/generated/`
- Ignoring compiler warnings
