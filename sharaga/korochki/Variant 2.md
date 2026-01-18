Ниже **полный обновлённый гайд + весь код** (только Django admin, без кастомной админ‑панели).

---

# ✅ ПОЛНЫЙ ГАЙД (Django + MySQL + v2.sql)

## 1) Создай проект и приложение
```bash
django-admin startproject helpdesk
cd helpdesk
python manage.py startapp app
```

---

## 2) Ус��анови зависимости
```bash
pip install django mysqlclient
```

---

## 3) Создай БД и импортируй v2.sql
1. Создай базу `v2`
2. Импортируй `v2.sql` через phpMyAdmin

---

## 4) Вставь файлы (ниже)

---

## 5) Миграции Django
```bash
python manage.py migrate
```

---

## 6) Создай суперпол��зователя
```bash
python manage.py createsuperuser
```

---

## 7) Запуск
```bash
python manage.py runserver
```

---

## 8) Адреса
- сайт: http://127.0.0.1:8000/
- админка Django: http://127.0.0.1:8000/admin/

---

# ✅ ФАЙЛЫ

## `helpdesk/settings.py`
```python name=settings.py
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = 'change-this'
DEBUG = True
ALLOWED_HOSTS = []

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'helpdesk.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'helpdesk.wsgi.application'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'v2',
        'USER': 'root',
        'PASSWORD': '',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    }
}

LANGUAGE_CODE = 'ru'
TIME_ZONE = 'Europe/Moscow'
USE_I18N = True
USE_TZ = True

STATIC_URL = 'static/'
STATICFILES_DIRS = [BASE_DIR / 'static']

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

---

## `helpdesk/urls.py`
```python name=urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('', include('app.urls')),
    path('admin/', admin.site.urls),
]
```

---

## `app/models.py`
```python name=models.py
from django.db import models

class Category(models.Model):
    name = models.CharField(max_length=255)

    class Meta:
        db_table = 'category'
        managed = False

class Department(models.Model):
    code = models.CharField(max_length=255)
    name = models.CharField(max_length=255)

    class Meta:
        db_table = 'department'
        managed = False

class Role(models.Model):
    code = models.CharField(max_length=255)
    name = models.CharField(max_length=255)

    class Meta:
        db_table = 'role'
        managed = False

class Status(models.Model):
    code = models.CharField(max_length=255)
    name = models.CharField(max_length=255)

    class Meta:
        db_table = 'status'
        managed = False

class User(models.Model):
    id_role = models.ForeignKey(Role, on_delete=models.CASCADE, db_column='id_role')
    id_department = models.ForeignKey(Department, on_delete=models.CASCADE, db_column='id_department')
    password = models.CharField(max_length=255)
    full_name = models.CharField(max_length=255)
    phone = models.CharField(max_length=255)
    email = models.EmailField(max_length=100)

    class Meta:
        db_table = 'user'
        managed = False

class Task(models.Model):
    id_user = models.ForeignKey(User, on_delete=models.CASCADE, db_column='id_user')
    id_category = models.ForeignKey(Category, on_delete=models.CASCADE, db_column='id_category')
    id_status = models.ForeignKey(Status, on_delete=models.CASCADE, db_column='id_status')
    description = models.TextField()

    class Meta:
        db_table = 'task'
        managed = False
```

---

## `app/forms.py`
```python name=forms.py
from django import forms
from .models import User, Category

class RegisterForm(forms.Form):
    login = forms.CharField(min_length=6)
    password = forms.CharField(min_length=8, widget=forms.PasswordInput)
    full_name = forms.CharField()
    phone = forms.CharField()
    email = forms.EmailField()

    def clean_login(self):
        login = self.cleaned_data['login']
        if User.objects.filter(email=login).exists():
            raise forms.ValidationError("Логин уже занят.")
        return login

class LoginForm(forms.Form):
    login = forms.CharField()
    password = forms.CharField(widget=forms.PasswordInput)

class TaskForm(forms.Form):
    category = forms.ModelChoiceField(queryset=Category.objects.all())
    description = forms.CharField(widget=forms.Textarea)
    priority = forms.ChoiceField(choices=[('low','Низкий'),('mid','Средний'),('high','Высокий')])
```

---

## `app/views.py`
```python name=views.py
from django.shortcuts import render, redirect
from .forms import RegisterForm, LoginForm, TaskForm
from .models import User, Task, Status, Role, Department

def index(request):
    return render(request, 'index.html')

def register_view(request):
    form = RegisterForm(request.POST or None)
    if request.method == 'POST' and form.is_valid():
        role = Role.objects.get(id=1)
        dept = Department.objects.get(id=1)
        User.objects.create(
            id_role=role,
            id_department=dept,
            password=form.cleaned_data['password'],
            full_name=form.cleaned_data['full_name'],
            phone=form.cleaned_data['phone'],
            email=form.cleaned_data['login'],
        )
        return redirect('login')
    return render(request, 'register.html', {'form': form})

def login_view(request):
    form = LoginForm(request.POST or None)
    error = ''
    if request.method == 'POST' and form.is_valid():
        login = form.cleaned_data['login']
        password = form.cleaned_data['password']
        try:
            user = User.objects.get(email=login, password=password)
            request.session['user_id'] = user.id
            return redirect('tasks')
        except User.DoesNotExist:
            error = 'Неверный логин или пароль'
    return render(request, 'login.html', {'form': form, 'error': error})

def tasks_view(request):
    user_id = request.session.get('user_id')
    if not user_id:
        return redirect('login')
    tasks = Task.objects.filter(id_user=user_id)
    return render(request, 'tasks.html', {'tasks': tasks})

def task_create(request):
    user_id = request.session.get('user_id')
    if not user_id:
        return redirect('login')
    form = TaskForm(request.POST or None)
    if request.method == 'POST' and form.is_valid():
        status = Status.objects.get(id=1)
        Task.objects.create(
            id_user_id=user_id,
            id_category=form.cleaned_data['category'],
            id_status=status,
            description=form.cleaned_data['description']
        )
        return redirect('tasks')
    return render(request, 'task_create.html', {'form': form})
```

---

## `app/urls.py`
```python name=urls.py
from django.urls import path
from .views import index, register_view, login_view, tasks_view, task_create

urlpatterns = [
    path('', index, name='index'),
    path('register/', register_view, name='register'),
    path('login/', login_view, name='login'),
    path('tasks/', tasks_view, name='tasks'),
    path('tasks/create/', task_create, name='task_create'),
]
```

---

## `app/admin.py`
```python name=admin.py
from django.contrib import admin
from .models import User, Task, Category, Department, Role, Status

admin.site.register(User)
admin.site.register(Task)
admin.site.register(Category)
admin.site.register(Department)
admin.site.register(Role)
admin.site.register(Status)
```

---

# ✅ ШАБЛОНЫ

Создай папку `templates` в корне.

---

## `templates/base.html`
````html name=base.html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>HelpDesk</title>
    <style>
        :root {
            --bg: #f5f7fb;
            --card: #ffffff;
            --text: #1f2937;
            --muted: #6b7280;
            --primary: #2563eb;
            --border: #e5e7eb;
        }
        * { box-sizing: border-box; }
        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background: var(--bg);
            color: var(--text);
        }
        nav {
            background: var(--card);
            border-bottom: 1px solid var(--border);
            padding: 12px 20px;
        }
        nav a {
            color: var(--primary);
            text-decoration: none;
            margin-right: 12px;
            font-weight: 600;
        }
        .container {
            max-width: 900px;
            margin: 24px auto;
            background: var(--card);
            padding: 24px;
            border: 1px solid var(--border);
            border-radius: 12px;
            box-shadow: 0 6px 16px rgba(0,0,0,0.05);
        }
        h2 { margin-top: 0; }
        form {
            display: grid;
            gap: 12px;
        }
        input, select, textarea {
            width: 100%;
            padding: 10px 12px;
            border: 1px solid var(--border);
            border-radius: 8px;
            font-size: 14px;
        }
        button {
            background: var(--primary);
            color: #fff;
            border: none;
            padding: 10px 16px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 600;
        }
        ul { padding-left: 18px; }
        .muted { color: var(--muted); }
        .error { color: #dc2626; }
        a { color: var(--primary); }
    </style>
</head>
<body>
    <nav>
        <a href="/">Главная</a>
        <a href="/register/">Регистрация</a>
        <a href="/login/">Авторизация</a>
        <a href="/tasks/">Мои обращения</a>
        <a href="/tasks/create/">Создать</a>
    </nav>

    <div class="container">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
````

---

## `templates/index.html`
````html name=index.html
{% extends 'base.html' %}
{% block content %}
<h2>HelpDesk</h2>
<p class="muted">Добро пожаловать.</p>
{% endblock %}
````

---

## `templates/register.html`
````html name=register.html
{% extends 'base.html' %}
{% block content %}
<h2>Регистрация</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Зарегистрироваться</button>
</form>
<p><a href="/login/">Уже есть аккаунт? Войти</a></p>
{% endblock %}
````

---

## `templates/login.html`
````html name=login.html
{% extends 'base.html' %}
{% block content %}
<h2>Авторизация</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Войти</button>
</form>
<p class="error">{{ error }}</p>
<p><a href="/register/">Еще не зарегистрированы? Регистрация</a></p>
{% endblock %}
````

---

## `templates/tasks.html`
````html name=tasks.html
{% extends 'base.html' %}
{% block content %}
<h2>Мои обращения</h2>
<ul>
    {% for t in tasks %}
        <li>{{ t.id }} - {{ t.description }} ({{ t.id_status.name }})</li>
    {% empty %}
        <li class="muted">Нет обращений</li>
    {% endfor %}
</ul>
{% endblock %}
````

---

## `templates/task_create.html`
````html name=task_create.html
{% extends 'base.html' %}
{% block content %}
<h2>Создать обращение</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Отправить</button>
</form>
{% endblock %}
````

---

Если хочешь — добавлю строгую валидацию по ТЗ (логин/телефон/ФИО), слайдер и адаптивные шаблоны.