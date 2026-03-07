# Dashboard API Reference

Complete dashboard configuration for EasyAdminBundle 4.x.

## Overview

The Dashboard controller is the entry point for your admin panel. It configures the interface appearance, routing, and navigation menu.

## Basic Dashboard

```php
namespace App\Controller\Admin;

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
            ->setTitle('My Admin Panel');
    }

    public function configureMenuItems(): iterable
    {
        yield MenuItem::linkToDashboard('Dashboard', 'fa fa-home');
    }
}
```

## Dashboard Configuration

### `configureDashboard()` Options

```php
public function configureDashboard(): Dashboard
{
    return Dashboard::new()
        // Title
        ->setTitle('My Admin Panel')
        ->setTitle('<img src="logo.png"> My Admin')  // HTML supported

        // Favicon
        ->setFaviconPath('favicon.ico')
        ->setFaviconPath('build/favicon.png')

        // Translation
        ->setTranslationDomain('admin')              // Use custom translation domain
        ->setLocales(['en', 'fr', 'es'])            // Available locales

        // Layout
        ->renderContentMaximized()                   // Full-width content (no sidebar)
        ->renderSidebarMinimized()                   // Start with collapsed sidebar

        // Theme
        ->disableDarkMode()                          // Disable dark mode toggle
        ->setColorScheme('light')                    // Force light/dark mode

        // Help & Documentation
        ->setHelpUrl('https://docs.example.com')     // Help link in sidebar

        // Custom CSS/JS
        ->addCssFile('admin.css')
        ->addJsFile('admin.js');
}
```

## Menu Configuration

### `configureMenuItems()` Overview

```php
public function configureMenuItems(): iterable
{
    // Link to dashboard
    yield MenuItem::linkToDashboard('Dashboard', 'fa fa-home');

    // Section headers
    yield MenuItem::section('Content Management');

    // CRUD links
    yield MenuItem::linkToCrud('Posts', 'fa fa-newspaper', Post::class);
    yield MenuItem::linkToCrud('Categories', 'fa fa-tags', Category::class);

    // Submenus
    yield MenuItem::subMenu('E-commerce', 'fa fa-shopping-cart')->setSubItems([
        MenuItem::linkToCrud('Products', 'fa fa-box', Product::class),
        MenuItem::linkToCrud('Orders', 'fa fa-shopping-bag', Order::class),
    ]);

    // Custom links
    yield MenuItem::linkToRoute('Analytics', 'fa fa-chart-line', 'app_analytics');
    yield MenuItem::linkToUrl('Visit Site', 'fa fa-globe', '/');

    // Logout
    yield MenuItem::linkToLogout('Logout', 'fa fa-sign-out');
}
```

### MenuItem Types

#### 1. Dashboard Link

```php
MenuItem::linkToDashboard('Dashboard', 'fa fa-home')
    ->setPermission('ROLE_ADMIN')
    ->setBadge(3, 'info')
```

#### 2. CRUD Link

```php
MenuItem::linkToCrud('Posts', 'fa fa-newspaper', Post::class)
    ->setController(PostCrudController::class)     // Optional: specify controller
    ->setAction(Crud::PAGE_INDEX)                  // Default page
    ->setPermission('ROLE_EDITOR')
    ->setBadge($this->countPosts(), 'warning')     // Dynamic badge
    ->setQueryParameter('status', 'published')     // Add query params
```

#### 3. Section Header

```php
MenuItem::section('Content Management')
    ->setPermission('ROLE_EDITOR')
    ->setIcon('fa fa-folder')
```

#### 4. Submenu

```php
MenuItem::subMenu('E-commerce', 'fa fa-shopping-cart')
    ->setSubItems([
        MenuItem::linkToCrud('Products', 'fa fa-box', Product::class),
        MenuItem::linkToCrud('Orders', 'fa fa-shopping-bag', Order::class),
        MenuItem::linkToCrud('Customers', 'fa fa-users', Customer::class),
    ])
    ->setPermission('ROLE_ADMIN')
    ->setExpanded(true)  // Start expanded
```

#### 5. Route Link

```php
MenuItem::linkToRoute('Analytics', 'fa fa-chart-line', 'app_analytics')
    ->setQueryParameters(['year' => date('Y')])
    ->setLinkTarget('_blank')  // Open in new tab
```

#### 6. URL Link

```php
MenuItem::linkToUrl('Documentation', 'fa fa-book', 'https://docs.example.com')
    ->setLinkTarget('_blank')
    ->setLinkRel('noopener noreferrer')
```

#### 7. Controller Link

```php
MenuItem::linkTo('Custom Dashboard', 'fa fa-chart', CustomDashboardController::class)
```

#### 8. Logout Link

```php
MenuItem::linkToLogout('Logout', 'fa fa-sign-out')
    ->setLinkTarget('_self')
```

#### 9. Exit Impersonation

```php
if ($this->isGranted('ROLE_PREVIOUS_ADMIN')) {
    yield MenuItem::linkToExitImpersonation('Exit Impersonation', 'fa fa-user-times');
}
```

### MenuItem Configuration Methods

```php
MenuItem::linkToCrud('Posts', 'fa fa-newspaper', Post::class)
    // Label & Icon
    ->setLabel('Blog Posts')
    ->setIcon('fa fa-edit')

    // Permissions
    ->setPermission('ROLE_EDITOR')

    // Badges
    ->setBadge(10, 'success')                      // Static badge
    ->setBadge($this->countPendingPosts(), 'warning')  // Dynamic badge

    // Link behavior
    ->setLinkTarget('_blank')                      // Open in new window
    ->setLinkRel('noopener')

    // CSS classes
    ->setCssClass('menu-item-featured')
    ->addCssClass('highlighted')

    // Query parameters
    ->setQueryParameter('status', 'published')
    ->setQueryParameters(['year' => 2024, 'status' => 'active'])

    // HTML attributes
    ->setHtmlAttribute('data-turbo', 'false')
```

## Advanced Patterns

### Dynamic Menu Items

```php
public function configureMenuItems(): iterable
{
    yield MenuItem::linkToDashboard('Dashboard', 'fa fa-home');

    // Show different menus based on role
    if ($this->isGranted('ROLE_ADMIN')) {
        yield MenuItem::linkToCrud('Users', 'fa fa-users', User::class);
        yield MenuItem::linkToCrud('Settings', 'fa fa-cog', Setting::class);
    }

    // Dynamic menu from database
    foreach ($this->getActiveCategories() as $category) {
        yield MenuItem::linkToCrud(
            $category->getName(),
            'fa fa-folder',
            Post::class
        )->setQueryParameter('categoryId', $category->getId());
    }
}

private function getActiveCategories(): array
{
    return $this->entityManager
        ->getRepository(Category::class)
        ->findBy(['isActive' => true]);
}
```

### Badge Counts

```php
yield MenuItem::linkToCrud('Orders', 'fa fa-shopping-cart', Order::class)
    ->setBadge($this->countPendingOrders(), 'danger');

private function countPendingOrders(): int
{
    return $this->entityManager
        ->getRepository(Order::class)
        ->count(['status' => 'pending']);
}
```

### Multi-level Submenus

```php
yield MenuItem::subMenu('Settings', 'fa fa-cog')->setSubItems([
    MenuItem::subMenu('System', 'fa fa-server')->setSubItems([
        MenuItem::linkToCrud('General', 'fa fa-sliders', GeneralSetting::class),
        MenuItem::linkToCrud('Security', 'fa fa-shield', SecuritySetting::class),
    ]),
    MenuItem::linkToCrud('User Preferences', 'fa fa-user-cog', UserPreference::class),
]);
```

### Conditional Menu Visibility

```php
public function configureMenuItems(): iterable
{
    yield MenuItem::linkToDashboard('Dashboard', 'fa fa-home');

    // Show only for specific users
    if ($this->getUser()->hasRole('ROLE_CONTENT_MANAGER')) {
        yield MenuItem::linkToCrud('Posts', 'fa fa-newspaper', Post::class);
    }

    // Show based on feature flag
    if ($this->featureEnabled('analytics')) {
        yield MenuItem::linkToRoute('Analytics', 'fa fa-chart-line', 'app_analytics');
    }
}
```

### External Links with Icons

```php
yield MenuItem::section('External Resources');

yield MenuItem::linkToUrl('Documentation', 'fa fa-book', 'https://docs.example.com')
    ->setLinkTarget('_blank')
    ->setLinkRel('noopener noreferrer');

yield MenuItem::linkToUrl('Support', 'fa fa-life-ring', 'https://support.example.com')
    ->setLinkTarget('_blank');
```

## Assets Configuration

### Custom CSS & JavaScript

```php
public function configureDashboard(): Dashboard
{
    return Dashboard::new()
        ->setTitle('My Admin')
        ->addCssFile('css/admin-custom.css')
        ->addCssFile('https://cdn.example.com/theme.css')
        ->addJsFile('js/admin-custom.js')
        ->addJsFile('https://cdn.example.com/analytics.js');
}
```

### Content Security Policy (CSP)

```php
use EasyCorp\Bundle\EasyAdminBundle\Config\Assets;

public function configureAssets(): Assets
{
    return Assets::new()
        ->addCspNonce($this->getRequest()->attributes->get('csp_nonce'))
        ->addWebpackEncoreEntry('admin');  // Webpack Encore integration
}
```

### Asset Packages

```php
public function configureDashboard(): Dashboard
{
    return Dashboard::new()
        ->setAssetPackage('admin_assets')  // Use custom asset package
        ->addCssFile('admin.css', 'admin_assets')
        ->addJsFile('admin.js', 'admin_assets');
}
```

## Dashboard Index Page

### Custom Dashboard Content

```php
#[Route('/admin', name: 'admin')]
public function index(): Response
{
    // Option 1: Render custom template
    return $this->render('admin/dashboard.html.twig', [
        'stats' => $this->getDashboardStats(),
    ]);

    // Option 2: Redirect to a CRUD index
    return $this->redirect($this->generateUrl('admin', [
        'crudAction' => Crud::PAGE_INDEX,
        'crudControllerFqcn' => PostCrudController::class,
    ]));

    // Option 3: Redirect to first menu item
    $routeBuilder = $this->container->get(AdminUrlGenerator::class);
    return $this->redirect($routeBuilder->setController(PostCrudController::class)->generateUrl());
}
```

### Dashboard Statistics

```php
#[Route('/admin', name: 'admin')]
public function index(): Response
{
    $stats = [
        'total_users' => $this->entityManager->getRepository(User::class)->count([]),
        'total_posts' => $this->entityManager->getRepository(Post::class)->count([]),
        'pending_orders' => $this->entityManager->getRepository(Order::class)
            ->count(['status' => 'pending']),
    ];

    return $this->render('admin/dashboard.html.twig', ['stats' => $stats]);
}
```

## Translation

### Translation Domain

```php
public function configureDashboard(): Dashboard
{
    return Dashboard::new()
        ->setTitle('admin.title')                // Translation key
        ->setTranslationDomain('admin');         // Use translations/admin.yaml
}
```

Translation file (`translations/admin.en.yaml`):
```yaml
admin.title: My Admin Panel
menu.dashboard: Dashboard
menu.content: Content Management
menu.posts: Posts
```

### Locale Selector

```php
public function configureDashboard(): Dashboard
{
    return Dashboard::new()
        ->setTitle('My Admin')
        ->setLocales(['en', 'fr', 'es', 'de'])   // Available locales
        ->setDefaultLocale('en');
}
```

## User Menu

### Custom User Menu

```php
use EasyCorp\Bundle\EasyAdminBundle\Config\UserMenu;
use Symfony\Component\Security\Core\User\UserInterface;

public function configureUserMenu(UserInterface $user): UserMenu
{
    return parent::configureUserMenu($user)
        ->setName($user->getFullName())
        ->setAvatarUrl($user->getAvatarUrl())
        ->addMenuItems([
            MenuItem::linkToRoute('My Profile', 'fa fa-user', 'app_profile'),
            MenuItem::linkToRoute('Settings', 'fa fa-cog', 'app_settings'),
            MenuItem::section(),
            MenuItem::linkToLogout('Logout', 'fa fa-sign-out'),
        ]);
}
```

## Configuration Files

### Route Configuration

```yaml
# config/routes.yaml
admin:
    resource: App\Controller\Admin\DashboardController
    type: easyadmin
    prefix: /admin

# Multiple dashboards
admin_main:
    resource: App\Controller\Admin\MainDashboardController
    type: easyadmin
    prefix: /admin

admin_reports:
    resource: App\Controller\Admin\ReportsDashboardController
    type: easyadmin
    prefix: /admin/reports
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

> **Note:** EasyAdmin 4.x uses PHP configuration in the Dashboard controller. YAML configuration (`config/packages/easyadmin.yaml`) was used in EasyAdmin 3.x and earlier.

All settings are configured via `configureDashboard()`, `configureAssets()`, and `configureUserMenu()` methods in your Dashboard controller (see examples above).

## Multiple Dashboards

```php
// Main Dashboard
class MainDashboardController extends AbstractDashboardController
{
    #[Route('/admin', name: 'admin_main')]
    public function index(): Response { /* ... */ }
}

// Reports Dashboard
class ReportsDashboardController extends AbstractDashboardController
{
    #[Route('/admin/reports', name: 'admin_reports')]
    public function index(): Response { /* ... */ }

    public function configureMenuItems(): iterable
    {
        yield MenuItem::linkToRoute('Back to Main', 'fa fa-arrow-left', 'admin_main');
        yield MenuItem::section('Reports');
        yield MenuItem::linkToRoute('Sales', 'fa fa-dollar', 'reports_sales');
    }
}
```

## Troubleshooting

- **Menu items not showing**: Check permissions and `isGranted()` calls
- **Icons not displaying**: Ensure Font Awesome is loaded or use custom icon set
- **Badges not updating**: Clear cache or check badge calculation logic
- **CSP violations**: Add nonce to `configureAssets()`
- **Translation not working**: Set correct `setTranslationDomain()` and create translation files
