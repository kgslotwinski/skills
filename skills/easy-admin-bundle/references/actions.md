# EasyAdmin Actions API Reference

Complete reference for actions, batch operations, and action configuration.

## Table of Contents

- [Built-in Actions](#built-in-actions)
- [Actions Configuration](#actions-configuration)
- [Action Creation](#action-creation)
- [Batch Actions](#batch-actions)
- [Action Groups](#action-groups)

---

## Built-in Actions

### Available by Page

**INDEX page (Crud::PAGE_INDEX):**
- `Action::NEW` - Create new entity (global)
- `Action::EDIT` - Edit entity (per-row, default)
- `Action::DELETE` - Delete entity (per-row, default)
- `Action::DETAIL` - View details (per-row, optional)

**DETAIL page (Crud::PAGE_DETAIL):**
- `Action::EDIT` - Edit entity (default)
- `Action::DELETE` - Delete entity (default)
- `Action::INDEX` - Back to list (default)

**EDIT page (Crud::PAGE_EDIT):**
- `Action::SAVE_AND_RETURN` - Save and return (default)
- `Action::SAVE_AND_CONTINUE` - Save and stay (default)
- `Action::DELETE` - Delete entity (optional)
- `Action::DETAIL` - View details (optional)
- `Action::INDEX` - Back to list (optional)

**NEW page (Crud::PAGE_NEW):**
- `Action::SAVE_AND_RETURN` - Save and return (default)
- `Action::SAVE_AND_ADD_ANOTHER` - Save and create new (default)
- `Action::SAVE_AND_CONTINUE` - Save and edit (optional)
- `Action::INDEX` - Back to list (optional)

---

## Actions Configuration

### Main Methods

```php
use EasyCorp\Bundle\EasyAdminBundle\Config\{Action, Actions, Crud};

public function configureActions(Actions $actions): Actions
{
    // Add action to page
    $actions->add(string $pageName, Action $action)

    // Remove action from page
    $actions->remove(string $pageName, string $actionName)

    // Disable action globally (all pages)
    $actions->disable(string ...$actionNames)

    // Update existing action
    $actions->update(string $pageName, string $actionName, callable $callback)

    // Set permission
    $actions->setPermission(string $actionName, string $permission)

    // Reorder actions
    $actions->reorder(string $pageName, array $actionNames)

    // Disable automatic ordering
    $actions->disableAutomaticOrdering()

    // Add batch action
    $actions->addBatchAction(Action $action)

    return $actions;
}
```

### Examples

```php
public function configureActions(Actions $actions): Actions
{
    return $actions
        // Add DETAIL to index page
        ->add(Crud::PAGE_INDEX, Action::DETAIL)

        // Remove DELETE from index
        ->remove(Crud::PAGE_INDEX, Action::DELETE)

        // Disable DELETE everywhere
        ->disable(Action::DELETE)

        // Set permission
        ->setPermission(Action::DELETE, 'ROLE_SUPER_ADMIN')

        // Update existing action
        ->update(Crud::PAGE_INDEX, Action::EDIT, function (Action $action) {
            return $action
                ->setIcon('fa fa-pencil')
                ->displayIf(fn($entity) => $this->canEdit($entity));
        })

        // Reorder actions
        ->reorder(Crud::PAGE_INDEX, [
            Action::DETAIL,
            Action::EDIT,
            Action::DELETE,
        ]);
}
```

---

## Action Creation

### Constructor

```php
Action::new(string $name, ?string $label = null, ?string $icon = null)
```

### Link Target

```php
// Link to CRUD action (method in same controller)
->linkToCrudAction('methodName')

// Link to Symfony route
->linkToRoute('route_name', ['param' => 'value'])
->linkToRoute('route_name', fn($entity) => ['id' => $entity->getId()])

// Link to URL
->linkToUrl('https://example.com')
->linkToUrl(fn($entity) => sprintf('/custom/%d', $entity->getId()))
```

### Rendering

```php
->renderAsLink()                    // Link (default for per-entity)
->renderAsButton('submit')          // Button
->renderAsForm()                    // Hidden form with POST
```

### Styling

```php
->asDefaultAction()                 // btn-secondary
->asPrimaryAction()                 // btn-primary
->asSuccessAction()                 // btn-success
->asWarningAction()                 // btn-warning
->asDangerAction()                  // btn-danger
->asTextLink()                      // Plain text link

->setCssClass('btn btn-info')       // Replace classes
->addCssClass('custom-class')       // Add class
->setHtmlAttributes([...])
```

### Icon and Label

```php
->setIcon('fa fa-download')
->setLabel('Export')
->setLabel(fn($entity) => "Export {$entity->getName()}")
```

### Conditional Display

```php
->displayIf(fn($entity) => $this->isGranted('ROLE_ADMIN'))
->displayIf(fn($entity) => $entity->getStatus() === 'draft')
```

For global actions, callback receives `null`:
```php
->displayIf(fn($entity) => $entity === null || $this->isGranted('ROLE_ADMIN'))
```

### Confirmation Dialog

```php
->askConfirmation()                                          // Default message
->askConfirmation('Are you sure?')                          // Custom message
->askConfirmation('Delete this item?', 'Confirm Deletion')  // Message + button
->askConfirmation(false)                                     // Disable confirmation
```

**Available placeholders:**
- `%action_name%` - Action label
- `%entity_name%` - Entity label (singular)
- `%entity_id%` - Entity ID

### Global Action

```php
->createAsGlobalAction()            // Action for entire page (not per-entity)
```

---

## Batch Actions

### Configuration

```php
use EasyCorp\Bundle\EasyAdminBundle\Dto\BatchActionDto;

public function configureActions(Actions $actions): Actions
{
    $batchPublish = Action::new('batchPublish', 'Publish')
        ->linkToCrudAction('batchPublish')
        ->addCssClass('btn btn-success')
        ->setIcon('fa fa-check');

    return $actions->addBatchAction($batchPublish);
}

// In configureCrud() - optional confirmation
public function configureCrud(Crud $crud): Crud
{
    return $crud->askConfirmationOnBatchActions('Perform action on %num_items% items?');
}
```

### Batch Action Handler

```php
public function batchPublish(BatchActionDto $batchActionDto): Response
{
    $entityManager = $this->container->get('doctrine')->getManagerForClass($batchActionDto->getEntityFqcn());
    $repository = $entityManager->getRepository($batchActionDto->getEntityFqcn());

    foreach ($batchActionDto->getEntityIds() as $id) {
        $entity = $repository->find($id);
        if ($entity && method_exists($entity, 'setStatus')) {
            $entity->setStatus('published');
            $entity->setPublishedAt(new \DateTime());
        }
    }

    $entityManager->flush();

    $this->addFlash('success', sprintf('Published %d items', count($batchActionDto->getEntityIds())));

    return $this->redirect($batchActionDto->getReferrerUrl());
}
```

### BatchActionDto Methods

```php
$dto->getEntityFqcn()               // Entity class name
$dto->getEntityIds()                // Array of selected IDs
$dto->getReferrerUrl()              // URL to redirect back to
```

---

## Action Groups

Group related actions in a dropdown.

### Creating Action Groups

```php
use EasyCorp\Bundle\EasyAdminBundle\Config\ActionGroup;

$exportGroup = ActionGroup::new('Export', 'fa fa-download')
    ->addAction(Action::new('exportCsv')->linkToCrudAction('exportCsv'))
    ->addAction(Action::new('exportPdf')->linkToCrudAction('exportPdf'))
    ->addAction(Action::new('exportXml')->linkToCrudAction('exportXml'));

return $actions->add(Crud::PAGE_INDEX, $exportGroup);
```

### Split Button (Main Action + Dropdown)

```php
$exportGroup = ActionGroup::new('Export')
    ->addMainAction(Action::new('exportCsv')->linkToCrudAction('exportCsv'))
    ->addAction(Action::new('exportPdf')->linkToCrudAction('exportPdf'))
    ->addAction(Action::new('exportXml')->linkToCrudAction('exportXml'));
```

### Group Styling

```php
->asPrimaryActionGroup()
->asDefaultActionGroup()
->asSuccessActionGroup()
->asWarningActionGroup()
->asDangerActionGroup()
->setLabel('Actions')
->setIcon('fa fa-cog')
->addCssClass('custom-class')
->setHtmlAttributes([...])
```

### Group Organization

```php
$group = ActionGroup::new('Actions')
    ->addAction(Action::new('edit'))
    ->addDivider()                          // Add separator
    ->addHeader('Export Options')           // Add header
    ->addAction(Action::new('exportCsv'))
    ->addAction(Action::new('exportPdf'));
```

### Conditional Display

```php
->displayIf(fn($entity) => $this->isGranted('ROLE_ADMIN'))
```

---

## Complete Examples

### Custom Single Action

```php
public function configureActions(Actions $actions): Actions
{
    $export = Action::new('export', 'Export')
        ->linkToCrudAction('exportToCsv')
        ->setCssClass('btn btn-info')
        ->setIcon('fa fa-download')
        ->displayIf(fn($entity) => $entity->isExportable())
        ->askConfirmation('Export this entity?');

    return $actions->add(Crud::PAGE_DETAIL, $export);
}

public function exportToCsv(AdminContext $context): Response
{
    $entity = $context->getEntity()->getInstance();
    // Generate CSV...
    return new Response($csv, 200, [
        'Content-Type' => 'text/csv',
        'Content-Disposition' => 'attachment; filename="export.csv"',
    ]);
}
```

### Custom Global Action

```php
$exportAll = Action::new('exportAll', 'Export All')
    ->linkToCrudAction('exportAllToCsv')
    ->createAsGlobalAction()
    ->setCssClass('btn btn-success')
    ->setIcon('fa fa-file-export')
    ->askConfirmation('Export all items?', 'Export');

return $actions->add(Crud::PAGE_INDEX, $exportAll);
```

### Complex Permission Logic

```php
return $actions
    ->setPermission(Action::DELETE, 'ROLE_SUPER_ADMIN')
    ->update(Crud::PAGE_INDEX, Action::EDIT, fn(Action $action) =>
        $action->displayIf(fn($entity) =>
            $this->isGranted('ROLE_EDITOR') ||
            $entity->getAuthor() === $this->getUser()
        )
    )
    ->update(Crud::PAGE_INDEX, Action::DELETE, fn(Action $action) =>
        $action->displayIf(fn($entity) =>
            $this->isGranted('ROLE_SUPER_ADMIN') &&
            $entity->getId() !== $this->getUser()->getId()
        )
    );
```

---

## Row Action Configuration

### Default Row Action

In `configureCrud()`:
```php
$crud->setDefaultRowAction(Action::EDIT)
$crud->setDefaultRowAction([Action::DETAIL, Action::EDIT])  // Fallback chain
$crud->setDefaultRowAction(null)                             // Disable row click
```

### Custom Row Click Trigger

```php
$crud->setDefaultRowAction(Action::DETAIL)
    ->setRowClickTrigger('td:first-child')      // Only first cell
    ->setRowClickTrigger('td.entity-name')      // Specific class
    ->setRowClickTrigger(null)                  // Disable click entirely
```

### Display Actions Inline

In `configureCrud()`:
```php
$crud->showEntityActionsInlined()               // Show actions inline instead of dropdown
```

---

## URL Generation

### From Controller

```php
// Using route names
return $this->redirectToRoute('admin_product_edit', ['entityId' => $id]);

// Using AdminUrlGenerator
use EasyCorp\Bundle\EasyAdminBundle\Router\AdminUrlGenerator;

public function __construct(private AdminUrlGenerator $adminUrlGenerator) {}

public function someAction(): Response
{
    $url = $this->adminUrlGenerator
        ->setController(ProductCrudController::class)
        ->setAction(Action::EDIT)
        ->setEntityId($id)
        ->set('customParam', 'value')
        ->generateUrl();

    return $this->redirect($url);
}
```

### From Template

```twig
{# Route-based #}
{{ path('admin_product_edit', {entityId: product.id}) }}

{# AdminUrlGenerator #}
{% set url = ea_url()
    .setController('App\\Controller\\Admin\\ProductCrudController')
    .setAction('edit')
    .setEntityId(product.id) %}
```

---

## Quick Tips

1. **Permissions:** Use `setPermission()` for role-based access, `displayIf()` for complex logic
2. **Performance:** Conditional `displayIf()` runs for every entity - keep it fast
3. **Icons:** Use FontAwesome classes (e.g., `fa fa-download`)
4. **Batch actions:** Keep handlers fast, process in chunks for large datasets
5. **Confirmations:** Always confirm destructive batch actions
6. **URLs:** Prefer route names over AdminUrlGenerator when possible
7. **Global actions:** Use `createAsGlobalAction()` for page-level operations
