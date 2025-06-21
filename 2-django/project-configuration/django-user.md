---
layout: default
title: User configuration
parent: Project configuration
grand_parent: Django
nav_order: 1
---

# User configuration

## Choosing the Correct User Model in Django

We highly recommend overriding the `User` model at the start of the project because it becomes more difficult to do later, and in most cases, you will need to add features to the default user. Django provides two base classes that you can use to create a custom user model:

- **`AbstractUser`**: Extends Django's base `User` model with all the default fields and behaviors.
- **`AbstractBaseUser`**: Provides the core implementation of a user model, including password hashing and token generation, but does not include any actual fields.

### Considerations for Choosing a User Model

The choice between `AbstractUser` and `AbstractBaseUser` depends primarily on your application's specific requirements:

- **Use `AbstractUser` if**: You are satisfied with the existing fields on the Django user model but want to add some extra fields.
- **Use `AbstractBaseUser` if**: You need to build a completely customized user model from scratch, for instance, when you want to use an email address or any other field as the primary identifier instead of a username.

### Configuring `AbstractUser`

If you decide to extend `AbstractUser`, you can add additional fields and change some of the default behaviors. Here’s how to do it:

1. **Define the Custom User Model**

   Create a new model that inherits from `AbstractUser` and add your custom fields:

   ```python
   from django.contrib.auth.models import AbstractUser
   from django.db import models

   class CustomUser(AbstractUser):
       bio = models.TextField(blank=True)
   ```

2. **Update `settings.py`**

   Tell Django to use your custom user model by adding the following line to your `settings.py`:

   ```python
   AUTH_USER_MODEL = 'your_app.CustomUser'
   ```

3. **Create and Apply Migrations**

   Generate migrations for your new user model and apply them:

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

### Configuring `AbstractBaseUser`

Using `AbstractBaseUser` requires more setup because you need to define more of the model yourself, including the fields and the manager.

Start creating your model and selecting your primery key and required information for the user, then you can override the `BaseUserManager` 

1. **Define the Custom User Model**

   Create a model that inherits from `AbstractBaseUser`. You’ll also need to inherit from `PermissionsMixin` if you want to use Django’s permission framework:

   ```python
   from django.contrib.auth.models import AbstractBaseUser, PermissionsMixin, BaseUserManager
   from django.db import models

   class CustomUserManager(BaseUserManager):
       def create_user(self, email, password=None, **extra_fields):
           if not email:
               raise ValueError('The Email must be set')
           email = self.normalize_email(email)
           user = self.model(email=email, **extra_fields)
           user.set_password(password)
           user.save(using=self._db)
           return user

       def create_superuser(self, email, password, **extra_fields):
           extra_fields.setdefault('is_staff', True)
           extra_fields.setdefault('is_superuser', True)

           if extra_fields.get('is_staff') is not True:
               raise ValueError('Superuser must have is_staff=True.')
           if extra_fields.get('is_superuser') is not True:
               raise ValueError('Superuser must have is_superuser=True.')

           return self.create_user(email, password, **extra_fields)

   class CustomUser(AbstractBaseUser, PermissionsMixin):
       email = models.EmailField(unique=True)
       is_staff = models.BooleanField(default=False)
       is_active = models.BooleanField(default=True)

       USERNAME_FIELD = 'email'
       REQUIRED_FIELDS = []

       objects = CustomUserManager()

       def __str__(self):
           return self.email
   ```

2. **Update `settings.py`**

   Similarly to `AbstractUser`, you must inform Django about your custom model:

   ```python
   AUTH_USER_MODEL = 'your_app.CustomUser'
   ```

3. **Create and Apply Migrations**

   As before, create and apply migrations:

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```
