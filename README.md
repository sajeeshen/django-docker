# Docker'd Django
### Django, Postgres, and Redis, all in Docker

This is a boilerplate repo intended for quickly starting a new **Django** project with **PostgreSQL** and **Redis** support, all running within Docker containers. A **Nginx** service is also defined to enable immediate access to the site over port 80.

## Prerequisites

- Docker
- Pipenv
  - Make sure Python3 is available
  - Enables `pipenv install` to set up libraries locally for the editor to crawl. The Django container also uses Pipenv to install dependencies to encourage use of this new Python package management tool.

## Getting started

1. Clone this repo
2. Delete the **.git** folder
    - `rm -rf .git/`
3. Create a new git repo
    - `git init`
    - `git add .`
    - `git commit -m "Initial Commit"`
4. Install Python dependencies in a Python3 virtual environment
    - `pipenv install --three`
5. Create a new Django project
    - `pipenv run django-admin startproject appname _app/`
6. Make the following changes to your Django project's **settings.py**:

```py
# appname/settings.py
import os

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.getenv('DJANGO_SECRET_KEY')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = os.getenv('DEBUG', False) == 'true'

ALLOWED_HOSTS = [
    'localhost',
]

INSTALLED_APPS = [
    # ...snip...
    'django_redis',
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': os.getenv('POSTGRES_USER'),
        'USER': os.getenv('POSTGRES_USER'),
        'PASSWORD': os.getenv('POSTGRES_PASSWORD'),
        'HOST': 'db',
        'PORT': 5432,
    }
}

STATIC_ROOT = 'static'

# django-redis
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://redis:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'
```

7. Update the **.env** file to specify values for the environment variables defined within
8. Start Django for development
    - `docker-compose up`

## Components

### Dockerfile

Builds the Django container. The container is built from a standard **python** Docker image and will run Django's `colletstatic` when being built.

### docker-compose.yml

Tasked with spinning up three containers: the above container for **Django**, one for **PostgreSQL**, and one for **Redis**.

By default an **Nginx** container is also created to reverse-proxy requests to Django and serve static files. In this configuration, the Django server will be available on port 80 during production.

If the Nginx container is removed, Docker can be accessed directly on port 8000. Static files can then be served from the **static_files_volume** Docker volume.

### docker-compose.override.yml

Loads automatically when running a standard `docker-compose up`. These values are intended for development purposes as they tweak the container configurations in the following way:

- Make the Postgres container accessible externally on port 5432
- Make the site available through Nginx at http://localhost:8000
- Start gunicorn to reload when it detects a file change
- Set an environmental flag telling Django that it is running in debug mode

### Pipfile/Pipfile.lock

Includes Python packages needed to make Django, Postgre, and Redis work together.

### .env

Contains environment variables for the containers. Several variables are included for configuring Postgres and Django secrets.

### .dockerignore

Defines files that Docker should _never_ include when building the Django image.

### .editorconfig

Defines some common settings to help ensure consistency of styling across files.

### .flake8

Configures the **flake8** Python linter. Includes a few common settings to my personal preferences.

### .vscode/settings.json

Helps configure the Python plugin to lint with flake8. A placeholder Python interpreter setting is left in to simplify pointing to the local virtual environment created with Pipenv.

### _app/gunicorn.cfg.py

Defines settings for gunicorn, including a port binding, workers, and a gunicorn-specific error log.

### _ngingx/nginx.conf

Establishes a reverse-proxy to Django, and serves Django static files.

### start-dev.sh

An executable script for starting the server in development mode.

### start-prod.sh

An executable script for starting the server in production mode.

### update-prod-django.sh

An executable script for rebuilding and restarting production Django whenever there's a change. This script also restarts Nginx to ensure no traffic hits the server while the update occurs.
