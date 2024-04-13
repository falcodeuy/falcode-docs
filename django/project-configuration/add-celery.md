---
layout: default
title: Celery configuration
parent: Project configuration
grand_parent: Django
nav_order: 3
---

# Celery

This document provides guidance on configuring Celery and Celery Beat, which are used for performing asynchronous and periodic tasks, respectively. It is important to note that Celery requires a message queue to manage asynchronous tasks and to capture their output. Suitable queue options include Redis, RabbitMQ, Amazon SQS, or any other preferred queue service. For Redis setup instructions, refer to our [`Redis documentation`](add-redis.md)

## Configuration

Configuration can be performed with or without Docker. The initial setup steps are common for both methods:

### Step1: Add Celery to your project

Activate the virtual environment for your project and install `Celery` using the following command:

```bash
pip install celery
```

In the directory containing your application's main files, such as `urls.py`, `wsgi.py`, etc., create a new file named `celery.py` with the following content:

```python
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'your_project_name.settings')

app = Celery('your_project_name')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()
```

Replace `your_project_name` with the name of your current project. If your main `settings.py` is in a subfolder, replace `your_project_name.settings` with `your_project_name.subfolder.settings`.

In the same directory, modify or create `__init__.py` to include the following:

```python
from config.celery import app as celery_app

__all__ = ['celery_app']
```

Next, add the broker configuration for `Celery` in your `settings.py`. Here is an example configuration using a locally running `Redis` server:

```python
# Celery Configuration
CELERY_BROKER_URL = 'redis://127.0.0.1:6379/0'
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/0'
CELERY_TIMEZONE = 'America/Santiago' # Optional
```

The result backend is optional and manages the responses of tasks; the essential setting is the `CELERY_BROKER_URL`.

### Step2 (With Docker)

Configuring `Celery` in Docker is straightforward. Simply add `Celery` and `Celery Beat` (if required) to your Docker Compose file:

```yaml
celery:
    restart: always
    build:
      context: .
      dockerfile: Dockerfile.staging
    command: celery -A backend worker -l info -c 4
    volumes:
      - .:/code
    env_file:
      - .env
    depends_on:
      - db
    networks:
      - backend_network

celery-beat:
    restart: always
    build:
      context: .
      dockerfile: Dockerfile.staging
    command: celery -A backend beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
    volumes:
      - .:/code
    env_file:
      - .env
    depends_on:
      - db
    networks:
      - backend_network
```

The `-c` parameter on `Celery` configuration specify the number of workers that will take your asynchronous tasks to run, adjust this parameter based on your hardware and requirements.

### Step2 (Without Docker)

When running without Docker, you need to have both the Celery worker and Celery Beat running as background processes. To start each, use the following commands:

For `Celery`:

```bash
celery -A project_name worker --loglevel=info
```

And for `Celery Beat`:

```bash
celery -A project_name beat --loglevel=info
```

Alternatively, for development purposes, you can run both the worker and Beat in the same process using the `--beat` option as shown below:

```bash
celery -A project_name worker --loglevel=info --beat
```

For a production environment, it is advisable to configure services that run in the background and start automatically with the server. To set up these services, starting with `Celery`. Create a service file:

```bash
sudo nano /etc/systemd/system/celery.service
```

Add the following content to the file:

```bash
[Unit]
Description=Celery Service
After=network.target

[Service]
Type=forking
User=your_user
Group=your_group
WorkingDirectory=/path/to/your/django/project
Environment="PATH=/path/to/your/virtualenv"
ExecStart=/path/to/your/virtualenv/bin/celery -A your_project_name worker --loglevel=info
ExecStop=/path/to/your/virtualenv/bin/celery -A your_project_name control shutdown
Restart=always

[Install]
WantedBy=multi-user.target
```

Replace all placeholders with your virtual environment paths and folder locations. Repeat this process for `Celery Beat` by creating a file named `celerybeat.service`. Use similar content but replace `worker` with `beat` in the `ExecStart` command.

Then, enable and start both services with the following commands:

```bash
sudo systemctl enable celery.service
sudo systemctl enable celerybeat.service
sudo systemctl start celery.service
sudo systemctl start celerybeat.service
```

## Using Celery

This section covers basic usage of `Celery` and `Celery Beat`. Typically, we create a `tasks.py` file inside the app that will use the task, though the function can be placed anywhere.

### Celery

To run asynchronous tasks with `Celery`, first define a task function with the `@shared_task` annotation:

```python
@shared_task
def foo(param1, param2):
    ...content of the function
```

This function can be called synchronously or asynchronously. Here’s an example of an asynchronous call:

```python
foo.delay(param1, param2)
```

The function can also be called in a synchronous manner:

```python
foo(param1, param2)
```

In the asynchronous case, it is important to handle connection errors to the broker (e.g., Redis). A good practice is to fallback to synchronous execution if an error occurs:

```python
try:
    foo.delay(param1, param2)
except BrokerConnectionError:
    foo(param1, param2)
```

### Celery Beat

For `Celery Beat`, task scheduling is defined in the settings. A task similar to the one above can be configured as follows:

```python
@shared_task
def foo(param1, param2):
    ...content of the function
```

We normally define this tasks in `tasks.py` but could be on any file. Then just add the configuration for your settings file, like this one:

```python
CELERY_BEAT_SCHEDULE = {
    'foo-example': {
        'task': 'bar.tasks.foo',
        'schedule': timedelta(minutes=10)
    }
}
```

The `foo-example` key can be any name that helps you identify the task. The task attribute should follow the format `app.file.function`. In our example, the function foo is located in an app named `bar` within a file called `tasks.py`, but it can be placed in any appropriate file or app.

The `timedelta(minutes=10)` is used to schedule the task to run every 10 minutes. The interval can be adjusted to days, seconds, or any other unit of time like `timedelta(days=2)`. Additionally, you have the option to specify exact hours for tasks that need to run at specific times, such as processes typically executed at night. For example:

```python
CELERY_BEAT_SCHEDULE = {
    'foo-example': {
        'task': 'bar.tasks.foo',
        'schedule': crontab(minute=0, hour='3,18')
    }
}
```

This configuration schedules the task to run at `3 AM` and `6 PM`. If you need to run tasks at specific hours, this setup is ideal. For more details, refer to the [official Celery documentation](https://docs.celeryq.dev/en/stable/userguide/index.html).
