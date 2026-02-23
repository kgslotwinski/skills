# EasyAdmin CRUD Configuration API Reference

## Required Configuration

### Entity FQCN
```php
public static function getEntityFqcn(): string
{
    return Entity::class;
}
```

## configureCrud() Methods

### Design Options
```php
$crud->renderContentMaximized()        // Full browser width
$crud->renderSidebarMinimized()        // Narrow sidebar
```

### Entity Options
```php
$crud->setEntityLabelInSingular(string|callable $label)
$crud->setEntityLabelInPlural(string|callable $label)
$crud->setEntityPermission(string $permission)
```

**Callable signatures:**
- `fn(?Entity $entity, ?string $pageName): string`

### Page Titles and Help
```php
$crud->setPageTitle(string $pageName, string|callable $title)
$crud->setHelp(string $pageName, string $help)
```

**Available placeholders in titles:**
- `%entity_name%` - Entity class name
- `%entity_as_string%` - Entity __toString()
- `%entity_id%` - Entity ID
- `%entity_short_id%` - Shortened entity ID
- `%entity_label_singular%` - Singular label
- `%entity_label_plural%` - Plural label

**Callable signature for title:**
- `fn(Entity $entity): string` (for DETAIL and EDIT pages)

### Date, Time, Number Formats
```php
$crud->setDateFormat(string $format)
$crud->setTimeFormat(string $format)
$crud->setDateTimeFormat(string $dateFormat, ?string $timeFormat = null)
$crud->setDateIntervalFormat(string $format)
$crud->setTimezone(string $timezone)
```

**Format options:**
- Predefined: `'short'`, `'medium'`, `'long'`, `'full'`, `'none'`
- Constants: `DateTimeField::FORMAT_SHORT`, etc.
- ICU Datetime Pattern: `'yyyy.MM.dd HH:mm:ss'`

```php
$crud->setNumberFormat(string $format)
$crud->setThousandsSeparator(string $separator)
$crud->setDecimalSeparator(string $separator)
```

### Search Configuration
```php
$crud->setSearchFields(array|null $fields)
$crud->setAutofocusSearch()
$crud->setSearchMode(string $mode)
```

**Search modes:**
- `SearchMode::ALL_TERMS` - Match all terms (AND logic)
- `SearchMode::ANY_TERMS` - Match any term (OR logic)

**Search field examples:**
- Simple: `['name', 'description']`
- Associations: `['seller.email', 'seller.address.zipCode']`
- Disable: `null`

### Sorting and Pagination
```php
$crud->setDefaultSort(array $sort)
$crud->setPaginatorPageSize(int $size)
$crud->setPaginatorRangeSize(int $range)
$crud->setPaginatorUseOutputWalkers(bool $use)
$crud->setPaginatorFetchJoinCollection(bool $fetch)
```

**Sort examples:**
```php
['id' => 'DESC']
['id' => 'DESC', 'title' => 'ASC']
['seller.name' => 'ASC']  // One level deep
```

### Autocomplete Configuration
```php
$crud->autocomplete(
    bool $enable = true,
    ?callable $callback = null,
    ?string $template = null,
    bool $renderAsHtml = false
)
```

**Callback signature:**
- `fn($entity): string`

**Template variables:**
- `entity` - The entity instance

### Template and Form Options
```php
$crud->overrideTemplate(string $templateName, string $templatePath)
$crud->addFormTheme(string $theme)
$crud->setFormThemes(array $themes)
$crud->setFormOptions(array $options)
$crud->setFormOptions(array $newOptions, array $editOptions)
```

### Default Row Action
```php
$crud->setDefaultRowAction(string|array|null $action)
```

**Options:**
- Single action: `Action::EDIT`, `Action::DETAIL`, `'customAction'`
- Fallback array: `[Action::EDIT, Action::DETAIL]`
- `null` - Disable row click

### Batch Actions
```php
$crud->askConfirmationOnBatchActions(bool|string|TranslatableInterface $config)
```

**Placeholders:**
- `%action_name%` - Batch action name
- `%num_items%` - Number of items

### Misc Options
```php
$crud->hideNullValues()
$crud->showEntityActionsInlined()
```

## configureFields() Method

### Method Signature
```php
public function configureFields(string $pageName): iterable
```

**Return types:**
- Array of field objects
- Array of property name strings
- Generator (using `yield`)

### Page-specific Fields
Use `$pageName` parameter:
- `Crud::PAGE_INDEX`
- `Crud::PAGE_DETAIL`
- `Crud::PAGE_EDIT`
- `Crud::PAGE_NEW`

## CRUD Controller Lifecycle Methods

### Entity Creation and Persistence
```php
public function createEntity(string $entityFqcn)
public function updateEntity(EntityManagerInterface $em, $entity): void
public function persistEntity(EntityManagerInterface $em, $entity): void
public function deleteEntity(EntityManagerInterface $em, $entity): void
```

### Query Builder Customization
```php
public function createIndexQueryBuilder(
    SearchDto $searchDto,
    EntityDto $entityDto,
    FieldCollection $fields,
    FilterCollection $filters
): QueryBuilder
```

### Template Variables
```php
public function configureResponseParameters(KeyValueStore $responseParameters): KeyValueStore
```

**Key methods:**
- `$responseParameters->get(string $key)`
- `$responseParameters->set(string $key, $value)`
- `$responseParameters->has(string $key)`
- `$responseParameters->setIfNotSet(string $key, $value)`

**Supports dot notation:**
```php
$responseParameters->set('bar.foo', 'value')
// Equivalent to: $parameters['bar']['foo'] = 'value'
```

**Mandatory keys:**
- `templateName` or `templatePath` - Template to render

### Redirect After Save
```php
protected function getRedirectResponseAfterSave(
    AdminContext $context,
    string $action
): RedirectResponse
```

## CRUD Routes Configuration

### Dashboard-level (AdminDashboard attribute)
```php
#[AdminDashboard(routePath: '/admin', routeName: 'admin', routes: [
    'index' => ['routePath' => '/all', 'routeName' => 'list'],
    'new' => ['routePath' => '/create'],
    'edit' => ['routePath' => '/{entityId}/edit'],
    'delete' => ['routePath' => '/remove/{entityId}'],
    'detail' => ['routeName' => 'view'],
])]
```

### Controller-level (AdminRoute attribute)
```php
#[AdminRoute(path: '/custom-path', name: 'custom_name')]
class CustomCrudController extends AbstractCrudController
```

### Action-level (AdminRoute attribute)
```php
#[AdminRoute(path: '/custom-action', name: 'custom')]
public function customAction(AdminContext $context) { }
```

## Default CRUD Routes

| Action | Default Path | Default Name Suffix |
|--------|-------------|-------------------|
| index | `/` | `_index` |
| new | `/new` | `_new` |
| edit | `/{entityId}/edit` | `_edit` |
| detail | `/{entityId}` | `_detail` |
| delete | `/{entityId}/delete` | `_delete` |
| batch_delete | `/batch-delete` | `_batch_delete` |
| autocomplete | `/autocomplete` | `_autocomplete` |

## AdminContext Access

### In Controllers
```php
public function someAction(AdminContext $context)
{
    $entity = $context->getEntity()->getInstance();
    $crud = $context->getCrud();
    $request = $context->getRequest();
}
```

### Key Methods
```php
$context->getEntity()           // EntityDto
$context->getCrud()             // CrudDto
$context->getRequest()          // Request
$context->getI18n()            // I18nDto
$context->getAssets()          // AssetsDto
$context->getDashboardRouteName()
$context->getDashboardControllerFqcn()
```

## URL Generation

### Pretty URLs (Recommended)
```php
// Controller
return $this->redirectToRoute('admin_product_edit', ['entityId' => $id]);

// Template
{{ path('admin_product_detail', {entityId: product.id}) }}
```

### AdminUrlGenerator
```php
$url = $adminUrlGenerator
    ->setController(ProductCrudController::class)
    ->setAction(Action::INDEX)
    ->setEntityId($id)
    ->set('customParam', 'value')
    ->unset('menuIndex')
    ->unsetAll()
    ->generateUrl();
```

**Template:**
```twig
{% set url = ea_url()
    .setController('App\\Controller\\Admin\\ProductCrudController')
    .setAction('edit')
    .setEntityId(product.id) %}
```

### From Outside EasyAdmin
Must specify dashboard and controller:
```php
$url = $adminUrlGenerator
    ->setDashboard(DashboardController::class)
    ->setController(ProductCrudController::class)
    ->setAction(Action::INDEX)
    ->generateUrl();
```

## Dashboard Global Configuration

Define `configureCrud()` in dashboard controller to apply settings to all CRUD controllers:

```php
class DashboardController extends AbstractDashboardController
{
    public function configureCrud(): Crud
    {
        return Crud::new()
            ->setPaginatorPageSize(30)
            ->setDateFormat('yyyy-MM-dd')
            // ... other global settings
        ;
    }
}
```

Individual CRUD controllers can override these settings.

## Events

EasyAdmin triggers events during CRUD operations:

### Entity Events
- `BeforeEntityPersistedEvent` - Before persisting new entity
- `AfterEntityPersistedEvent` - After persisting new entity
- `BeforeEntityUpdatedEvent` - Before updating entity
- `AfterEntityUpdatedEvent` - After updating entity
- `BeforeEntityDeletedEvent` - Before deleting entity
- `AfterEntityDeletedEvent` - After deleting entity

### Action Events
- `BeforeCrudActionEvent` - Before any CRUD action
- `AfterCrudActionEvent` - After any CRUD action

### Usage
```php
class EntitySubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents()
    {
        return [
            BeforeEntityPersistedEvent::class => 'onBeforePersist',
        ];
    }

    public function onBeforePersist(BeforeEntityPersistedEvent $event)
    {
        $entity = $event->getEntityInstance();
        // Modify entity
    }
}
```

## Field Configurators

Custom field processing during runtime:

```php
class CustomFieldConfigurator implements FieldConfiguratorInterface
{
    public function supports(FieldDto $field, EntityDto $entityDto): bool
    {
        return TextField::class === $field->getFieldFqcn();
    }

    public function configure(FieldDto $field, EntityDto $entityDto, AdminContext $context): void
    {
        // Modify field configuration
    }
}
```

Tag service with `ea.field_configurator` (optional priority attribute).

## Template Names

Common template names for `overrideTemplate()`:

- `crud/index` - Index page
- `crud/detail` - Detail page
- `crud/edit` - Edit/new page
- `crud/field/id` - ID field rendering
- `crud/field/text` - Text field rendering
- `crud/field/association` - Association field rendering
- `layout` - Main layout
- `menu` - Menu sidebar
- `flash_messages` - Flash messages
- `label/null` - Null value label

Use `ea.templatePath('templateName')` in Twig to get full path.
