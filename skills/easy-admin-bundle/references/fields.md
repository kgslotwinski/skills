# EasyAdmin Fields API Reference

Complete reference for all 30+ field types and their configuration methods.

## Table of Contents

- [Common Configuration](#common-configuration)
- [Field Types](#field-types)
- [Form Layout](#form-layout)
- [Doctrine Mappings](#doctrine-mappings)

---

## Common Configuration

All fields inherit these methods:

### Display Control
```php
->hideOnIndex()          // Hide on list page
->hideOnDetail()         // Hide on detail page
->hideOnForm()           // Hide on create/edit forms
->hideWhenCreating()     // Hide on create form only
->hideWhenUpdating()     // Hide on edit form only
->onlyOnIndex()          // Show on list page only
->onlyOnDetail()         // Show on detail page only
->onlyOnForms()          // Show on create/edit forms only
->onlyWhenCreating()     // Show on create form only
->onlyWhenUpdating()     // Show on edit form only
```

### Formatting & Layout
```php
->setColumns(int|string $cols)          // Column width: int (6) or string ('col-sm-6 col-lg-4')
->formatValue(callable $callback)       // Custom formatter: fn($value, $entity): mixed
->setTemplatePath(string $path)         // Custom Twig template
->setTextAlign('left'|'right'|'center') // Text alignment
->setCssClass(string $class)            // Replace CSS classes
->addCssClass(string $class)            // Add CSS class
```

### Form Options
```php
->setRequired(bool $required = true)
->setHelp(string $help)                 // Help text below field
->setEmptyData($value)                  // Default value when empty
->setFormType(string $formTypeClass)    // Symfony form type
->setFormTypeOptions(array $options)    // Form type options
->setHtmlAttribute(string $name, $value)
->addFormTheme(string ...$themes)
```

### Other
```php
->setSortable(bool $sortable)           // Enable/disable column sorting
->setLabel(string|false|null $label)    // false: no label, null: auto-generate
->addCssFiles(string ...$files)
->addJsFiles(string ...$files)
```

---

## Field Types

### Text Fields

#### TextField
Single-line text input.
```php
TextField::new('title')
    ->setMaxLength(100)        // Max display length (read-only pages)
    ->renderAsHtml()           // Don't escape HTML
    ->stripTags()              // Strip HTML tags
```

#### TextareaField
Multi-line text input.
```php
TextareaField::new('description')
    ->setMaxLength(500)
    ->setNumOfRows(5)
    ->stripTags()
```

#### TextEditorField
WYSIWYG HTML editor (Trix).
```php
TextEditorField::new('content')
    ->setNumOfRows(15)
    ->setTrixEditorConfig([...])
```

#### CodeEditorField
Code editor with syntax highlighting (CodeMirror).
```php
CodeEditorField::new('customCss')
    ->setLanguage('css')
    ->setNumOfRows(10)
    ->setIndentWithTabs(true)
```

#### EmailField
Email with mailto link.
```php
EmailField::new('email')
```

#### UrlField
URL as clickable link.
```php
UrlField::new('website')
```

#### TelephoneField
Phone number as tel link.
```php
TelephoneField::new('phone')
```

---

### Numeric Fields

#### IntegerField
Integer numbers.
```php
IntegerField::new('quantity')
    ->setNumberFormat('%d')
    ->setThousandsSeparator(',')
```

#### NumberField
Decimal numbers.
```php
NumberField::new('price')
    ->setNumDecimals(2)
    ->setRoundingMode(\NumberFormatter::ROUND_HALFUP)
    ->setThousandsSeparator(',')
    ->setDecimalSeparator('.')
```

#### MoneyField
Monetary values with currency.
```php
MoneyField::new('price')
    ->setCurrency('USD')
    ->setStoredAsCents(false)    // true if stored as cents
    ->setCustomPattern('¤#,##0.00')
    ->setNumDecimals(2)
```

#### PercentField
Percentage values.
```php
PercentField::new('discount')
    ->setNumDecimals(2)
    ->setStoredAsFractional(true)  // true: 0-1, false: 0-100
    ->setSymbol('%')
```

---

### Date & Time Fields

#### DateField
Date values.
```php
DateField::new('publishedAt')
    ->setFormat('yyyy-MM-dd')         // ICU pattern or 'short','medium','long','full'
    ->setTimezone('UTC')
    ->renderAsChoice()                // Separate selects for day/month/year
    ->renderAsNativeWidget()          // HTML5 date input
    ->renderAsText()                  // Text input
```

#### DateTimeField
Date and time values.
```php
DateTimeField::new('createdAt')
    ->setFormat('yyyy-MM-dd HH:mm:ss')
    ->setTimezone('America/New_York')
    ->renderAsChoice()
    ->renderAsNativeWidget()
    ->renderAsText()
```

#### TimeField
Time values.
```php
TimeField::new('openingTime')
    ->setFormat('HH:mm')
    ->renderAsChoice()
    ->renderAsNativeWidget()
```

---

### Boolean & Choice Fields

#### BooleanField
Boolean values as Yes/No or toggle.
```php
BooleanField::new('isActive')
    ->renderAsSwitch(true)  // Toggle switch in forms
```

#### ChoiceField
Predefined choices.
```php
ChoiceField::new('status')
    ->setChoices([
        'Draft' => 'draft',
        'Published' => 'published',
    ])
    ->setPreferredChoices(['published'])   // Display at top
    ->allowMultipleChoices()
    ->renderAsBadges([                      // Badges on read-only pages
        'draft' => 'warning',
        'published' => 'success',
    ])
    ->renderExpanded()                      // Radio buttons/checkboxes
    ->renderAsNativeWidget()                // Standard <select>
    ->autocomplete()                        // Autocomplete dropdown
    ->escapeHtml(false)
```

---

### Association Fields

#### AssociationField
Doctrine entity relationships.
```php
AssociationField::new('category')
    ->autocomplete()                        // Ajax autocomplete
    ->renderAsNativeWidget()                // Standard <select>
    ->renderAsEmbeddedForm()                // Embed related entity form
    ->renderAsHtml()                        // Don't escape HTML
    ->setCrudController(CategoryCrudController::class)
    ->setQueryBuilder(fn(QueryBuilder $qb) =>
        $qb->andWhere('entity.isActive = :active')
           ->setParameter('active', true)
    )
    ->setPreferredChoices([...])            // Display at top
    ->setSortProperty('name')               // Sort property for display
    ->setFormTypeOptions(['by_reference' => false]) // For collections
```

---

### File & Image Fields

#### ImageField
Image upload with preview.
```php
ImageField::new('thumbnail')
    ->setBasePath('uploads/images')
    ->setUploadDir('public/uploads/images')
    ->setUploadedFileNamePattern('[randomhash].[extension]')
    // Patterns: [name], [slug], [extension], [timestamp], [day], [month], [year], [randomhash], [uuid]
    ->setUploadFilename(fn(UploadedFile $file) => ...)
```

#### FileField
File upload.
```php
FileField::new('attachment')
    ->setBasePath('uploads/files')
    ->setUploadDir('public/uploads/files')
    ->setUploadedFileNamePattern('[slug]-[timestamp].[extension]')
```

#### AvatarField
User avatars with Gravatar support.
```php
AvatarField::new('avatar')
    ->setHeight(40)
    ->setIsGravatarEmail()   // Value is Gravatar email
    ->setIsGravatarUrl()     // Value is Gravatar URL
```

---

### Special Purpose Fields

#### IdField
Entity ID.
```php
IdField::new('id')->hideOnForm()
```

#### SlugField
URL slugs with auto-generation.
```php
SlugField::new('slug')
    ->setTargetFieldName('title')
    ->setUnlockConfirmationMessage('Are you sure?')
```

#### ColorField
Color picker.
```php
ColorField::new('brandColor')
```

#### ArrayField
Array values displayed as comma-separated list.
```php
ArrayField::new('roles')
```

#### CollectionField
Collections of embedded forms.
```php
CollectionField::new('addresses')
    ->setEntryType(AddressType::class)
    ->allowAdd()
    ->allowDelete()
    ->setEntryIsComplex(true)
    ->renderExpanded()
    ->showEntryLabel(true)
```

---

### Location & Language Fields

#### CountryField
Country selector.
```php
CountryField::new('country')
    ->showFlag()
    ->showName()
    ->useAlpha3Codes()                      // ISO 3166-1 alpha-3
    ->setIncludeOnly(['US', 'CA', 'MX'])
```

#### CurrencyField
Currency codes.
```php
CurrencyField::new('currency')
    ->setCurrencyCodeVisible(true)
```

#### LanguageField
Language selector.
```php
LanguageField::new('language')
    ->showCode()
    ->showName()
    ->useAlpha3Codes()                      // ISO 639-2 alpha-3
    ->setIncludeOnly(['en', 'es', 'fr'])
```

#### LocaleField
Locale selector.
```php
LocaleField::new('locale')
    ->showCode()
    ->showName()
    ->setIncludeOnly(['en_US', 'es_ES'])
```

#### TimezoneField
Timezone selector.
```php
TimezoneField::new('timezone')
```

---

## Form Layout

### FormField Methods

#### addPanel()
Create a panel to group fields.
```php
FormField::addPanel('Basic Information')
    ->setIcon('fa fa-info')
    ->setHelp('Enter the basic details')
    ->addCssClass('custom-panel')
```

#### addTab()
Create a tab.
```php
FormField::addTab('General')
    ->setIcon('fa fa-home')
    ->setHelp('General settings')
```

#### addColumn()
Create a column layout.
```php
FormField::addColumn(6)                    // col-md-6
FormField::addColumn('col-sm-12 col-lg-4')
```

#### addFieldset()
Create a collapsible fieldset.
```php
FormField::addFieldset('Advanced Options')
    ->setIcon('fa fa-cog')
    ->setHelp('Advanced configuration')
    ->collapsible()                        // Collapsible, expanded by default
    ->renderCollapsed()                    // Collapsible, collapsed by default
```

#### addRow()
Force a new row.
```php
FormField::addRow()
FormField::addRow('lg')                    // Only at 'lg' breakpoint
```

### Layout Example
```php
public function configureFields(string $pageName): iterable
{
    yield FormField::addTab('Basic Info');

    yield FormField::addPanel('Product Details');
    yield TextField::new('name');
    yield SlugField::new('slug')->setTargetFieldName('name');

    yield FormField::addPanel('Pricing')->setIcon('fa fa-dollar-sign');
    yield FormField::addColumn(6);
    yield MoneyField::new('price')->setCurrency('USD');
    yield FormField::addColumn(6);
    yield IntegerField::new('stock');

    yield FormField::addTab('SEO');
    yield TextField::new('metaTitle');
    yield TextareaField::new('metaDescription');
}
```

---

## Doctrine Mappings

| Doctrine Type | Recommended Field |
|--------------|-------------------|
| string, ascii_string | TextField |
| text | TextareaField, TextEditorField, CodeEditorField |
| integer, smallint | IntegerField |
| bigint | TextField |
| decimal, float | NumberField |
| boolean | BooleanField |
| date, date_immutable | DateField |
| datetime, datetime_immutable, datetimetz | DateTimeField |
| time, time_immutable | TimeField |
| array, simple_array, json | ArrayField, CodeEditorField |
| guid, uuid | TextField |

---

## Quick Tips

1. **Performance:** Use `->hideOnIndex()` for expensive fields (TextEditor, associations)
2. **Associations:** Use `->autocomplete()` for large datasets
3. **Collections:** Always add `->setFormTypeOptions(['by_reference' => false])`
4. **Images:** Use patterns like `[slug]-[timestamp].[extension]` for unique filenames
5. **Sorting:** Disable sorting on computed fields with `->setSortable(false)`
6. **Security:** Use `->setFormTypeOptions(['attr' => ['readonly' => true]])` for read-only fields
7. **Validation:** Add constraints via `->setFormTypeOptions(['constraints' => [...]])`
