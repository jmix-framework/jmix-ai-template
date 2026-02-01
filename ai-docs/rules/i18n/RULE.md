---
description: Rules for message bundles (i18n/l10n)
globs: ["**/messages*.properties"]
alwaysApply: false
---

# i18n Rules

## Scope
Applies to `src/main/resources/**/messages*.properties`

## File Structure
```
src/main/resources/com/company/project/
├── messages.properties      # Default (English)
├── messages_ru.properties   # Russian
├── messages_de.properties   # German
└── ...                      # Other locales
```

## Critical Rule
**Every message key MUST exist in ALL locale files.** No exceptions.

## Naming Conventions
| Type | Pattern | Example |
|------|---------|---------|
| View title | `{viewId}.title` | `orderListView.title=Orders` |
| Entity | `{Entity}` | `Order=Order` |
| Field | `{Entity}.{field}` | `Order.status=Status` |
| Enum | `{Enum}.{VALUE}` | `OrderStatus.NEW=New` |
| Action | camelCase | `saveAndClose=Save and Close` |
| Filter | `filter.{name}` | `filter.status=Status` |
| Validation | `validation.{rule}` | `validation.required=Required` |

## Usage in XML
```xml
<view title="msg://orderListView.title">
<column property="status" header="msg://Order.status"/>
<button text="msg://save"/>
```

## Usage in Java
```java
// In Views — auto-detects message group from view class
@ViewComponent
private MessageBundle messageBundle;
String msg = messageBundle.getMessage("key");
String formatted = messageBundle.formatMessage("key", param1, param2);  // for %s

// In Services/Beans — requires class parameter
@Autowired
private Messages messages;
String msg = messages.getMessage(MyClass.class, "key");
String formatted = messages.formatMessage(MyClass.class, "key", param1);
```

## Forbidden
- Adding message to only ONE locale file
- Hardcoded text in XML (use `msg://` prefix)
- Inconsistent naming patterns
