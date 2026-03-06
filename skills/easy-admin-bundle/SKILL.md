---
name: easy-admin-bundle
version: 1.2.0
compatibility:
  easyadmin: "^4.0"
  php: ">=8.1"
  symfony: ">=5.4"
updated: 2026-03-06
description: "Rapid Symfony admin panel development with EasyAdminBundle. Capabilities: CRUD generation, dashboard creation, entity management, form customization, menu configuration, field configuration, action customization, filters, permissions, batch actions, custom themes, file uploads, image handling, associations, submenus, custom queries, event listeners. Use for: admin panels, backend interfaces, content management, data administration, CRUD operations, user management, product catalogs, blog administration, e-commerce backends. Triggers: easyadmin, symfony admin, crud, admin panel, dashboard, backend, entity crud, admin interface, symfony backend, easyadmin, easyadmin bundle, admin generator, backend panel"
allowed-tools: Bash(composer *, symfony *, php bin/console *), Read, Write, Edit, Glob, Grep
---

# EasyAdmin Bundle Skill

Build powerful Symfony admin panels quickly with EasyAdminBundle. This skill provides complete examples and patterns for production-ready admin interfaces.

> **Version:** EasyAdminBundle 4.x (current stable)
> **Requirements:** PHP 8.1+, Symfony 5.4/6.x/7.x/8.x, Doctrine ORM
> **Official Docs:** https://symfony.com/bundles/EasyAdminBundle/4.x/index.html
>
> **📚 Quick Reference:** See [API Reference](#api-reference) for complete field types, actions, and CRUD methods

---

## Table of Contents

- [Quick Start](#quick-start)
- [Dashboard Setup](#dashboard-setup)
- [CRUD Controllers](#crud-controllers)
- [Fields Overview](#fields-overview)
- [Filters](#filters)
- [Actions](#actions)
- [Advanced Features](#advanced-features)
- [Common Patterns](#common-patterns)
- [API Reference](#api-reference)
- [Troubleshooting](#troubleshooting)

---

## Quick Start

```bash
# Install
composer require easycorp/easyadmin-bundle

# Create dashboard
php bin/console make:admin:dashboard

# Create CRUD controllers
php bin/console make:admin:crud
```

Visit `/admin` in your browser.

---

## Dashboard Setup

### Basic Dashboard with Menu

```php
// src/Controller/Admin/DashboardController.php
namespace App\Controller\Admin;

use App\Entity\{User, Post, Category};
use EasyCorp\Bundle\EasyAdminBundle\Config\{Dashboard, MenuItem};
use EasyCorp\Bundle\EasyAdminBundle\Controller\AbstractDashboardController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class DashboardController extends AbstractDashboardController
{
    #[Route('/admin', name: 'admin')]
    public function index(): Response
    {
        return $this->render('admin/dashboard.html.twig');
    }

    public function configureDashboard(): Dashboard
    {
        return Dashboard::new()
            ->setTitle('My Admin Panel')
            ->setFaviconPath('favicon.ico')
            ->setTranslationDomain('admin')
            ->renderContentMaximized()
            ->disableDarkMode();
    }

    public function configureMenuItems(): iterable
    {
        yield MenuItem::linkToDashboard('Dashboard', 'fa fa-home');

        // Simple menu items
        yield MenuItem::section('Content');
        yield MenuItem::linkToCrud('Posts', 'fa fa-newspaper', Post::class);
        yield MenuItem::linkToCrud('Categories', 'fa fa-tags', Category::class);

        // Submenus
        yield MenuItem::subMenu('E-commerce', 'fa fa-shopping-cart')->setSubItems([
            MenuItem::linkToCrud('Products', 'fa fa-box', Product::class),
            MenuItem::linkToCrud('Orders', 'fa fa-shopping-bag', Order::class),
        ]);

        // Link to any controller (not just CRUD)
        yield MenuItem::linkTo('Reports', 'fa fa-chart', ReportsController::class);

        // Routes with query parameters
        yield MenuItem::linkToRoute('Analytics', 'fa fa-chart-line', 'app_analytics', [
            'year' => date('Y'),
        ]);

        // External links
        yield MenuItem::linkToUrl('Homepage', 'fa fa-globe', '/');
        yield MenuItem::linkToLogout('Logout', 'fa fa-sign-out');
    }
}
```

---

## CRUD Controllers

### Complete CRUD Example

```php
namespace App\Controller\Admin;

use App\Entity\Product;
use Doctrine\ORM\QueryBuilder;
use EasyCorp\Bundle\EasyAdminBundle\Config\{Action, Actions, Crud, Filters};
use EasyCorp\Bundle\EasyAdminBundle\Controller\AbstractCrudController;
use EasyCorp\Bundle\EasyAdminBundle\Field\{
    IdField, TextField, TextEditorField, MoneyField, IntegerField,
    SlugField, AssociationField, ImageField, BooleanField, DateTimeField, ChoiceField
};
use EasyCorp\Bundle\EasyAdminBundle\Filter\{BooleanFilter, DateTimeFilter};

class ProductCrudController extends AbstractCrudController
{
    public static function getEntityFqcn(): string
    {
        return Product::class;
    }

    public function configureCrud(Crud $crud): Crud
    {
        return $crud
            ->setEntityLabelInSingular('Product')
            ->setEntityLabelInPlural('Products')
            ->setPageTitle('index', 'Product Catalog')
            ->setSearchFields(['name', 'sku', 'description'])
            ->setDefaultSort(['createdAt' => 'DESC'])
            ->setPaginatorPageSize(30)
            ->setEntityPermission('ROLE_ADMIN')
            ->setDefaultRowAction(Action::DETAIL);
    }

    public function configureFields(string $pageName): iterable
    {
        yield IdField::new('id')->onlyOnIndex();

        yield TextField::new('name')->setColumns(8);
        yield SlugField::new('slug')->setTargetFieldName('name')->setColumns(4)->hideOnIndex();

        yield TextField::new('sku')->setColumns(6);
        yield MoneyField::new('price')->setCurrency('USD')->setColumns(6);
        yield IntegerField::new('stock')->setColumns(6);

        yield AssociationField::new('category')
            ->setColumns(6)
            ->autocomplete()
            ->setPreferredChoices(fn() => $this->getPopularCategories());

        yield AssociationField::new('tags')
            ->hideOnIndex()
            ->autocomplete()
            ->setFormTypeOptions(['by_reference' => false]);

        yield TextEditorField::new('description')->hideOnIndex();

        yield ImageField::new('image')
            ->setBasePath('uploads/products')
            ->setUploadDir('public/uploads/products')
            ->setUploadedFileNamePattern('[slug]-[timestamp].[extension]');

        yield ChoiceField::new('status')
            ->setChoices([
                'Draft' => 'draft',
                'Published' => 'published',
                'Archived' => 'archived',
            ])
            ->renderAsBadges([
                'draft' => 'warning',
                'published' => 'success',
                'archived' => 'secondary',
            ]);

        yield BooleanField::new('isFeatured')->renderAsSwitch(false);
        yield BooleanField::new('isActive')->renderAsSwitch(false);

        yield DateTimeField::new('createdAt')->hideOnForm();
        yield DateTimeField::new('updatedAt')->hideOnForm()->hideOnIndex();
    }

    public function configureFilters(Filters $filters): Filters
    {
        return $filters
            ->add('name')
            ->add('category')
            ->add(BooleanFilter::new('isActive'))
            ->add(BooleanFilter::new('isFeatured'))
            ->add(DateTimeFilter::new('createdAt'));
    }

    public function configureActions(Actions $actions): Actions
    {
        // Custom action with confirmation
        $publish = Action::new('publish', 'Publish')
            ->linkToCrudAction('publishProduct')
            ->setCssClass('btn btn-success')
            ->displayIf(fn(Product $p) => $p->getStatus() === 'draft')
            ->askConfirmation('Publish this product?', 'Confirm');

        return $actions
            ->add(Crud::PAGE_INDEX, Action::DETAIL)
            ->add(Crud::PAGE_DETAIL, $publish)
            ->setPermission(Action::DELETE, 'ROLE_SUPER_ADMIN');
    }

    // Custom query for index page
    public function createIndexQueryBuilder(/* ... */): QueryBuilder
    {
        $qb = parent::createIndexQueryBuilder(...func_get_args());

        // Only show active products for non-admins
        if (!$this->isGranted('ROLE_ADMIN')) {
            $qb->andWhere('entity.isActive = :active')
               ->setParameter('active', true);
        }

        return $qb;
    }

    // Custom action method
    public function publishProduct(AdminContext $context): Response
    {
        $product = $context->getEntity()->getInstance();
        $product->setStatus('published');
        $product->setPublishedAt(new \DateTime());

        $this->entityManager->flush();
        $this->addFlash('success', 'Product published successfully');

        return $this->redirect($context->getReferrer());
    }

    private function getPopularCategories(): array
    {
        return $this->entityManager->getRepository(Category::class)
            ->findBy(['featured' => true], ['name' => 'ASC']);
    }
}
```

---

## Fields Overview

EasyAdmin provides 30+ field types. For complete reference, see [📋 Fields API](references/fields.md).

### Common Fields Quick Reference

```php
use EasyCorp\Bundle\EasyAdminBundle\Field\*;

// Text
TextField::new('title')
TextareaField::new('description')
TextEditorField::new('content')  // WYSIWYG
CodeEditorField::new('customCss')->setLanguage('css')

// Numbers & Money
IntegerField::new('quantity')
NumberField::new('price')->setNumDecimals(2)
MoneyField::new('price')->setCurrency('USD')
PercentField::new('discount')

// Dates
DateField::new('publishedAt')->setFormat('dd/MM/yyyy')
DateTimeField::new('createdAt')
TimeField::new('openingTime')

// Boolean & Choices
BooleanField::new('isActive')->renderAsSwitch()
ChoiceField::new('status')->setChoices([...])->renderAsBadges([...])

// Files & Images
ImageField::new('thumbnail')
    ->setBasePath('uploads/images')
    ->setUploadDir('public/uploads/images')
FileField::new('attachment')

// Associations
AssociationField::new('category')
    ->autocomplete()
    ->setPreferredChoices([...])
    ->setQueryBuilder(fn(QueryBuilder $qb) => ...)

// Special
SlugField::new('slug')->setTargetFieldName('title')
EmailField::new('email')
UrlField::new('website')
ColorField::new('brandColor')
CountryField::new('country')
```

### Field Configuration Methods

All fields support:
```php
->hideOnIndex()          // Hide on list page
->hideOnForm()           // Hide on edit/new forms
->onlyOnForms()          // Show only on forms
->onlyOnIndex()          // Show only on list
->setColumns(6)          // Bootstrap grid columns
->setRequired(true)
->setHelp('Help text')
->formatValue(callable)  // Custom formatting
```

### Form Layout

```php
use EasyCorp\Bundle\EasyAdminBundle\Field\FormField;

yield FormField::addPanel('Basic Information');
yield TextField::new('name');
yield TextField::new('sku');

yield FormField::addPanel('Pricing')->setIcon('fa fa-dollar-sign');
yield MoneyField::new('price');
yield PercentField::new('taxRate');

yield FormField::addTab('Details');
yield TextEditorField::new('description');
```

---

## Filters

### Standard Filters

```php
use EasyCorp\Bundle\EasyAdminBundle\Config\Filters;
use EasyCorp\Bundle\EasyAdminBundle\Filter\*;

public function configureFilters(Filters $filters): Filters
{
    return $filters
        ->add('name')                                // Text filter
        ->add(BooleanFilter::new('isActive'))
        ->add(DateTimeFilter::new('createdAt'))
        ->add(EntityFilter::new('category'))

        // New specialized filters (v4.28+)
        ->add(CountryFilter::new('country'))
        ->add(CurrencyFilter::new('currency'))
        ->add(LanguageFilter::new('language'))
        ->add(LocaleFilter::new('locale'))
        ->add(TimezoneFilter::new('timezone'));
}
```

---

## Actions

### Built-in Actions

```php
use EasyCorp\Bundle\EasyAdminBundle\Config\{Action, Actions, Crud};

public function configureActions(Actions $actions): Actions
{
    return $actions
        // Add built-in actions
        ->add(Crud::PAGE_INDEX, Action::DETAIL)

        // Remove actions
        ->remove(Crud::PAGE_INDEX, Action::DELETE)

        // Disable globally
        ->disable(Action::DELETE)

        // Set permissions
        ->setPermission(Action::DELETE, 'ROLE_SUPER_ADMIN')

        // Update existing actions
        ->update(Crud::PAGE_INDEX, Action::EDIT, fn(Action $action) =>
            $action->displayIf(fn($entity) => $this->canEdit($entity))
        )

        // Reorder
        ->reorder(Crud::PAGE_INDEX, [Action::DETAIL, Action::EDIT, Action::DELETE]);
}
```

### Custom Actions

```php
public function configureActions(Actions $actions): Actions
{
    // Single entity action
    $export = Action::new('export', 'Export')
        ->linkToCrudAction('exportAction')
        ->setCssClass('btn btn-info')
        ->setIcon('fa fa-download')
        ->askConfirmation('Export this item?', 'Confirm');

    // Global action
    $exportAll = Action::new('exportAll', 'Export All')
        ->linkToCrudAction('exportAllAction')
        ->createAsGlobalAction()
        ->setCssClass('btn btn-success');

    // Batch action
    $batchPublish = Action::new('batchPublish', 'Publish Selected')
        ->linkToCrudAction('batchPublish')
        ->addCssClass('btn btn-success')
        ->setIcon('fa fa-check');

    return $actions
        ->add(Crud::PAGE_DETAIL, $export)
        ->add(Crud::PAGE_INDEX, $exportAll)
        ->addBatchAction($batchPublish);
}

// Action handlers
public function exportAction(AdminContext $context): Response { /* ... */ }
public function batchPublish(BatchActionDto $dto): Response { /* ... */ }
```

For complete actions reference, see [⚡ Actions API](references/actions.md).

---

## Advanced Features

### Content Security Policy (CSP)

```php
use EasyCorp\Bundle\EasyAdminBundle\Config\Assets;

public function configureAssets(): Assets
{
    return Assets::new()
        ->addCspNonce($this->getRequest()->attributes->get('csp_nonce'));
}
```

### Entity Translation

```php
public function configureCrud(Crud $crud): Crud
{
    return $crud
        ->setEntityLabelInSingular('product.singular')
        ->setEntityLabelInPlural('product.plural')
        ->setTranslationDomain('admin');
}

// translations/admin.en.yaml
// product.singular: Product
// entity.Product.field.name: Product Name
```

### Custom Row Click Behavior

```php
public function configureCrud(Crud $crud): Crud
{
    return $crud
        ->setDefaultRowAction(Action::DETAIL)
        ->setRowClickTrigger('td:first-child');  // Only first cell triggers action
        // ->setRowClickTrigger(null);  // Disable row click
}
```

### Enhanced Autocomplete

```php
$crud->autocomplete(
    enable: true,
    callback: fn($entity) => sprintf('<strong>%s</strong> (#%d)', $entity->getName(), $entity->getId()),
    template: 'admin/autocomplete/product.html.twig',
    renderAsHtml: true
);
```

### Security & Permissions

```php
public function configureCrud(Crud $crud): Crud
{
    return $crud->setEntityPermission('ROLE_ADMIN');
}

public function configureActions(Actions $actions): Actions
{
    return $actions
        ->setPermission(Action::DELETE, 'ROLE_SUPER_ADMIN')
        ->update(Crud::PAGE_INDEX, Action::EDIT, fn(Action $action) =>
            $action->displayIf(fn($entity) =>
                $this->isGranted('ROLE_EDITOR') || $entity->getAuthor() === $this->getUser()
            )
        );
}
```

### Event Listeners

```php
use EasyCorp\Bundle\EasyAdminBundle\Event\{BeforeEntityPersistedEvent, BeforeEntityUpdatedEvent};
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class EntitySubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [
            BeforeEntityPersistedEvent::class => 'setCreatedAt',
            BeforeEntityUpdatedEvent::class => 'setUpdatedAt',
        ];
    }

    public function setCreatedAt(BeforeEntityPersistedEvent $event): void
    {
        $entity = $event->getEntityInstance();
        if (method_exists($entity, 'setCreatedAt')) {
            $entity->setCreatedAt(new \DateTime());
        }
    }

    public function setUpdatedAt(BeforeEntityUpdatedEvent $event): void
    {
        $entity = $event->getEntityInstance();
        if (method_exists($entity, 'setUpdatedAt')) {
            $entity->setUpdatedAt(new \DateTime());
        }
    }
}
```

### AdminControllerRegistry

```php
use EasyCorp\Bundle\EasyAdminBundle\Registry\AdminControllerRegistry;

public function __construct(private AdminControllerRegistry $registry) {}

public function example(): void
{
    $controllers = $this->registry->getAll();                         // All CRUD controllers
    $controllerFqcn = $this->registry->getCrudControllerFqcn($entity); // Get controller for entity
    $dashboards = $this->registry->getDashboards();                   // All dashboards
}
```

---

## Common Patterns

### Multi-Tenant Applications

```php
public function createIndexQueryBuilder(/* ... */): QueryBuilder
{
    $qb = parent::createIndexQueryBuilder(...func_get_args());
    $qb->andWhere('entity.tenant = :tenant')
       ->setParameter('tenant', $this->getUser()->getTenant());
    return $qb;
}

public function persistEntity(EntityManagerInterface $em, $entity): void
{
    $entity->setTenant($this->getUser()->getTenant());
    parent::persistEntity($em, $entity);
}
```

### Soft Delete Support

```php
public function createIndexQueryBuilder(/* ... */): QueryBuilder
{
    $qb = parent::createIndexQueryBuilder(...func_get_args());
    $qb->andWhere('entity.deletedAt IS NULL');
    return $qb;
}
```

### Audit Trail

```php
public function updateEntity(EntityManagerInterface $em, $entity): void
{
    if (method_exists($entity, 'setUpdatedBy')) {
        $entity->setUpdatedBy($this->getUser());
        $entity->setUpdatedAt(new \DateTime());
    }
    parent::updateEntity($em, $entity);
}
```

### Custom Form Validation

```php
public function configureFields(string $pageName): iterable
{
    yield TextField::new('email')
        ->setFormTypeOptions([
            'constraints' => [
                new Assert\Email(),
                new Assert\NotBlank(),
            ],
        ]);
}
```

### Performance Optimization

```php
public function configureCrud(Crud $crud): Crud
{
    return $crud
        ->setPaginatorPageSize(30)           // Limit page size
        ->setSearchFields(['name', 'sku'])   // Index only searchable fields
        ->setPaginatorUseOutputWalkers(false) // Disable output walkers for better performance
        ->setPaginatorFetchJoinCollection(true); // Optimize JOIN queries
}

public function configureFields(string $pageName): iterable
{
    // Use autocomplete for large datasets
    yield AssociationField::new('category')->autocomplete();

    // Hide expensive fields on index
    yield TextEditorField::new('content')->hideOnIndex();
}
```

### Custom Templates

```php
public function configureCrud(Crud $crud): Crud
{
    return $crud
        ->overrideTemplate('crud/index', 'admin/custom_index.html.twig')
        ->overrideTemplate('crud/detail', 'admin/custom_detail.html.twig');
}
```

---

## Configuration

### Route Configuration

```yaml
# config/routes.yaml
admin:
    resource: App\Controller\Admin\DashboardController
    type: easyadmin
    prefix: /admin
```

### Security Configuration

```yaml
# config/packages/security.yaml
security:
    access_control:
        - { path: ^/admin/login, roles: PUBLIC_ACCESS }
        - { path: ^/admin, roles: ROLE_ADMIN }

    role_hierarchy:
        ROLE_EDITOR: ROLE_USER
        ROLE_ADMIN: ROLE_EDITOR
        ROLE_SUPER_ADMIN: ROLE_ADMIN
```

### EasyAdmin Configuration

```yaml
# config/packages/easyadmin.yaml
easy_admin:
    site_name: 'My Admin Panel'
    formats:
        date: 'd/m/Y'
        time: 'H:i'
        datetime: 'd/m/Y H:i:s'
    design:
        brand_color: '#1976D2'
```

---

## API Reference

Complete API documentation in separate files for quick lookup:

### [📋 Fields API](references/fields.md)
- 30+ field types with all configuration methods
- Common methods: hideOnIndex(), setColumns(), setRequired()
- Field types: TextField, AssociationField, ImageField, MoneyField, etc.
- Form layout: FormField panels, tabs, columns
- Doctrine type mappings

### [⚡ Actions API](references/actions.md)
- Built-in actions: INDEX, DETAIL, EDIT, NEW, DELETE
- Configuration: add(), remove(), update(), disable(), setPermission()
- Custom actions: linkToCrudAction(), linkToRoute(), linkToUrl()
- Batch actions: addBatchAction(), BatchActionDto
- Action groups and styling

### [🔧 CRUD API](references/crud.md)
- configureCrud() methods: setEntityLabel(), setSearchFields(), setDefaultSort()
- Query builders: createIndexQueryBuilder(), createEditQueryBuilder()
- Lifecycle: createEntity(), persistEntity(), updateEntity(), deleteEntity()
- Routes, context, and URL generation
- Events and field configurators

---

## Troubleshooting

### Routes Not Found
```bash
php bin/console cache:clear
php bin/console debug:router | grep admin
```

### Images Not Displaying
Ensure directory structure:
```
public/
  uploads/
    images/
    products/
```

### Association Fields Empty
Add `by_reference => false` for collections:
```php
AssociationField::new('tags')->setFormTypeOptions(['by_reference' => false])
```

### Autocomplete Not Working
```bash
composer require symfony/ux-autocomplete
php bin/console importmap:install
```

### Performance Issues
- Use `->autocomplete()` for large associations
- Limit search fields to indexed columns only
- Use pagination (20-50 items per page)
- Hide expensive fields on index with `->hideOnIndex()`
- Disable unnecessary features

---

## Official Documentation

- **Repository:** https://github.com/EasyCorp/EasyAdminBundle
- **Documentation:** https://symfony.com/bundles/EasyAdminBundle/4.x/index.html
- **Demo Application:** https://github.com/EasyCorp/easyadmin-demo

### Version Notes

- **4.x (Current Stable):** Production ready, actively maintained
- **5.x (Latest):** Stable release, removes deprecated features, new features only added here
- This skill covers 4.x patterns compatible with both 4.x and 5.x

### Key Documentation Links

- [CRUD Operations](https://symfony.com/bundles/EasyAdminBundle/current/crud.html)
- [Dashboard Configuration](https://symfony.com/bundles/EasyAdminBundle/current/dashboards.html)
- [Fields Reference](https://symfony.com/bundles/EasyAdminBundle/current/fields.html)
- [Actions](https://symfony.com/bundles/EasyAdminBundle/current/actions.html)
- [Filters](https://symfony.com/bundles/EasyAdminBundle/current/filters.html)
- [Security](https://symfony.com/bundles/EasyAdminBundle/current/security.html)
- [Events](https://symfony.com/bundles/EasyAdminBundle/current/events.html)
- [Design Customization](https://symfony.com/bundles/EasyAdminBundle/current/design.html)
