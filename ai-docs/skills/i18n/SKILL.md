---
name: i18n
description: Working with message bundles (i18n/l10n) — adding translations, naming conventions, using messages in views.
---

# Internationalization (i18n)

## When to Use
- When creating any UI view
- When adding new entity fields or enums
- When adding buttons, actions, or notifications

## File Locations
```
src/main/resources/com/company/project/
├── messages.properties        # Default (English)
├── messages_ru.properties     # Russian
├── messages_de.properties     # German (if needed)
└── ...
```

## Critical Rule
**Every message key MUST exist in ALL locale files.**

## Naming Conventions

### Views
```properties
orderListView.title=Orders
orderDetailView.title=Order
```

### Entity Fields
```properties
Order.number=Order Number
Order.status=Status
Order.customer=Customer
```

### Enums
```properties
OrderStatus.NEW=New
OrderStatus.PROCESSING=Processing
OrderStatus.COMPLETED=Completed
```

### Actions
```properties
save=Save
saveAndClose=Save and Close
cancel=Cancel
create=Create
edit=Edit
remove=Remove
```

### Filters
```properties
filter.type=Type
filter.status=Status
filter.reset=Reset
```

## Usage in XML
```xml
<view title="msg://orderListView.title">
    <button text="msg://create"/>
    <column property="status" header="msg://Order.status"/>
</view>
```

## Usage in Java

### In Views (MessageBundle)
Auto-detects message group from view class — no class parameter needed.
```java
@ViewComponent
private MessageBundle messageBundle;

String title = messageBundle.getMessage("orderListView.title");
String formatted = messageBundle.formatMessage("items.count", count);  // for %s params
// messages.properties: items.count=Found %s items
```

### In Services/Beans (Messages)
Requires class parameter to identify message group.
```java
@Autowired
private Messages messages;

String msg = messages.getMessage(OrderService.class, "order.created");
String formatted = messages.formatMessage(OrderService.class, "order.total", total);
```

## Workflow: Adding New Messages

1. Add to `messages.properties`:
```properties
myNewView.title=My New View
```

2. Add to ALL other locale files:
```properties
# messages_ru.properties
myNewView.title=Мой новый экран

# messages_de.properties
myNewView.title=Meine neue Ansicht
```

3. Use in XML:
```xml
<view title="msg://myNewView.title">
```

## Checklist
- [ ] Key follows naming convention
- [ ] Added to ALL locale files
- [ ] Used `msg://` prefix in XML
- [ ] No duplicate keys

## Forbidden
- Adding message to only ONE locale file
- Hardcoded text in XML views
- Inconsistent naming patterns
