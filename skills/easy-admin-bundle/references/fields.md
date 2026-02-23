# EasyAdmin Fields API Reference

## Field Configuration

All fields share common configuration methods:

### Display Control
- `hideOnIndex()` - Hide field on index page
- `hideOnDetail()` - Hide field on detail page
- `hideOnForm()` - Hide field on edit and new pages
- `hideWhenCreating()` - Hide field on new page only
- `hideWhenUpdating()` - Hide field on edit page only
- `onlyOnIndex()` - Show field on index page only
- `onlyOnDetail()` - Show field on detail page only
- `onlyOnForms()` - Show field on edit and new pages only
- `onlyWhenCreating()` - Show field on new page only
- `onlyWhenUpdating()` - Show field on edit page only

### Design Options
- `addCssClass(string $cssClass)` - Add CSS class to existing classes
- `setCssClass(string $cssClass)` - Replace all CSS classes
- `setTemplatePath(string $path)` - Custom Twig template for rendering
- `setTextAlign(string $align)` - Text alignment (left, right, center)
- `addFormTheme(string ...$themes)` - Add form theme for rendering
- `addCssFiles(string|Asset ...$files)` - Load CSS files
- `addJsFiles(string|Asset ...$files)` - Load JavaScript files
- `addWebpackEncoreEntry(string|Asset $entry)` - Load Webpack Encore entry
- `addHtmlContentsToHead(string $html)` - Add HTML to <head>
- `addHtmlContentsToBody(string $html)` - Add HTML to <body>

### Formatting Options
- `formatValue(callable $callback)` - Apply callback to value before rendering
  - Callback signature: `fn($value, $entity): mixed`
- `setColumns(int|string $cols)` - Set field width in form rows
  - Integer: N → 'col-md-N'
  - String: Bootstrap grid classes (e.g., 'col-sm-6 col-lg-4')

### Misc Options
- `setSortable(bool $sortable)` - Enable/disable sorting (default: true)
- `setHelp(string $help)` - Help message for forms
- `setEmptyData($value)` - Default value when empty
- `setFormType(string $formTypeClass)` - Symfony form type class
- `setFormTypeOptions(array $options)` - Options for Symfony form type
- `setHtmlAttribute(string $name, $value)` - Add HTML attribute
- `setHtmlAttributes(array $attributes)` - Add multiple HTML attributes

### Label Options
Constructor: `Field::new(string $propertyName, string|false|null $label = null)`

- `null` - Auto-generate label from property name
- `''` (empty string) - No label, but renders empty <label> element
- `false` - No label and no <label> element
- `string` - Custom label (can include HTML tags)

## Field Types

### ArrayField
Displays array values as comma-separated list or badge collection.

**Options:**
- None specific

### AssociationField
Displays Doctrine entity relationships with autocomplete widget.

**Options:**
- `autocomplete(bool $enable = true, ?callable $callback = null, ?string $template = null, bool $renderAsHtml = false)` - Enable dynamic loading via Ajax
- `renderAsNativeWidget()` - Use standard <select> element
- `renderAsEmbeddedForm(?string $crudController = null, ?string $newPageName = null, ?string $editPageName = null)` - Embed form fields of related entity
- `renderAsHtml()` - Don't escape HTML in select items
- `setCrudController(string $crudController)` - CRUD controller for links
- `setPreferredChoices(array|callable $choices)` - Display choices at top
- `setQueryBuilder(callable $callback)` - Custom query builder
- `setSortProperty(string $property)` - Property to sort by in index page

### AvatarField
Displays user avatar images.

**Options:**
- `setHeight(int $height)` - Avatar height in pixels
- `setIsGravatarEmail()` - Use value as Gravatar email
- `setIsGravatarUrl()` - Use value as Gravatar URL

### BooleanField
Displays boolean values as Yes/No or custom labels.

**Options:**
- `renderAsSwitch(bool $asSwitch = true)` - Render as toggle switch in forms

### ChoiceField
Displays values from a predefined set of choices.

**Options:**
- `allowMultipleChoices()` - Allow multiple selections
- `autocomplete()` - Enable autocomplete filtering
- `escapeHtml(bool $escape)` - Escape HTML in choice labels
- `renderAsBadges(bool|array|callable $config = true)` - Render as badges in read-only pages
- `renderAsNativeWidget()` - Use standard <select> element
- `renderExpanded()` - Show all choices as radio buttons or checkboxes
- `setChoices(array|callable $choices)` - Set available choices ['Label' => 'value']
- `setPreferredChoices(array|callable $choices)` - Display choices at top
- `setTranslatableChoices(array $choices)` - Set choices with TranslatableMessage ['value' => label]

### CodeEditorField
Code editor with syntax highlighting (powered by CodeMirror).

**Options:**
- `setLanguage(string $language)` - Programming language for syntax highlighting
- `setNumOfRows(int $rows)` - Number of visible rows
- `setIndentWithTabs(bool $indentWithTabs)` - Use tabs for indentation

### CollectionField
Displays and edits collections of embedded forms.

**Options:**
- `allowAdd(bool $allow = true)` - Allow adding new items
- `allowDelete(bool $allow = true)` - Allow deleting items
- `setEntryIsComplex(bool $isComplex = true)` - Use complex rendering for entries
- `setEntryType(string $entryType)` - Form type for collection entries
- `renderExpanded()` - Render collection expanded by default
- `showEntryLabel(bool $show = true)` - Show labels for entries

### ColorField
Color picker field.

**Options:**
- None specific

### CountryField
Displays country selector with flags.

**Options:**
- `showFlag()` - Display country flag
- `showName()` - Display country name
- `setIncludeOnly(array $countries)` - Only include specific countries
- `useAlpha3Codes()` - Use ISO 3166-1 alpha-3 codes

### CurrencyField
Displays currency codes.

**Options:**
- `setCurrencyCodeVisible(bool $visible)` - Show/hide currency code

### DateField
Displays date values.

**Options:**
- `renderAsChoice()` - Render as separate select elements (day, month, year)
- `renderAsNativeWidget(bool $asNative = true)` - Use HTML5 date input
- `renderAsText()` - Render as text input
- `setFormat(string $dateFormatOrPattern)` - Date format ('short', 'medium', 'long', 'full', or ICU pattern)
- `setTimezone(string $timezone)` - Timezone for display

### DateTimeField
Displays date and time values.

**Options:**
- `renderAsChoice()` - Render as separate select elements
- `renderAsNativeWidget(bool $asNative = true)` - Use HTML5 datetime-local input
- `renderAsText()` - Render as text input
- `setFormat(string $dateFormatOrPattern, string $timeFormat = null)` - Date/time format
- `setTimezone(string $timezone)` - Timezone for display

### EmailField
Displays email addresses as clickable mailto links.

**Options:**
- None specific

### HiddenField
Hidden form field.

**Options:**
- None specific

### IdField
Displays entity ID.

**Options:**
- None specific

### ImageField
Displays images with preview.

**Options:**
- `setBasePath(string $path)` - Base path for images
- `setUploadDir(string $path)` - Upload directory path
- `setUploadedFileNamePattern(string $pattern)` - Filename pattern for uploads
- `setUploadFilename(callable $callback)` - Custom upload filename callback

### IntegerField
Displays integer numbers.

**Options:**
- `setNumberFormat(string $format)` - sprintf() format string
- `setThousandsSeparator(string $separator)` - Thousands separator character

### LanguageField
Language selector field.

**Options:**
- `showCode()` - Display language code
- `showName()` - Display language name
- `setIncludeOnly(array $languages)` - Only include specific languages
- `useAlpha3Codes()` - Use ISO 639-2 alpha-3 codes

### LocaleField
Locale selector field.

**Options:**
- `showCode()` - Display locale code
- `showName()` - Display locale name
- `setIncludeOnly(array $locales)` - Only include specific locales

### MoneyField
Displays monetary values with currency.

**Options:**
- `setCurrency(string $currencyCode)` - Currency code (ISO 4217)
- `setCustomPattern(string $pattern)` - Custom number format pattern
- `setNumDecimals(int $num)` - Number of decimal places
- `setRoundingMode(int $mode)` - Rounding mode (PHP NumberFormatter constants)
- `setStoredAsCents(bool $asCents = true)` - Value stored as cents/smallest unit

### NumberField
Displays numeric values.

**Options:**
- `setNumberFormat(string $format)` - sprintf() format string
- `setNumDecimals(int $num)` - Number of decimal places
- `setRoundingMode(int $mode)` - Rounding mode (PHP NumberFormatter constants)
- `setThousandsSeparator(string $separator)` - Thousands separator character
- `setDecimalSeparator(string $separator)` - Decimal separator character

### PercentField
Displays percentage values.

**Options:**
- `setNumDecimals(int $num)` - Number of decimal places
- `setRoundingMode(int $mode)` - Rounding mode
- `setStoredAsFractional(bool $asFractional = true)` - Value stored as 0-1 fraction (not 0-100)
- `setSymbol(string $symbol)` - Percentage symbol (default: '%')

### SlugField
Displays URL slugs with automatic generation.

**Options:**
- `setTargetFieldName(string $fieldName)` - Field to generate slug from
- `setUnlockConfirmationMessage(string $message)` - Confirmation message when unlocking

### TelephoneField
Displays telephone numbers as clickable tel links.

**Options:**
- None specific

### TextareaField
Multi-line text input.

**Options:**
- `setMaxLength(int $length)` - Maximum display length (read-only pages)
- `setNumOfRows(int $rows)` - Number of visible rows in forms
- `stripTags()` - Strip HTML tags before displaying

### TextEditorField
WYSIWYG HTML editor (powered by Trix).

**Options:**
- `setNumOfRows(int $rows)` - Number of visible rows
- `setTrixEditorConfig(array $config)` - Trix editor configuration

### TextField
Single-line text input.

**Options:**
- `renderAsHtml()` - Don't escape HTML in read-only pages
- `setMaxLength(int $length)` - Maximum display length (read-only pages)
- `stripTags()` - Strip HTML tags before displaying

### TimeField
Displays time values.

**Options:**
- `renderAsChoice()` - Render as separate select elements (hour, minute)
- `renderAsNativeWidget(bool $asNative = true)` - Use HTML5 time input
- `renderAsText()` - Render as text input
- `setFormat(string $timeFormatOrPattern)` - Time format
- `setTimezone(string $timezone)` - Timezone for display

### TimezoneField
Timezone selector field.

**Options:**
- None specific

### UrlField
Displays URLs as clickable links.

**Options:**
- None specific

## Form Layout Fields

### FormField::addTab()
Creates a tab to group fields.

**Method:** `FormField::addTab(?string $label = null, ?string $icon = null, ?string $propertySuffix = null)`

**Methods:**
- `setIcon(string $icon)` - FontAwesome icon class
- `addCssClass(string $class)` - Add CSS class
- `setHelp(string $help)` - Help message

### FormField::addColumn()
Creates a column layout (Bootstrap grid).

**Method:** `FormField::addColumn(int|string $cols, $label = null, ?string $icon = null, ?string $help = null, ?string $propertySuffix = null)`

**Arguments:**
- `$cols` - Column width (int → 'col-md-N' or string with Bootstrap classes)
- `$label` - Optional column title
- `$icon` - FontAwesome icon class
- `$help` - Help content

### FormField::addFieldset()
Creates a fieldset to group related fields.

**Method:** `FormField::addFieldset($label = null, ?string $icon = null, ?string $propertySuffix = null)`

**Methods:**
- `setIcon(string $icon)` - FontAwesome icon class
- `addCssClass(string $class)` - Add CSS class
- `setHelp(string $help)` - Help message
- `collapsible()` - Make fieldset collapsible (expanded by default)
- `renderCollapsed()` - Make fieldset collapsible and collapsed by default

### FormField::addRow()
Forces a new row in form layout.

**Method:** `FormField::addRow(?string $breakpoint = null, ?string $propertySuffix = null)`

**Arguments:**
- `$breakpoint` - Only add row at specific breakpoint ('xl', 'lg', etc.)

## Doctrine Type Mapping

| Doctrine Type | Recommended Field(s) |
|---------------|---------------------|
| array | ArrayField |
| ascii_string | TextField |
| bigint | TextField |
| boolean | BooleanField |
| date, date_immutable | DateField |
| datetime, datetime_immutable | DateTimeField |
| datetimetz, datetimetz_immutable | DateTimeField |
| dateinterval | TextField |
| decimal | NumberField |
| float | NumberField |
| guid | TextField |
| integer | IntegerField |
| json_array, json | TextField, TextareaField, CodeEditorField, ArrayField |
| object | TextField, TextareaField, CodeEditorField |
| simple_array | ArrayField |
| smallint | IntegerField |
| string | TextField |
| text | TextareaField, TextEditorField, CodeEditorField |
| time, time_immutable | TimeField |
