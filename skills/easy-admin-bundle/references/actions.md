# EasyAdmin Actions API Reference

## Built-in Actions

### Index Page (Crud::PAGE_INDEX)
**Default global actions:**
- `Action::NEW` - Create new entity

**Default per-entry actions:**
- `Action::EDIT` - Edit entity
- `Action::DELETE` - Delete entity

**Other available actions:**
- `Action::DETAIL` - View entity details

### Detail Page (Crud::PAGE_DETAIL)
**Default actions:**
- `Action::EDIT` - Edit entity
- `Action::DELETE` - Delete entity
- `Action::INDEX` - Back to listing

### Edit Page (Crud::PAGE_EDIT)
**Default actions:**
- `Action::SAVE_AND_RETURN` - Save and go back
- `Action::SAVE_AND_CONTINUE` - Save and stay on page

**Other available actions:**
- `Action::DELETE` - Delete entity
- `Action::DETAIL` - View details
- `Action::INDEX` - Back to listing

### New Page (Crud::PAGE_NEW)
**Default actions:**
- `Action::SAVE_AND_RETURN` - Save and go back
- `Action::SAVE_AND_ADD_ANOTHER` - Save and create another

**Other available actions:**
- `Action::SAVE_AND_CONTINUE` - Save and go to edit page
- `Action::INDEX` - Back to listing

## Actions Configuration

### Main Methods (in configureActions())

**Adding Actions:**
```php
$actions->add(string $pageName, Action $action)
```

**Removing Actions:**
```php
$actions->remove(string $pageName, string $actionName)
```

**Updating Actions:**
```php
$actions->update(string $pageName, string $actionName, callable $callback)
```
- Callback receives Action object, should return modified Action

**Disabling Actions:**
```php
$actions->disable(string ...$actionNames)
```
- Disables globally (all pages)

**Restricting Actions:**
```php
$actions->setPermission(string $actionName, string $permission)
```
- Requires Symfony security permission

**Reordering Actions:**
```php
$actions->reorder(string $pageName, array $actionNames)
```

**Disable Automatic Ordering:**
```php
$actions->disableAutomaticOrdering()
```

**Add Batch Action:**
```php
$actions->addBatchAction(Action $action)
```

## Action Creation

### Constructor
```php
Action::new(string $name, ?string $label = null, ?string $icon = null)
```

### Link Target Methods
```php
->linkToCrudAction(string $actionName)
->linkToRoute(string $routeName, array|callable $routeParameters = [])
->linkToUrl(string|callable $url)
```

### Rendering Methods
```php
->renderAsLink()
->renderAsButton(string $type = 'submit')
->renderAsForm()  // POST request via hidden form
```

### Styling Methods
```php
->asDefaultAction()     // btn-secondary
->asPrimaryAction()     // btn-primary
->asSuccessAction()     // btn-success
->asWarningAction()     // btn-warning
->asDangerAction()      // btn-danger
->asTextLink()          // Text link without button background
```

### CSS and HTML
```php
->setCssClass(string $class)
->addCssClass(string $class)
->setHtmlAttributes(array $attributes)
```

### Icon and Label
```php
->setIcon(string $iconClass)
->setLabel(string|callable $label)
```
- Callable receives entity instance

### Conditional Display
```php
->displayIf(callable $callback)
```
- Callback signature: `fn($entity): bool`
- For global actions, callback receives null

### Action Confirmation
```php
->askConfirmation()
->askConfirmation(string|TranslatableInterface $message)
->askConfirmation(string|TranslatableInterface $message, string|TranslatableInterface $buttonLabel)
->askConfirmation(false)  // Disable confirmation
```

Available placeholders in confirmation message:
- `%action_name%` - Action label
- `%entity_name%` - Entity label (singular)
- `%entity_id%` - Entity ID

### Global Actions
```php
->createAsGlobalAction()
```
- Creates action for entire page (not per entity)

## Action Groups

### Creating Action Groups
```php
ActionGroup::new(string $name, ?string $label = null, ?string $icon = null)
    ->addAction(Action $action)
    ->addMainAction(Action $action)  // For split button
```

### Action Group Configuration
```php
->createAsGlobalActionGroup()
->displayIf(callable $callback)
```

### Styling
```php
->asPrimaryActionGroup()
->asDefaultActionGroup()
->asSuccessActionGroup()
->asWarningActionGroup()
->asDangerActionGroup()
->setLabel(string|false $label)
->setIcon(string $icon)
->addCssClass(string $class)
->setHtmlAttributes(array $attributes)
```

### Organization
```php
->addHeader(string $header)
->addDivider()
```

## Batch Actions

### Configuration
```php
$actions->addBatchAction(Action::new('actionName')
    ->linkToCrudAction('methodName')
    ->addCssClass('btn btn-primary')
    ->setIcon('fa fa-icon'))
```

### Batch Action Handler
Method receives `BatchActionDto` with:
```php
$batchActionDto->getEntityFqcn()      // Entity class name
$batchActionDto->getEntityIds()       // Array of selected IDs
```

### Batch Confirmation
In `configureCrud()`:
```php
$crud->askConfirmationOnBatchActions(bool|string|TranslatableInterface $config)
```

Available placeholders:
- `%action_name%` - Batch action name
- `%num_items%` - Number of selected items

## Action Extensions

Create extension class implementing `ActionsExtensionInterface`:

```php
class CustomActionExtension implements ActionsExtensionInterface
{
    public function supports(AdminContext $context): bool
    {
        // Return true to enable extension
        return $context->getCrud()->getCurrentPage() === Crud::PAGE_DETAIL;
    }

    public function extend(Actions $actions, AdminContext $context): void
    {
        // Add, remove, or modify actions
        $actions->add(Crud::PAGE_DETAIL, Action::new('custom'));
    }
}
```

Tag service with `ea.action_extension`.

## Entity Action Display

### Dropdown vs Inline (Index Page)
In `configureCrud()`:
```php
$crud->showEntityActionsInlined()
```
- Default: Actions in dropdown
- After method call: Actions displayed inline

## Custom Symfony Controller Integration

### AdminRoute Attribute
```php
#[AdminRoute(path: '/custom', name: 'custom')]
class CustomController extends AbstractController
{
    #[AdminRoute(path: '/{id}', name: 'detail')]
    public function detail() { }
}
```

**Options:**
- `path` - URL path segment
- `name` - Route name segment
- `allowedDashboards` - Array of dashboard classes to include
- `deniedDashboards` - Array of dashboard classes to exclude

**Route Generation:**
- Final path: `/admin` + controller path + action path
- Final name: `admin_` + controller name + `_` + action name

## URL Generation

### Pretty URLs
```php
// In controller
return $this->redirectToRoute('admin_product_edit', ['entityId' => $id]);

// In template
{{ path('admin_product_detail', {entityId: product.id}) }}
```

### AdminUrlGenerator (Legacy/Dynamic)
```php
$url = $adminUrlGenerator
    ->setController(ProductCrudController::class)
    ->setAction(Action::EDIT)
    ->setEntityId($id)
    ->generateUrl();
```

### Template Helper
```twig
{% set url = ea_url()
    .setController('App\\Controller\\Admin\\ProductCrudController')
    .setAction('edit')
    .setEntityId(product.id) %}
```

## Default Row Action

Configure in `configureCrud()`:
```php
$crud->setDefaultRowAction(Action::EDIT)
$crud->setDefaultRowAction([Action::EDIT, Action::DETAIL])  // Fallback chain
$crud->setDefaultRowAction(null)  // Disable
```

Available options:
- Single action: `Action::EDIT`, `Action::DETAIL`, `'customAction'`
- Fallback array: `[Action::DETAIL, Action::EDIT]`
- `null` - Disable row click behavior
