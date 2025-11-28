# 01-todo — Django TODO application

This document contains a complete, minimal Django TODO app you can copy into a repository folder (e.g. `01-todo`). It includes project-level files, app files, templates, tests, and instructions to run and test the project.

---

## Quick setup instructions

1. Create a Python virtual environment and activate it (example using `venv`):

```bash
python -m venv .venv
source .venv/bin/activate  # macOS / Linux
.\.venv\Scripts\activate   # Windows (PowerShell)
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Make migrations and migrate:

```bash
python manage.py makemigrations
python manage.py migrate
```

4. (Optional) Create a superuser to access admin:

```bash
python manage.py createsuperuser
```

5. Run the test suite:

```bash
python manage.py test
```

6. Run the development server:

```bash
python manage.py runserver
```

Open http://127.0.0.1:8000/ to see the TODO app.

---

## Project structure

```
01-todo/
├── README.md
├── requirements.txt
├── manage.py
├── todo_project/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── todos/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── forms.py
    ├── models.py
    ├── tests.py
    ├── urls.py
    ├── views.py
    └── templates/
        └── todos/
            ├── base.html
            ├── home.html
            └── todo_form.html
```

---

## requirements.txt

```
Django>=4.2
pytest-django
```

---

## manage.py

```python
#!/usr/bin/env python
import os
import sys

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "todo_project.settings")
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and available on your PYTHONPATH?"
        ) from exc
    execute_from_command_line(sys.argv)
```

---

## todo_project/settings.py

```python
from pathlib import Path
import os

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = "django-insecure-change-me-for-prod"

DEBUG = True

ALLOWED_HOSTS = []

INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "todos",
]

MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
]

ROOT_URLCONF = "todo_project.urls"

TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [BASE_DIR / "templates"],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]

WSGI_APPLICATION = "todo_project.wsgi.application"

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}

AUTH_PASSWORD_VALIDATORS = []

LANGUAGE_CODE = "en-us"
TIME_ZONE = "UTC"
USE_I18N = True
USE_TZ = True

STATIC_URL = "/static/"
DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"
```

---

## todo_project/urls.py

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("todos.urls")),
]
```

---

## todo_project/wsgi.py

```python
import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "todo_project.settings")
application = get_wsgi_application()
```

---

## todos/apps.py

```python
from django.apps import AppConfig


class TodosConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name = "todos"
```

---

## todos/models.py

```python
from django.db import models


class Todo(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    due_date = models.DateField(null=True, blank=True)
    resolved = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

---

## todos/admin.py

```python
from django.contrib import admin
from .models import Todo


@admin.register(Todo)
class TodoAdmin(admin.ModelAdmin):
    list_display = ("id", "title", "due_date", "resolved", "created_at")
    list_filter = ("resolved",)
    search_fields = ("title", "description")
```

---

## todos/forms.py

```python
from django import forms
from .models import Todo


class TodoForm(forms.ModelForm):
    class Meta:
        model = Todo
        fields = ["title", "description", "due_date", "resolved"]
        widgets = {
            "due_date": forms.DateInput(attrs={"type": "date"}),
        }
```

---

## todos/urls.py

```python
from django.urls import path
from . import views

app_name = "todos"

urlpatterns = [
    path("", views.home, name="home"),
    path("create/", views.todo_create, name="create"),
    path("<int:pk>/edit/", views.todo_edit, name="edit"),
    path("<int:pk>/delete/", views.todo_delete, name="delete"),
    path("<int:pk>/toggle/", views.todo_toggle, name="toggle"),
]
```

---

## todos/views.py

```python
from django.shortcuts import render, get_object_or_404, redirect
from .models import Todo
from .forms import TodoForm


def home(request):
    todos = Todo.objects.order_by("-created_at")
    return render(request, "todos/home.html", {"todos": todos})


def todo_create(request):
    if request.method == "POST":
        form = TodoForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("todos:home")
    else:
        form = TodoForm()
    return render(request, "todos/todo_form.html", {"form": form})


def todo_edit(request, pk):
    todo = get_object_or_404(Todo, pk=pk)
    if request.method == "POST":
        form = TodoForm(request.POST, instance=todo)
        if form.is_valid():
            form.save()
            return redirect("todos:home")
    else:
        form = TodoForm(instance=todo)
    return render(request, "todos/todo_form.html", {"form": form, "todo": todo})


def todo_delete(request, pk):
    todo = get_object_or_404(Todo, pk=pk)
    if request.method == "POST":
        todo.delete()
        return redirect("todos:home")
    return render(request, "todos/todo_form.html", {"form": None, "todo": todo, "confirm_delete": True})


def todo_toggle(request, pk):
    todo = get_object_or_404(Todo, pk=pk)
    todo.resolved = not todo.resolved
    todo.save()
    return redirect("todos:home")
```

---

## todos/templates/todos/base.html

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>TODO App</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/water.css@2/out/water.css">
  </head>
  <body>
    <header>
      <h1><a href="/">TODOs</a></h1>
    </header>
    <main>
      {% block content %}{% endblock %}
    </main>
  </body>
</html>
```

---

## todos/templates/todos/home.html

```html
{% extends "todos/base.html" %}

{% block content %}
  <a href="{% url 'todos:create' %}">Create TODO</a>
  <ul>
    {% for todo in todos %}
      <li>
        <strong>{{ todo.title }}</strong>
        {% if todo.due_date %} — due {{ todo.due_date }}{% endif %}
        {% if todo.resolved %} ✅{% endif %}
        <div>
          <a href="{% url 'todos:edit' todo.pk %}">Edit</a>
          <form action="{% url 'todos:delete' todo.pk %}" method="post" style="display:inline">
            {% csrf_token %}
            <button type="submit">Delete</button>
          </form>
          <a href="{% url 'todos:toggle' todo.pk %}">{% if todo.resolved %}Mark unresolved{% else %}Mark resolved{% endif %}</a>
        </div>
        <p>{{ todo.description }}</p>
      </li>
    {% empty %}
      <li>No TODOs yet.</li>
    {% endfor %}
  </ul>
{% endblock %}
```

---

## todos/templates/todos/todo_form.html

```html
{% extends "todos/base.html" %}

{% block content %}
  {% if confirm_delete %}
    <h2>Delete TODO: {{ todo.title }}</h2>
    <p>Are you sure?</p>
    <form method="post">
      {% csrf_token %}
      <button type="submit">Yes, delete</button>
      <a href="{% url 'todos:home' %}">Cancel</a>
    </form>
  {% else %}
    <h2>{% if todo %}Edit{% else %}Create{% endif %} TODO</h2>
    <form method="post">
      {% csrf_token %}
      {{ form.as_p }}
      <button type="submit">Save</button>
      <a href="{% url 'todos:home' %}">Cancel</a>
    </form>
  {% endif %}
{% endblock %}
```

---

## todos/tests.py

```python
from django.test import TestCase
from django.urls import reverse
from .models import Todo
from datetime import date


class TodoTests(TestCase):
    def test_create_todo(self):
        resp = self.client.post(reverse('todos:create'), data={
            'title': 'Test todo',
            'description': 'testing',
            'due_date': date.today(),
            'resolved': False,
        })
        self.assertEqual(resp.status_code, 302)  # redirect
        self.assertEqual(Todo.objects.count(), 1)

    def test_edit_todo(self):
        t = Todo.objects.create(title='Old', description='x')
        resp = self.client.post(reverse('todos:edit', args=[t.pk]), data={
            'title': 'New',
            'description': 'y',
            'resolved': True,
        })
        self.assertEqual(resp.status_code, 302)
        t.refresh_from_db()
        self.assertEqual(t.title, 'New')
        self.assertTrue(t.resolved)

    def test_delete_todo(self):
        t = Todo.objects.create(title='ToDelete')
        resp = self.client.post(reverse('todos:delete', args=[t.pk]))
        self.assertEqual(resp.status_code, 302)
        self.assertEqual(Todo.objects.count(), 0)

    def test_toggle_todo(self):
        t = Todo.objects.create(title='Toggle', resolved=False)
        resp = self.client.get(reverse('todos:toggle', args=[t.pk]))
        self.assertEqual(resp.status_code, 302)
        t.refresh_from_db()
        self.assertTrue(t.resolved)
```

---

## README.md

```
Django TODO app minimal example.

Usage:
- python -m venv .venv
- source .venv/bin/activate
- pip install -r requirements.txt
- python manage.py makemigrations && python manage.py migrate
- python manage.py runserver
```

---

That's the full scaffold. Paste each file into the corresponding path and run the commands from the Quick setup.

---

*If you want, I can also produce a zip/tar of these files, or create the same scaffold as a single `startproject`-style tree with exact file contents for copying. Tell me which you prefer.*

