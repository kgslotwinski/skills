# EasyAdmin CRUD Configuration API Reference

Complete reference for CRUD controller configuration, lifecycle methods, and query customization.

## Table of Contents

- [Required Configuration](#required-configuration)
- [configureCrud() Methods](#configurecrud-methods)
- [configureFields()](#configurefields)
- [Lifecycle Methods](#lifecycle-methods)
- [Query Builders](#query-builders)
- [Events](#events)
- [URL Generation](#url-generation)

---

## Required Configuration

### Entity FQCN

```php
public static function getEntityFqcn(): string
{
    return Product::class;
}
```

---

## configureCrud() Methods

### Entity Labels

```php
public function configureCrud(Crud $crud): Crud
{
    return $crud
        ->setEntityLabelInSingular('Product')
        ->setEntityLabelInSingular(fn(?Product $p) => $p ? $p->getName() : 'Product')
        ->setEntityLabelInPlural('Products');
}
```

### Page Titles

```php
$crud
    ->setPageTitle('index', 'Product Catalog')
    ->setPageTitle('edit', 'Edit %entity_label_singular%')
    ->setPageTitle('detail', fn(Product $p) => $p->getName());
```

**Available placeholders:**
- `%entity_name%` - Entity class name
- `%entity_as_string%` - Entity `__toString()`
- `%entity_id%` - Entity ID
- `%entity_short_id%` - Shortened ID
- `%entity_label_singular%` - Singular label
- `%entity_label_plural%` - Plural label

### Page Help Text

```php
$crud->setHelp('index', 'Manage your product catalog')
    ->setHelp('edit', 'Update product information');
```

### Search Configuration

```php
$crud
    ->setSearchFields(['name', 'sku', 'description'])
    ->setSearchFields(['seller.email', 'seller.address.zipCode'])  // Nested properties
    ->setSearchFields(null)                                         // Disable search
    ->setAutofocusSearch()
    ->setSearchMode(SearchMode::ALL_TERMS)                         // AND logic (default)
    ->setSearchMode(SearchMode::ANY_TERMS);                        // OR logic
```

### Sorting and Pagination

```php
$crud
    ->setDefaultSort(['createdAt' => 'DESC'])
    ->setDefaultSort(['id' => 'DESC', 'name' => 'ASC'])
    ->setDefaultSort(['seller.name' => 'ASC'])                     // One level deep only
    ->setPaginatorPageSize(30)
    ->setPaginatorRangeSize(5)                                     // Number of page links
    ->setPaginatorUseOutputWalkers(false)                          // Performance optimization
    ->setPaginatorFetchJoinCollection(true);                       // Fetch joined collections
```

### Date/Time Formats

```php
$crud
    ->setDateFormat('yyyy-MM-dd')                                  // ICU pattern
    ->setDateFormat('short')                                       // Predefined: short, medium, long, full
    ->setTimeFormat('HH:mm')
    ->setDateTimeFormat('yyyy-MM-dd', 'HH:mm:ss')
    ->setDateIntervalFormat('%y years %m months')
    ->setTimezone('America/New_York');
```

### Number Formats

```php
$crud
    ->setNumberFormat('%.2f')                                      // sprintf format
    ->setThousandsSeparator(',')
    ->setDecimalSeparator('.');
```

### Display Options

```php
$crud
    ->renderContentMaximized()                                     // Full width
    ->renderSidebarMinimized()                                     // Narrow sidebar
    ->hideNullValues()                                             // Hide null values
    ->showEntityActionsInlined();                                  // Inline actions (not dropdown)
```

### Row Actions

```php
$crud
    ->setDefaultRowAction(Action::EDIT)                            // Single action
    ->setDefaultRowAction([Action::DETAIL, Action::EDIT])         // Fallback chain
    ->setDefaultRowAction(null)                                    // Disable row click
    ->setRowClickTrigger('td:first-child')                        // Custom trigger selector
    ->setRowClickTrigger(null);                                    // Disable entirely
```

### Autocomplete

```php
$crud->autocomplete(
    enable: true,
    callback: fn($entity) => sprintf('%s (#%d)', $entity->getName(), $entity->getId()),
    template: 'admin/autocomplete/product.html.twig',
    renderAsHtml: true
);
```

### Security

```php
$crud->setEntityPermission('ROLE_ADMIN');                          // Required for all operations
```

### Forms

```php
$crud
    ->setFormThemes(['@EasyAdmin/crud/form_theme.html.twig', 'admin/form_theme.html.twig'])
    ->addFormTheme('admin/custom_theme.html.twig')
    ->setFormOptions(['validation_groups' => ['Default', 'create']])
    ->setFormOptions(
        ['validation_groups' => ['Default', 'create']],            // New form options
        ['validation_groups' => ['Default', 'edit']]               // Edit form options
    );
```

### Templates

```php
$crud->overrideTemplate('crud/index', 'admin/custom_index.html.twig')
    ->overrideTemplate('crud/detail', 'admin/custom_detail.html.twig');
```

**Common template names:**
- `crud/index` - List page
- `crud/detail` - Detail page
- `crud/edit` - Edit/new page
- `crud/field/*` - Field rendering templates
- `layout` - Main layout
- `menu` - Sidebar menu

### Batch Actions

```php
$crud->askConfirmationOnBatchActions('Perform action on %num_items% items?');
$crud->askConfirmationOnBatchActions(true);                        // Default message
$crud->askConfirmationOnBatchActions(false);                       // Disable confirmation
```

---

## configureFields()

### Method Signature

```php
public function configureFields(string $pageName): iterable
{
    // $pageName: Crud::PAGE_INDEX, Crud::PAGE_DETAIL, Crud::PAGE_EDIT, Crud::PAGE_NEW

    // Return array or yield fields
    yield IdField::new('id')->onlyOnIndex();
    yield TextField::new('name');
}
```

### Page-Specific Configuration

```php
public function configureFields(string $pageName): iterable
{
    yield IdField::new('id')->hideOnForm();
    yield TextField::new('name');

    if ($pageName === Crud::PAGE_EDIT) {
        yield DateTimeField::new('updatedAt')->setFormTypeOptions(['disabled' => true]);
    }

    if ($pageName === Crud::PAGE_INDEX) {
        yield TextField::new('summary');
    } else {
        yield TextEditorField::new('content');
    }
}
```

---

## Lifecycle Methods

### Entity Creation

```php
public function createEntity(string $entityFqcn)
{
    $product = new Product();
    $product->setCreatedAt(new \DateTime());
    $product->setCreatedBy($this->getUser());
    return $product;
}
```

### Persistence

```php
public function persistEntity(EntityManagerInterface $em, $entityInstance): void
{
    if ($entityInstance instanceof Product) {
        $entityInstance->setSlug($this->slugger->slug($entityInstance->getName()));
    }

    parent::persistEntity($em, $entityInstance);
}

public function updateEntity(EntityManagerInterface $em, $entityInstance): void
{
    if ($entityInstance instanceof Product) {
        $entityInstance->setUpdatedAt(new \DateTime());
        $entityInstance->setUpdatedBy($this->getUser());
    }

    parent::updateEntity($em, $entityInstance);
}

public function deleteEntity(EntityManagerInterface $em, $entityInstance): void
{
    // Hard delete
    parent::deleteEntity($em, $entityInstance);

    // Or soft delete
    if ($entityInstance instanceof SoftDeletable) {
        $entityInstance->setDeletedAt(new \DateTime());
        $em->flush();
    } else {
        parent::deleteEntity($em, $entityInstance);
    }
}
```

---

## Query Builders

### Index Query Builder

```php
use Doctrine\ORM\QueryBuilder;
use EasyCorp\Bundle\EasyAdminBundle\Collection\{FieldCollection, FilterCollection};
use EasyCorp\Bundle\EasyAdminBundle\Dto\{EntityDto, SearchDto};

public function createIndexQueryBuilder(
    SearchDto $searchDto,
    EntityDto $entityDto,
    FieldCollection $fields,
    FilterCollection $filters
): QueryBuilder {
    $qb = parent::createIndexQueryBuilder($searchDto, $entityDto, $fields, $filters);

    // Only show active products
    $qb->andWhere('entity.isActive = :active')
       ->setParameter('active', true);

    // Multi-tenant filtering
    if (!$this->isGranted('ROLE_ADMIN')) {
        $qb->andWhere('entity.tenant = :tenant')
           ->setParameter('tenant', $this->getUser()->getTenant());
    }

    // Soft delete support
    $qb->andWhere('entity.deletedAt IS NULL');

    return $qb;
}
```

### Other Query Builders

```php
public function createEditQueryBuilder(/* ... */): QueryBuilder { }
public function createNewQueryBuilder(/* ... */): QueryBuilder { }
public function createDetailQueryBuilder(/* ... */): QueryBuilder { }
```

---

## Events

### Available Events

```php
use EasyCorp\Bundle\EasyAdminBundle\Event\*;

// Entity events
BeforeEntityPersistedEvent::class    // Before creating entity
AfterEntityPersistedEvent::class     // After creating entity
BeforeEntityUpdatedEvent::class      // Before updating entity
AfterEntityUpdatedEvent::class       // After updating entity
BeforeEntityDeletedEvent::class      // Before deleting entity
AfterEntityDeletedEvent::class       // After deleting entity

// CRUD events
BeforeCrudActionEvent::class         // Before any CRUD action
AfterCrudActionEvent::class          // After any CRUD action
```

### Event Subscriber Example

```php
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class ProductSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [
            BeforeEntityPersistedEvent::class => 'setDefaults',
            BeforeEntityUpdatedEvent::class => 'updateTimestamp',
        ];
    }

    public function setDefaults(BeforeEntityPersistedEvent $event): void
    {
        $entity = $event->getEntityInstance();

        if (!$entity instanceof Product) {
            return;
        }

        $entity->setCreatedAt(new \DateTime());
        $entity->setSku($this->generateSku());
    }

    public function updateTimestamp(BeforeEntityUpdatedEvent $event): void
    {
        $entity = $event->getEntityInstance();

        if (!$entity instanceof Product) {
            return;
        }

        $entity->setUpdatedAt(new \DateTime());
    }
}
```

---

## Response Customization

### Template Variables

```php
use EasyCorp\Bundle\EasyAdminBundle\Config\KeyValueStore;

public function configureResponseParameters(KeyValueStore $responseParameters): KeyValueStore
{
    if (Crud::PAGE_DETAIL === $responseParameters->get('pageName')) {
        $responseParameters->set('custom_var', 'value');
        $responseParameters->set('product_stats', $this->getProductStats());
    }

    return $responseParameters;
}
```

**Methods:**
- `$responseParameters->get(string $key)`
- `$responseParameters->set(string $key, $value)`
- `$responseParameters->has(string $key)`
- `$responseParameters->setIfNotSet(string $key, $value)`

**Supports dot notation:**
```php
$responseParameters->set('stats.total', 100);  // $parameters['stats']['total'] = 100
```

### Redirect After Save

```php
protected function getRedirectResponseAfterSave(AdminContext $context, string $action): RedirectResponse
{
    $submitButtonName = $context->getRequest()->request->all()['ea']['newForm']['btn'] ?? '';

    if ('saveAndViewDetail' === $submitButtonName) {
        $url = $this->adminUrlGenerator
            ->setAction(Action::DETAIL)
            ->setEntityId($context->getEntity()->getPrimaryKeyValue())
            ->generateUrl();

        return $this->redirect($url);
    }

    return parent::getRedirectResponseAfterSave($context, $action);
}
```

---

## URL Generation

### AdminContext

```php
use EasyCorp\Bundle\EasyAdminBundle\Context\AdminContext;

public function customAction(AdminContext $context): Response
{
    $entity = $context->getEntity()->getInstance();
    $entityId = $context->getEntity()->getPrimaryKeyValue();
    $crud = $context->getCrud();
    $request = $context->getRequest();
    $dashboardFqcn = $context->getDashboardControllerFqcn();
    $dashboardRoute = $context->getDashboardRouteName();
    $referrer = $context->getReferrer();

    // ...
}
```

### AdminUrlGenerator

```php
use EasyCorp\Bundle\EasyAdminBundle\Router\AdminUrlGenerator;

public function __construct(private AdminUrlGenerator $adminUrlGenerator) {}

public function someAction(): Response
{
    $url = $this->adminUrlGenerator
        ->setController(ProductCrudController::class)
        ->setAction(Action::EDIT)
        ->setEntityId($id)
        ->set('customParam', 'value')
        ->unset('menuIndex')
        ->unsetAll()
        ->generateUrl();

    return $this->redirect($url);
}
```

### From Outside EasyAdmin Context

```php
$url = $this->adminUrlGenerator
    ->setDashboard(DashboardController::class)        // Required when outside context
    ->setController(ProductCrudController::class)
    ->setAction(Action::INDEX)
    ->generateUrl();
```

### Route-Based (Recommended)

```php
// Controller
return $this->redirectToRoute('admin_product_edit', ['entityId' => $id]);

// Template
{{ path('admin_product_detail', {entityId: product.id}) }}
```

---

## Dashboard Global Configuration

Apply settings to all CRUD controllers:

```php
// src/Controller/Admin/DashboardController.php
class DashboardController extends AbstractDashboardController
{
    public function configureCrud(): Crud
    {
        return Crud::new()
            ->setPaginatorPageSize(30)
            ->setDateFormat('dd/MM/yyyy')
            ->setTimezone('UTC')
            ->renderContentMaximized();
    }
}
```

Individual CRUD controllers can override these settings.

---

## Custom Routes

### Dashboard Level

```php
use EasyCorp\Bundle\EasyAdminBundle\Config\Dashboard;

#[AdminDashboard(
    routePath: '/admin',
    routeName: 'admin',
    routes: [
        'index' => ['routePath' => '/products', 'routeName' => 'list'],
        'new' => ['routePath' => '/product/new'],
        'edit' => ['routePath' => '/product/{entityId}/edit'],
    ]
)]
class DashboardController extends AbstractDashboardController
```

### Controller Level

```php
#[AdminRoute(path: '/products', name: 'products')]
class ProductCrudController extends AbstractCrudController
```

### Action Level

```php
#[AdminRoute(path: '/export', name: 'export')]
public function exportAction(AdminContext $context): Response
```

**Final route:** `/admin` + controller path + action path
**Final name:** `admin_` + controller name + `_` + action name

---

## Field Configurators

Custom field processing at runtime:

```php
use EasyCorp\Bundle\EasyAdminBundle\Contracts\Field\FieldConfiguratorInterface;
use EasyCorp\Bundle\EasyAdminBundle\Dto\{FieldDto, EntityDto};
use EasyCorp\Bundle\EasyAdminBundle\Context\AdminContext;

class CustomFieldConfigurator implements FieldConfiguratorInterface
{
    public function supports(FieldDto $field, EntityDto $entityDto): bool
    {
        return TextField::class === $field->getFieldFqcn();
    }

    public function configure(FieldDto $field, EntityDto $entityDto, AdminContext $context): void
    {
        // Modify field configuration dynamically
        $field->setFormattedValue(strtoupper($field->getValue()));
    }
}
```

Tag service with `ea.field_configurator` (with optional priority).

---

## Quick Tips

1. **Performance:** Override `createIndexQueryBuilder()` to add eager loading with `->leftJoin()` for associations
2. **Security:** Use `setEntityPermission()` for blanket permissions, `displayIf()` in actions for granular control
3. **Multi-tenant:** Filter in `createIndexQueryBuilder()` and set tenant in `persistEntity()`
4. **Soft delete:** Filter `deletedAt IS NULL` in query builder, implement soft delete in `deleteEntity()`
5. **Audit:** Use events (BeforeEntityUpdatedEvent) or override lifecycle methods
6. **Custom redirects:** Override `getRedirectResponseAfterSave()` for custom post-save behavior
7. **Template vars:** Use `configureResponseParameters()` to pass custom variables to templates
8. **Forms:** Use `setFormOptions()` for validation groups, form themes, and other Symfony form options
