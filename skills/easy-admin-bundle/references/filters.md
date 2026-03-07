# Filters API Reference

Complete filters configuration for EasyAdminBundle 4.x.

## Overview

Filters add search/filter capabilities to the index page. EasyAdmin provides multiple filter types for common use cases.

## Basic Usage

```php
use EasyCorp\Bundle\EasyAdminBundle\Config\Filters;
use EasyCorp\Bundle\EasyAdminBundle\Filter\*;

public function configureFilters(Filters $filters): Filters
{
    return $filters
        ->add('name')                              // Auto-detect filter type from property
        ->add(BooleanFilter::new('isActive'))
        ->add(DateTimeFilter::new('createdAt'))
        ->add(EntityFilter::new('category'))
        ->add(NumericFilter::new('price'));
}
```

## All Filter Types

### Text Filters

```php
// TextFilter - default for string properties
->add('title')
->add(TextFilter::new('description'))
```

### Numeric Filters

```php
// NumericFilter - for integer/decimal properties
->add(NumericFilter::new('price'))
->add(NumericFilter::new('quantity'))
```

### Boolean Filters

```php
// BooleanFilter - for boolean properties
->add(BooleanFilter::new('isActive'))
->add(BooleanFilter::new('isFeatured'))
    ->setFormTypeOption('choices', [
        'Active' => true,
        'Inactive' => false,
    ])
```

### Date/Time Filters

```php
// DateTimeFilter - for datetime properties
->add(DateTimeFilter::new('createdAt'))
->add(DateTimeFilter::new('publishedAt'))

// DateFilter - for date-only properties
->add(DateFilter::new('birthDate'))

// TimeFilter - for time-only properties
->add(TimeFilter::new('openingTime'))
```

### Entity/Association Filters

```php
// EntityFilter - for associations (ManyToOne, ManyToMany)
->add(EntityFilter::new('category'))
    ->setFormTypeOption('value_type_options.multiple', true)  // Multiple selection
    ->setFormTypeOption('value_type_options.attr.data-widget', 'select2')  // Autocomplete

->add(EntityFilter::new('author'))
    ->setFormTypeOption('value_type_options.class', User::class)
    ->setFormTypeOption('value_type_options.query_builder', function (UserRepository $repo) {
        return $repo->createQueryBuilder('u')
            ->where('u.isActive = true')
            ->orderBy('u.name', 'ASC');
    })
```

### Choice Filters

```php
// ChoiceFilter - for predefined choices
->add(ChoiceFilter::new('status'))
    ->setChoices([
        'Draft' => 'draft',
        'Published' => 'published',
        'Archived' => 'archived',
    ])
    ->setFormTypeOption('multiple', true)  // Allow multiple selection
```

### Array Filters

```php
// ArrayFilter - for JSON/array properties
->add(ArrayFilter::new('tags'))
->add(ArrayFilter::new('permissions'))
```

### Special Filters (v4.28+)

```php
// CountryFilter - for country codes
->add(CountryFilter::new('country'))

// CurrencyFilter - for currency codes
->add(CurrencyFilter::new('currency'))

// LanguageFilter - for language codes
->add(LanguageFilter::new('language'))

// LocaleFilter - for locale codes
->add(LocaleFilter::new('locale'))

// TimezoneFilter - for timezone identifiers
->add(TimezoneFilter::new('timezone'))
```

## Filter Configuration

### Global Filter Configuration

```php
public function configureFilters(Filters $filters): Filters
{
    return $filters
        ->setMaxCount(10)                    // Limit number of filters shown
        ->add('name')
        ->add(BooleanFilter::new('isActive'));
}
```

### Individual Filter Options

```php
// Label
->add(BooleanFilter::new('isActive')
    ->setLabel('Active Status')
)

// Form type options
->add(EntityFilter::new('category')
    ->setFormTypeOptions([
        'value_type_options' => [
            'class' => Category::class,
            'multiple' => true,
            'attr' => ['data-widget' => 'select2'],
        ],
    ])
)

// Disable filter
->add(BooleanFilter::new('isActive')
    ->canSelectMultiple()  // Allow multiple values
)
```

## Advanced Patterns

### Custom Filter Logic

Create a custom filter by extending `EasyCorp\Bundle\EasyAdminBundle\Filter\FilterTrait`:

```php
use Doctrine\ORM\QueryBuilder;
use EasyCorp\Bundle\EasyAdminBundle\Contracts\Filter\FilterInterface;
use EasyCorp\Bundle\EasyAdminBundle\Dto\EntityDto;
use EasyCorp\Bundle\EasyAdminBundle\Dto\FieldDto;
use EasyCorp\Bundle\EasyAdminBundle\Dto\FilterDataDto;
use EasyCorp\Bundle\EasyAdminBundle\Filter\FilterTrait;
use Symfony\Component\Form\Extension\Core\Type\TextType;

class PriceRangeFilter implements FilterInterface
{
    use FilterTrait;

    public static function new(string $propertyName, $label = null): self
    {
        return (new self())
            ->setFilterFqcn(__CLASS__)
            ->setProperty($propertyName)
            ->setLabel($label)
            ->setFormType(TextType::class);
    }

    public function apply(
        QueryBuilder $queryBuilder,
        FilterDataDto $filterDataDto,
        ?FieldDto $fieldDto,
        EntityDto $entityDto
    ): void {
        $value = $filterDataDto->getValue();

        if (str_contains($value, '-')) {
            [$min, $max] = explode('-', $value);
            $queryBuilder
                ->andWhere(sprintf('%s.%s BETWEEN :min AND :max', $filterDataDto->getEntityAlias(), $filterDataDto->getProperty()))
                ->setParameter('min', $min)
                ->setParameter('max', $max);
        }
    }
}
```

Usage:
```php
->add(PriceRangeFilter::new('price')->setLabel('Price Range'))
```

### Conditional Filters

```php
public function configureFilters(Filters $filters): Filters
{
    $filters->add('name');

    // Only show certain filters for admins
    if ($this->isGranted('ROLE_ADMIN')) {
        $filters
            ->add(BooleanFilter::new('isDeleted'))
            ->add(EntityFilter::new('createdBy'));
    }

    return $filters;
}
```

### Multiple Entity Filters

```php
->add(EntityFilter::new('category'))
    ->setFormTypeOptions([
        'value_type_options' => [
            'multiple' => true,
            'query_builder' => function (CategoryRepository $repo) {
                return $repo->createQueryBuilder('c')
                    ->where('c.isActive = true')
                    ->orderBy('c.name', 'ASC');
            },
        ],
    ])
```

### Date Range Filters

```php
// Use DateTimeFilter for range filtering
->add(DateTimeFilter::new('createdAt'))
    ->setFormTypeOptions([
        'value_type_options' => [
            'widget' => 'single_text',
            'attr' => ['placeholder' => 'YYYY-MM-DD'],
        ],
    ])
```

### Optimized Autocomplete Filters

```php
->add(EntityFilter::new('user'))
    ->setFormTypeOptions([
        'value_type' => AutocompleteType::class,
        'value_type_options' => [
            'class' => User::class,
            'placeholder' => 'Select user...',
            'attr' => [
                'data-ea-autocomplete-endpoint-url' => '...',
            ],
        ],
    ])
```

## Filter Form Types

Filters can use any Symfony form type:

```php
use Symfony\Component\Form\Extension\Core\Type\*;

// Custom form type
->add(ChoiceFilter::new('status')
    ->setFormType(ChoiceType::class)
    ->setFormTypeOptions([
        'choices' => [
            'Active' => 1,
            'Inactive' => 0,
        ],
        'expanded' => true,   // Radio buttons
        'multiple' => false,
    ])
)

// Range filter with two inputs
->add(NumericFilter::new('price')
    ->setFormType(NumberType::class)
    ->setFormTypeOptions([
        'attr' => [
            'min' => 0,
            'max' => 10000,
            'step' => 0.01,
        ],
    ])
)
```

## Performance Considerations

1. **Index database columns** used in filters for better performance
2. **Limit filters** - Use `setMaxCount()` to avoid overwhelming users
3. **Autocomplete for large datasets** - Use autocomplete for entity filters with many records
4. **Avoid complex queries** - Keep filter query builders simple
5. **Cache filter queries** - Consider caching filter options that don't change often

## Filter Display

Filters appear in the sidebar by default. Customize with:

```php
public function configureCrud(Crud $crud): Crud
{
    return $crud
        ->showFiltersInFooter()         // Show filters in footer instead
        ->renderFiltersAsForm()         // Render as inline form
        ->setFiltersFormTheme('...')    // Custom form theme
}
```

## Common Patterns

### Status Badge Filters

```php
->add(ChoiceFilter::new('status')
    ->setChoices([
        'Draft' => 'draft',
        'Published' => 'published',
        'Archived' => 'archived',
    ])
)
```

### Date Shortcuts

```php
->add(DateTimeFilter::new('createdAt'))
    ->setFormTypeOptions([
        'value_type_options' => [
            'widget' => 'single_text',
            'html5' => true,
        ],
    ])
```

## Troubleshooting

- **Filter not appearing**: Ensure property exists and is mapped in Doctrine
- **Empty filter options**: Check EntityFilter query builder or entity repository
- **Performance issues**: Add database indexes on filtered columns
- **Multiple values not working**: Set `'multiple' => true` in form options
