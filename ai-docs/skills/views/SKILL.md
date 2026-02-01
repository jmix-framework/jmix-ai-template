---
name: views
description: Creating Vaadin Flow UI views — StandardListView, StandardDetailView, XML descriptors, lifecycle events, and component handling.
---

# Views (Screens)

## When to Use
- When creating a new UI screen
- When adding event handlers or renderers to dataGrid
- When opening views programmatically

## View Lifecycle (Order)

| Order | Event | Purpose |
|-------|-------|---------|
| 1 | `InitEvent` | View created, components exist but data NOT loaded |
| 2 | `InitEntityEvent<T>` | (Detail only) Entity created/loaded. Set defaults here |
| 3 | `BeforeShowEvent` | Before visible, data loaders triggered |
| 4 | `ReadyEvent` | View fully initialized and visible |
| 5 | `AfterShowEvent` | After view shown, for analytics/logging |
| 6 | `BeforeCloseEvent` | Before close, can prevent (unsaved changes) |
| 7 | `AfterCloseEvent` | After closed, cleanup |

**Detail view specific:** `ValidationEvent`, `BeforeSaveEvent`, `AfterSaveEvent`

## List View

### Java Controller
```java
package com.company.project.view.customer;

import com.company.project.entity.Customer;
import com.company.project.view.main.MainView;
import com.vaadin.flow.router.Route;
import io.jmix.flowui.view.*;

@Route(value = "customers", layout = MainView.class)
@ViewController("Customer.list")
@ViewDescriptor("customer-list-view.xml")
@LookupComponent("customersDataGrid")
@DialogMode(width = "64em")
public class CustomerListView extends StandardListView<Customer> {
}
```

### XML Descriptor
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<view xmlns="http://jmix.io/schema/flowui/view"
      title="msg://customerListView.title"
      focusComponent="customersDataGrid">
    <data readOnly="true">
        <collection id="customersDc" class="com.company.project.entity.Customer">
            <fetchPlan extends="_base"/>
            <loader id="customersDl">
                <query><![CDATA[select e from Customer e]]></query>
            </loader>
        </collection>
    </data>
    <facets>
        <dataLoadCoordinator auto="true"/>
    </facets>
    <layout>
        <hbox id="buttonsPanel" classNames="buttons-panel">
            <button id="createBtn" action="customersDataGrid.create"/>
            <button id="editBtn" action="customersDataGrid.edit"/>
            <button id="removeBtn" action="customersDataGrid.remove"/>
        </hbox>
        <dataGrid id="customersDataGrid" dataContainer="customersDc" width="100%">
            <actions>
                <action id="create" type="list_create"/>
                <action id="edit" type="list_edit"/>
                <action id="remove" type="list_remove"/>
            </actions>
            <columns resizable="true">
                <column property="name"/>
                <column property="email"/>
            </columns>
        </dataGrid>
    </layout>
</view>
```

## Detail View

### Java Controller
```java
@Route(value = "customers/:id", layout = MainView.class)
@ViewController("Customer.detail")
@ViewDescriptor("customer-detail-view.xml")
@EditedEntityContainer("customerDc")
public class CustomerDetailView extends StandardDetailView<Customer> {
}
```

### XML Descriptor
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<view xmlns="http://jmix.io/schema/flowui/view"
      title="msg://customerDetailView.title">
    <data>
        <instance id="customerDc" class="com.company.project.entity.Customer">
            <fetchPlan extends="_base"/>
            <loader/>
        </instance>
    </data>
    <facets>
        <dataLoadCoordinator auto="true"/>
    </facets>
    <actions>
        <action id="saveAction" type="detail_saveClose"/>
        <action id="closeAction" type="detail_close"/>
    </actions>
    <layout>
        <formLayout id="form" dataContainer="customerDc">
            <textField id="nameField" property="name"/>
            <textField id="emailField" property="email"/>
        </formLayout>
        <hbox id="detailActions">
            <button id="saveAndCloseBtn" action="saveAction"/>
            <button id="closeBtn" action="closeAction"/>
        </hbox>
    </layout>
</view>
```

## Dependency Injection

### UI Components from XML — `@ViewComponent`
```java
@ViewComponent
private DataGrid<Customer> customersDataGrid;
@ViewComponent
private CollectionLoader<Customer> customersDl;
@ViewComponent
private MessageBundle messageBundle;
@ViewComponent
private DataContext dataContext;  // Exception: NOT @Autowired!
```

### Services and Infrastructure — `@Autowired`
```java
@Autowired
private DialogWindows dialogWindows;
@Autowired
private DataManager dataManager;
@Autowired
private Notifications notifications;
@Autowired
private Messages messages;
```

### Quick Reference
| What | Annotation | Example |
|------|------------|---------|
| Spring beans | `@Autowired` | `@Autowired DataManager dataManager` |
| UI components | `@ViewComponent` | `@ViewComponent DataGrid<Customer> grid` |
| Data containers | `@ViewComponent` | `@ViewComponent CollectionContainer<Customer> dc` |
| Data loaders | `@ViewComponent` | `@ViewComponent CollectionLoader<Customer> dl` |
| Message bundle | `@ViewComponent` | `@ViewComponent MessageBundle messageBundle` |
| **DataContext** | `@ViewComponent` | `@ViewComponent DataContext dataContext` |

## Event Handlers (@Subscribe)

```java
// InitEntityEvent — set defaults for NEW entities
@Subscribe
public void onInitEntity(InitEntityEvent<Customer> event) {
    event.getEntity().setStatus(CustomerStatus.NEW);
}

// BeforeCloseEvent — prevent closing with unsaved changes
@Subscribe
public void onBeforeClose(BeforeCloseEvent event) {
    if (dataContext.hasChanges()) {
        event.preventClose();
    }
}

// ValidationEvent — cross-field validation
@Subscribe
public void onValidation(ValidationEvent event) {
    Customer c = getEditedEntity();
    if (c.getEndDate().isBefore(c.getStartDate())) {
        event.getErrors().add("End date must be after start date");
    }
}
```

## Renderers (@Supply)

```java
@Supply(to = "customersDataGrid.status", subject = "renderer")
private Renderer<Customer> statusRenderer() {
    return new ComponentRenderer<>(customer -> {
        Span badge = new Span(customer.getStatus().name());
        badge.getElement().getThemeList().add("badge");
        return badge;
    });
}
```

## Opening Views

### ViewNavigators (URL changes)
```java
@Autowired
private ViewNavigators viewNavigators;

viewNavigators.detailView(this, Customer.class)
    .editEntity(customer)
    .navigate();
```

### DialogWindows (Modal, no URL)
```java
@Autowired
private DialogWindows dialogWindows;

dialogWindows.detail(this, Customer.class)
    .editEntity(customer)
    .withAfterCloseListener(e -> {
        if (e.closedWith(StandardOutcome.SAVE)) {
            customersDl.load();
        }
    })
    .open();
```

## Fragments

```java
@FragmentDescriptor("address-fragment.xml")  // No @ViewController!
public class AddressFragment extends Fragment<FormLayout> {
    
    @Subscribe(target = Target.HOST_CONTROLLER)
    public void onHostBeforeShow(View.BeforeShowEvent event) {
        // Fragments have no BeforeShow — use host's event
    }
}
```

## Checklist
- [ ] Java controller extends `StandardListView` or `StandardDetailView`
- [ ] XML descriptor in `resources/.../view/`
- [ ] `@Route`, `@ViewController`, `@ViewDescriptor` annotations
- [ ] Menu entry in `menu.xml`
- [ ] Messages for title/labels
- [ ] Run application and test in browser

## View Verification

Views have both design-time and runtime errors. Verify by:

1. **Run application** — test in browser, click through functionality
2. **IDEA MCP (recommended)** — catches runtime errors without running app

### IDEA MCP — Views Check
For views, IDEA MCP catches **real runtime errors** that Jmix Studio detects:
- Wrong component names (e.g., `tagPicker` doesn't exist)
- Missing properties, wrong attributes
- @Subscribe/@Install annotation errors

```
open_file_in_editor("path/to/view.xml")
wait 3-10 seconds
get_file_problems("path/to/view.xml", onlyErrors=false)
```

This saves time — you see errors **before** running the app.

## Forbidden
- Business logic in view controllers (move to services)
- EntityManager usage (use DataManager)
- Direct database transactions
