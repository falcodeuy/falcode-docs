---
layout: default
title: Code style
parent: Django
has_children: true
nav_order: 3
---

# Code style guides

## Table of contents

{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The use of quotes

Use single quotes always, except when the string itself contains a single quote:

```python
sentence = "It's a beautiful day."
```

## Serializers (DRF)

- Keep serializers thin: validation, normalization, and input/output mapping.
- Serializers can include basic create/update behavior, but business logic and side effects belong in services.
- Validate all input (body and query params) with serializers before using it.
- For id-based relations, use typed relational fields (for example, `PrimaryKeyRelatedField`) instead of manual parsing.
- Business logic must consume `validated_data`; do not drive business decisions from raw `request.data`.

## Views / ViewSets

- Put all possible validations in serializers; in views, keep only validations that cannot be modeled directly in serializers.
- Keep a clear view structure:
    - Validation
    - Service call
    - Response
- Views orchestrate authentication/permissions, serializer invocation, and HTTP response.
- Thin-layer rule: business logic and side effects go to modules in the `services` folder.
- Prefer DRF `ViewSet` and generic views when applicable, avoiding unnecessary ad-hoc views.
- Keep responses consistent (stable API contract shape).

## Enums (Choices in Django)

- Use `IntegerChoices` or `TextChoices`.
- Use `ChoiceFieldWithDisplayName` in serializers.

## Admin

- Define `list_display` with relevant base attributes.
- Define `list_per_page` for performance.
- Define `search_fields` to improve admin search.
- Use `unfold.admin.ModelAdmin` as the default admin base in all projects.
- Use `autocomplete_fields` when a relation (`ForeignKey`/`ManyToMany`) points to another model with many records.
- Use `AutocompleteSelectFilter` in `list_filter` when filtering by relations to other models with many options.

### Using Unfold and a shared BaseAdmin

- **Unfold first**: use `ModelAdmin` from `unfold.admin` instead of Django's default `admin.ModelAdmin`.
- **Shared behavior in `BaseAdmin`**: centralize general behavior like page size or default ordering.

Example base class for shared behavior:

```python
from import_export.admin import ExportActionMixin
from unfold.admin import ModelAdmin


class BaseAdmin(ExportActionMixin, ModelAdmin):
    list_per_page = 10

    def get_ordering(self, request):
        # Respect explicit ordering declared in each admin class.
        class_ordering = self.__class__.__dict__.get('ordering')
        if class_ordering:
            return class_ordering

        model_field_names = {field.name for field in self.model._meta.get_fields()}
        if 'created_at' in model_field_names:
            return ('-created_at',)

        # Fallback: newest rows first by primary key for models without created_at.
        return (f'-{self.model._meta.pk.name}',)
```

Use this `BaseAdmin` as the parent class for model-specific admins.

### Relations to other models (Admin)

- **Use `autocomplete_fields` for related fields**: improves performance and usability when selecting objects from large tables.
- **Use `AutocompleteSelectFilter` for related filters**: keeps list filters responsive for `ForeignKey` relations with many options.
- **Remember required `search_fields`**: the related model admin must define `search_fields` so autocomplete queries can work.

```python
from django.contrib import admin
from admin_auto_filters.filters import AutocompleteSelectFilter

from .models import Invoice, Customer
from .base_admin import BaseAdmin


@admin.register(Customer)
class CustomerAdmin(BaseAdmin):
    search_fields = ['name', 'email']


class CustomerAutocompleteFilter(AutocompleteSelectFilter):
    title = 'Customer'
    field_name = 'customer'


@admin.register(Invoice)
class InvoiceAdmin(BaseAdmin):
    list_display = ['number', 'customer', 'status']
    autocomplete_fields = ['customer']
    list_filter = [CustomerAutocompleteFilter, 'status']
```
