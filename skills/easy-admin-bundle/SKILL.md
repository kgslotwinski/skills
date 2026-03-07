---
name: easy-admin-bundle
version: 1.2.1
compatibility:
  easyadmin: "^4.0"
  php: ">=8.1"
  symfony: ">=5.4"
updated: 2026-03-07
description: "Rapid Symfony admin panel development with EasyAdminBundle. Capabilities: CRUD generation, dashboard creation, entity management, form customization, menu configuration, field configuration, action customization, filters, permissions, batch actions, custom themes, file uploads, image handling, associations, submenus, custom queries, event listeners. Use for: admin panels, backend interfaces, content management, data administration, CRUD operations, user management, product catalogs, blog administration, e-commerce backends. Triggers: easyadmin, symfony admin, crud, admin panel, dashboard, backend, entity crud, admin interface, symfony backend, easyadmin, easyadmin bundle, admin generator, backend panel"
allowed-tools: Bash(composer *, symfony *, php bin/console *), Read, Write, Edit, Glob, Grep
---

# EasyAdmin Bundle Skill

Build Symfony admin panels with EasyAdminBundle 4.x. Quick examples and patterns for CRUD interfaces.

**Requirements:** PHP 8.1+, Symfony 5.4+, Doctrine ORM

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

**Complete API**: [Fields](references/fields.md) | [Actions](references/actions.md) | [CRUD](references/crud.md) | [Filters](references/filters.md) | [Dashboard](references/dashboard.md)

---

## Dashboard Setup

```php
namespace App\Controller\Admin;

use EasyCorp\Bundle\EasyAdminBundle\Config\{Dashboard, MenuItem};
use EasyCorp\Bundle\EasyAdminBundle\Controller\AbstractDashboardController;

class DashboardController extends AbstractDashboardController
{
    #[Route('/admin', name: 'admin')]
    public function index(): Response
    {
        return $this->render('admin/dashboard.html.twig');
    }

    public function configureDashboard(): Dashboard
    {
        return Dashboard::new()->setTitle('My Admin Panel');
    }

    public function configureMenuItems(): iterable
    {
        yield MenuItem::linkToDashboard('Dashboard', 'fa fa-home');
        yield MenuItem::linkToCrud('Posts', 'fa fa-newspaper', Post::class);
        yield MenuItem::linkToCrud('Categories', 'fa fa-tags', Category::class);
    }
}
```

---

## CRUD Controllers

```php
namespace App\Controller\Admin;

use App\Entity\Product;
use EasyCorp\Bundle\EasyAdminBundle\Controller\AbstractCrudController;
use EasyCorp\Bundle\EasyAdminBundle\Field\*;

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
            ->setSearchFields(['name', 'sku'])
            ->setDefaultSort(['createdAt' => 'DESC']);
    }

    public function configureFields(string $pageName): iterable
    {
        yield TextField::new('name');
        yield MoneyField::new('price')->setCurrency('USD');
        yield AssociationField::new('category')->autocomplete();
        yield ImageField::new('photo')
            ->setBasePath('uploads')
            ->setUploadDir('public/uploads');
        yield BooleanField::new('isActive')->renderAsSwitch();
    }
}
```

**Add filters**:
```php
public function configureFilters(Filters $filters): Filters
{
    return $filters
        ->add('name')
        ->add(BooleanFilter::new('isActive'));
}
```

See [Fields](references/fields.md), [Actions](references/actions.md), [CRUD](references/crud.md), [Filters](references/filters.md) for complete examples

---

## Common Patterns

### Custom Query Filtering
```php
public function createIndexQueryBuilder(/* ... */): QueryBuilder
{
    $qb = parent::createIndexQueryBuilder(...func_get_args());
    $qb->andWhere('entity.isActive = :active')
       ->setParameter('active', true);
    return $qb;
}
```

### Audit Trail
```php
public function updateEntity(EntityManagerInterface $em, $entity): void
{
    $entity->setUpdatedBy($this->getUser());
    parent::updateEntity($em, $entity);
}
```

### Security
```php
return $crud->setEntityPermission('ROLE_ADMIN');
return $actions->setPermission(Action::DELETE, 'ROLE_SUPER_ADMIN');
```

More patterns in [CRUD API](references/crud.md)

## Configuration

```yaml
# config/routes.yaml
admin:
    resource: App\Controller\Admin\DashboardController
    type: easyadmin
    prefix: /admin

# config/packages/security.yaml
security:
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
```

## Troubleshooting

- **Routes not found**: `php bin/console cache:clear`
- **Images not displaying**: Check `public/uploads/` exists
- **Collections not saving**: Add `->setFormTypeOptions(['by_reference' => false])`
- **Autocomplete not working**: `composer require symfony/ux-autocomplete`

## Resources

- **Docs**: https://symfony.com/bundles/EasyAdminBundle/current/index.html
- **Repo**: https://github.com/EasyCorp/EasyAdminBundle
