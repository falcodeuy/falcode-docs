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

Use single quotes always except when the string itself has a single quote, for example:

```python
sentence = "It's a beautiful day."
```

## Serializers

- Serializers only have validation or basic creation and update logic. If a model need complex logic on any request make 
- Always use serializers for request body (Avoid read and validate manually)
- User `PrimaryKeyRelatedField` for id validation of nested models

## Views

- Estrcutura:
    - Lectura con serializer
    - Logica de negocio (encapsulada)
    - Respuesta con serializer (o custom)
- Always use views from rest framework `viewsets` when is possible, if not, use `generics` views from rest framework and override methods.

## Enums (Choices in Django)

- Usar IntegerChoice o TextChoices
- En el serializer usar ChoiceFieldWithDisplayName

## Admin

* Always specify display name in order to have the basic attributes
* Always specify list_per_page for performance
* Add search for admin 

### Using a Custom Admin Class

- **Custom Admin Class**: Extend the default admin interface using a custom admin class. This allows you to customize admin panels, control which fields are displayed, customize how lists are filtered, apply ordering, group fields logically, and much more. Django’s modularity allows you to override or add to the configuration of model data as it’s presented in the admin interface, enhancing both usability and functionality.

Example Django model showing these implementations:

```python
from django.db import models
from django.contrib import admin

class MyModel(models.Model):
    name = models.CharField(max_length=100, help_text="Enter the full name.")
    description = models.TextField(help_text="Enter a detailed description.")

    class Meta:
        verbose_name = "My Model"
        verbose_name_plural = "My Models"

    def __str__(self):
        return self.name

class MyModelAdmin(admin.ModelAdmin):
    list_display = ['name', 'description']
    search_fields = ['name']

# Register your models and admin class with the admin site
admin.site.register(MyModel, MyModelAdmin)
```

This example illustrates how to make a Django model and its corresponding admin customization for enhanced interaction through the Django admin site.
