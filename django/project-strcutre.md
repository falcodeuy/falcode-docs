---
layout: default
title: Projects structure
parent: Django
nav_order: 1
permalink: /django/project-structure
---

# Project structure

## Files and folder structure

Out projects structure has this key points:

- `manage.py` is in the main folder (`project_name`), which serves as the entry point to various Django management commands.
  
- Individual applications reside inside the `apps` folder. Each application (like `app1`, `app2`, etc.) will have its standard Django app structure with `models.py`, `views.py`, `admin.py`, etc.
  
- The `backend` folder contains the root `urls.py` and `settings.py`, along with other main configurations like `wsgi.py`.

```
project_name/
├── manage.py
├── apps/
│ ├── app1/
│ │ ├── migrations/
│ │ ├── static/
│ │ ├── __init__.py
| | ├── admin.py
│ │ ├── apps.py
│ │ ├── filters.py
│ │ ├── models.py
│ │ ├── serializers.py
│ │ ├── tests.py
│ │ ├── urls.py
│ │ ├── utils.py
│ │ └── views.py
│ ├── app2/
│ │ ├── ...
│ └── ...
├── backend/
| ├── init.py
| ├── settings.py
| ├── urls.py
| ├── wsgi.py
└── ...
```

Create .env file to read any variable that depends on the working environment or need to be hidden.

## Settings file

- Use DJANGO_ENV variable to define different logic between local working environment, example for images configuration:

```python
# Images storage configuration
if DJANGO_ENV == 'production' or DJANGO_ENV == 'staging':
    # Use Amazon S3 for storing media files
    AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
    AWS_STORAGE_BUCKET_NAME = os.environ.get('AWS_STORAGE_BUCKET_NAME')
    AWS_S3_REGION_NAME = os.environ.get('AWS_S3_REGION_NAME')
    AWS_S3_CUSTOM_DOMAIN = '%s.s3.amazonaws.com' % AWS_STORAGE_BUCKET_NAME
    AWS_S3_OBJECT_PARAMETERS = {
        'CacheControl': 'max-age=86400',
    }
    DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
    MEDIA_URL = 'https://%s/' % AWS_S3_CUSTOM_DOMAIN

else:
    # Local development settings
    MEDIA_URL = '/media/'
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media/')

```



