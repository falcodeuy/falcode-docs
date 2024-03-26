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

- Forzar poner nombre en los fields
- Poner campos de help_text
- Definir verbose name y verbose name plural
- Definir __str__

- Usar la clase custom de admin
