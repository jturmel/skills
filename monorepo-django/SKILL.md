---
name: setup-monorepo-django
description: New project/greenfield setup for monorepo with Django
---

## What I do

- Help setup a new monorepo in greenfield/new project situations
- Define setup in the base of the repo
- Define where and how Django is configured

## Root of repo

- Setup generic `.gitignore`
- Setup LICENSE, user should choose from common open source licenses,
show options (MIT, GPLv3, other), or skip if proprietary.
- Setup a base `docker-compose.yaml` for Python and context of `/backend`

## Django setup

- Install latest release of Django
- Install into `/backend` directory
- Setup a `core` Django app
  - The `core` app is where we will keep all of the models for the entire system
- If there is a web interface, setup a `web` Django app
- Setup a `/backend/root` directory, inside it will be the following:
  - The wsgi.py/asgi.py entrypoints will go in this root directory.
  - Replace the default setup of `settings.py` with a `settings` module that has a `base.py` in it that then the other
configurations will import and can override specific settings as needed. Then when Django is started, the specific
configuration file (e.g. `dev.py`) is what is set as the DJANGO_SETTINGS_MODULE
- Python to be managed by `uv`

## CI/CD

- Setup ruff as the project linter
- If in Github, setup a lint and test workflow for Djanog/Python
linting and testing when opening PRs and merging to the default branch.
- Setup baseline of testing using pytest and django-pytest

## README

- Establish a baseline with information on how to get started (ie `docker compose up`)

