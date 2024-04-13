---
layout: default
title: Redis configuration
parent: Project configuration
grand_parent: Django
nav_order: 4
---

# Redis

Redis can be used in Django for caching, session storage, or as a task queue backend for [Celery](add-celery.md), so take in account on which feature of `Redis` you need when using this configuration.

## Adding Redis to a Django Project

### Step 1: Add redis to django project

First, you need to install `redis` and `django-redis` packages. `django-redis` is a full-featured `Redis` cache/session backend for Django.

```bash
pip install redis django-redis
```

Add `django-redis` to your apps:

```python
INSTALLED_APPS = [
    ...
    'django_redis',
]
```

### Step 2: Configure Caching

Add the following configuration to your Django settings file (`settings.py`) if you need redis for cache. This example sets up Redis as the caching backend.

```python
# settings.py

CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
```

In the above configuration:
- `"LOCATION"` points to the Redis server. The URL format is `redis://[:password]@hostname:port/db_number`.
- Change the `db_number` to avoid conflicts with other databases on the same Redis server.

### Step 3: Use Redis for Session Management (Optional)

If you need to use Redis for session storage, update the session engine in `settings.py`:

```python
# settings.py
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"
```

This configuration tells Django to store session data in the cache backend defined under the "default" alias.

## Installing and running Redis

### Without docker

Here are step-by-step instructions to install Redis on a Linux server, configure it to start automatically with the server, and apply basic security settings. This guide focuses on using a Debian/Ubuntu-based system, as these are among the most common for web servers.

### Step 1: Install Redis

First, update your package repository data and install Redis using `apt`:

```bash
sudo apt update
sudo apt install redis-server
```

or if you are using docker you can add the next configuration to the docker compose file:

```yaml
services:
  redis:
    image: redis
    command: redis-server --appendonly yes --requirepass yourverystrongpassword
    volumes:
        - ./data:/data
    ports:
        - "6379:6379"
    restart: unless-stopped
    env_file:
        - .env
```

## Using Redis cache

### Setting Cache Values

You can set cache values using either the default cache or a named cache. Here’s how you can do it:

```python
from django.core.cache import caches

# Using the default cache
default_cache = caches['default']  # This is the name that you use in settings
default_cache.set('key', 'value', timeout=300)

# Using a named cache
my_cache = caches['my_cache']
my_cache.set('another_key', 'another_value', timeout=300)
```

The set of a timeout is important in order to manage memory, this depends on your specific requirements, see the section [Memory usage in redis](#memory-usage-in-redis)

### Getting Cache Values

To retrieve values from the cache, you use the `get` method:

```python
# Using the default cache
value = default_cache.get('key')
```

### Invalidating Cache

Invalidating or clearing the cache can be done as follows:

```python
# Clear a specific key from the default cache
default_cache.delete('key')

# Clear all the data from the default cache
default_cache.clear()
```

We highly recommend to have cache keys in a separate file and import them to avoid typo on the caches key.

Redis is highly efficient at managing memory, which makes it popular for use as a caching solution. It provides several mechanisms for memory management and cache eviction, allowing it to handle memory allocation and data expiration gracefully. Here's an overview of how Redis uses memory and manages the deletion of cache items to free up space:

## Memory Usage in Redis

1. **Data Structures**: Redis supports various data structures like strings, hashes, lists, sets, and sorted sets. The memory usage depends on the type and size of these data structures. Redis is designed to be memory efficient with all its supported types.

2. **Memory Allocation**: Redis uses an allocator (by default, jemalloc in most installations) to manage memory more efficiently than the system's default allocator. This helps in reducing memory fragmentation.

3. **Transparent Huge Pages (THP)**: Redis works best with THP disabled, as it can cause increased memory usage and latency. It's recommended to turn off THP for production use of Redis.

### Managing Cache Deletion and Memory

Redis provides several strategies to handle memory limits and eviction policies, which define how keys are removed when memory is needed:

1. **Eviction Policies**: When the maximum memory limit set in Redis is reached, it follows an eviction policy to remove keys and make space for new data. The policies include:
   - **No eviction (`noeviction`)**: Returns errors when the memory limit is reached and the client tries to insert more data.
   - **All keys LRU (`allkeys-lru`)**: Evicts the least recently used (LRU) keys out of all keys.
   - **Volatile LRU (`volatile-lru`)**: Evicts the least recently used keys out of all keys with an "expire" set.
   - **All keys random (`allkeys-random`)**: Randomly evicts keys to make space.
   - **Volatile random (`volatile-random`)**: Randomly evicts keys among those that have an expiration set.
   - **Volatile TTL (`volatile-ttl`)**: Evicts keys with an expiration set and prefers keys with a shorter time to live (TTL).
   
   You can set the eviction policy in the Redis configuration file:

   ```conf
   maxmemory-policy allkeys-lru
   ```

   The `volatile-lru` is the default eviction policy for most databases, set this based on your needs.

2. **Max Memory Setting**: You can specify a maximum amount of memory for Redis to use before it begins evicting keys:

   ```conf
   maxmemory 2gb
   ```

   This setting is crucial for avoiding out-of-memory errors on the machine Redis is running on.

3. **Expiration of Keys**: Redis allows keys to be set with a TTL (time-to-live), after which they expire and are deleted automatically. This is useful for cache management, where stale data can be removed automatically without manual intervention.

4. **Manual Deletion**: You can also manually delete keys using commands like `DEL`, which is useful for managing cache entries that don't naturally expire but need to be refreshed periodically.

5. **Persistence**: If enabled, Redis supports snapshotting and append-only file (AOF) persistence that can help in recovering data after restarts. However, persistence settings can also impact memory usage and performance, and should be configured according to the use case.

By understanding and configuring these aspects of Redis, you can optimize its performance and memory usage effectively. This ensures that Redis operates efficiently within the constraints of your system's resources, maintaining high performance and availability.
