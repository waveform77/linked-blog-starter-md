Ниже — **новый гайд с нуля**, с минимальным кодом, SQLite, **стоковой админкой Django**, **очень простым CSS** и **слайдером на `<img>`**, как вы просили.  

---

# ✅ Пошаговый гайд с нуля

## 1) Установка
```bash
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install Django
```

## 2) Создание проекта
```bash
django-admin startproject avoska
cd avoska
python manage.py startapp shop
```

## 3) Подключаем приложение
Откройте `avoska/settings.py` и добавьте `shop` в `INSTALLED_APPS`, также настройте SQLite и кастомного пользователя (см. файл ниже).

---

# ✅ Структура проекта
```
avoska/
├─ manage.py
├─ requirements.txt
├─ avoska/
│  ├─ settings.py
│  ├─ urls.py
│  ├─ wsgi.py
│  └─ asgi.py
└─ shop/
   ├─ admin.py
   ├─ apps.py
   ├─ forms.py
   ├─ models.py
   ├─ urls.py
   ├─ views.py
   ├─ templates/shop/
   │  ├─ base.html
   │  ├─ register.html
   │  ├─ login.html
   │  ├─ catalog.html
   │  ├─ order_create.html
   │  └─ orders.html
   └─ static/shop/
      ├─ css/styles.css
      ├─ js/slider.js
      └─ img/slide1.jpg
      └─ img/slide2.jpg
      └─ img/slide3.jpg
      └─ img/slide4.jpg
```

---

# ✅ ФАЙЛЫ (вставьте как есть)

## requirements.txt
```txt name=requirements.txt
Django==5.1.2
```

---

## manage.py
```python name=manage.py
#!/usr/bin/env python
import os
import sys

def main():
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "avoska.settings")
    from django.core.management import execute_from_command_line
    execute_from_command_line(sys.argv)

if __name__ == "__main__":
    main()
```

---

## avoska/settings.py
```python name=avoska/settings.py
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = "django-insecure-avoska"
DEBUG = True
ALLOWED_HOSTS = []

INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "shop",
]

MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
]

ROOT_URLCONF = "avoska.urls"

TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]

WSGI_APPLICATION = "avoska.wsgi.application"

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}

AUTH_PASSWORD_VALIDATORS = []

LANGUAGE_CODE = "ru-ru"
TIME_ZONE = "Europe/Moscow"
USE_I18N = True
USE_TZ = True

STATIC_URL = "static/"

AUTH_USER_MODEL = "shop.User"

DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"
```

---

## avoska/urls.py
```python name=avoska/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("shop.urls")),
]
```

---

## avoska/wsgi.py
```python name=avoska/wsgi.py
import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "avoska.settings")
application = get_wsgi_application()
```

---

## avoska/asgi.py
```python name=avoska/asgi.py
import os
from django.core.asgi import get_asgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "avoska.settings")
application = get_asgi_application()
```

---

# shop приложение

## shop/apps.py
```python name=shop/apps.py
from django.apps import AppConfig

class ShopConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name = "shop"
```

---

## shop/models.py
```python name=shop/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    full_name = models.CharField("ФИО", max_length=150)
    phone = models.CharField("Телефон", max_length=20)

class Product(models.Model):
    name = models.CharField("Наименование", max_length=100)
    description = models.CharField("Описание", max_length=200)
    price = models.DecimalField("Стоимость", max_digits=10, decimal_places=2)

    def __str__(self):
        return self.name

class Order(models.Model):
    STATUS_NEW = "new"
    STATUS_CONFIRMED = "confirmed"
    STATUS_CANCELED = "canceled"

    STATUS_CHOICES = [
        (STATUS_NEW, "Новое"),
        (STATUS_CONFIRMED, "Подтверждено"),
        (STATUS_CANCELED, "Отменено"),
    ]

    user = models.ForeignKey(User, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField("Количество")
    address = models.CharField("Адрес доставки", max_length=255)
    status = models.CharField("Статус", max_length=20, choices=STATUS_CHOICES, default=STATUS_NEW)

    def __str__(self):
        return f"{self.user.full_name} — {self.product.name}"
```

---

## shop/forms.py
```python name=shop/forms.py
from django import forms
from django.contrib.auth.forms import AuthenticationForm
from django.contrib.auth import get_user_model
from .models import Order

User = get_user_model()

class RegisterForm(forms.ModelForm):
    password = forms.CharField(widget=forms.PasswordInput, label="Пароль")
    password2 = forms.CharField(widget=forms.PasswordInput, label="Повтор пароля")

    class Meta:
        model = User
        fields = ["username", "full_name", "phone", "email"]

    def clean(self):
        cleaned = super().clean()
        if cleaned.get("password") != cleaned.get("password2"):
            self.add_error("password2", "Пароли не совпадают")
        return cleaned

class LoginForm(AuthenticationForm):
    username = forms.CharField(label="Логин")
    password = forms.CharField(widget=forms.PasswordInput, label="Пароль")

class OrderForm(forms.ModelForm):
    class Meta:
        model = Order
        fields = ["product", "quantity", "address"]
```

---

## shop/views.py
```python name=shop/views.py
from django.contrib import messages
from django.contrib.auth import login
from django.contrib.auth.decorators import login_required
from django.shortcuts import render, redirect
from .forms import RegisterForm, LoginForm, OrderForm
from .models import Product, Order

def register_view(request):
    form = RegisterForm(request.POST or None)
    if request.method == "POST" and form.is_valid():
        user = form.save(commit=False)
        user.set_password(form.cleaned_data["password"])
        user.save()
        messages.success(request, "Регистрация успешна. Войдите.")
        return redirect("login")
    return render(request, "shop/register.html", {"form": form})

def login_view(request):
    form = LoginForm(request, data=request.POST or None)
    if request.method == "POST" and form.is_valid():
        login(request, form.get_user())
        return redirect("catalog")
    return render(request, "shop/login.html", {"form": form})

def catalog_view(request):
    products = Product.objects.all()
    return render(request, "shop/catalog.html", {"products": products})

@login_required
def order_create_view(request):
    form = OrderForm(request.POST or None)
    if request.method == "POST" and form.is_valid():
        order = form.save(commit=False)
        order.user = request.user
        order.save()
        messages.success(request, "Заказ создан")
        return redirect("orders")
    return render(request, "shop/order_create.html", {"form": form})

@login_required
def orders_view(request):
    orders = Order.objects.filter(user=request.user)
    return render(request, "shop/orders.html", {"orders": orders})
```

---

## shop/urls.py
```python name=shop/urls.py
from django.urls import path
from .views import register_view, login_view, catalog_view, order_create_view, orders_view

urlpatterns = [
    path("", catalog_view, name="catalog"),
    path("register/", register_view, name="register"),
    path("login/", login_view, name="login"),
    path("orders/", orders_view, name="orders"),
    path("order/new/", order_create_view, name="order_create"),
]
```

---

## shop/admin.py
```python name=shop/admin.py
from django.contrib import admin, messages
from django.contrib.auth.admin import UserAdmin
from .models import User, Product, Order

@admin.register(User)
class CustomUserAdmin(UserAdmin):
    fieldsets = UserAdmin.fieldsets + (
        ("Доп. данные", {"fields": ("full_name", "phone")}),
    )

@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ("name", "price")

@admin.register(Order)
class OrderAdmin(admin.ModelAdmin):
    list_display = ("user", "product", "quantity", "status")
    list_filter = ("status", "product")
    actions = ["make_confirmed", "make_canceled"]

    def make_confirmed(self, request, queryset):
        updated = queryset.update(status=Order.STATUS_CONFIRMED)
        self.message_user(request, f"Подтверждено: {updated}", level=messages.SUCCESS)

    def make_canceled(self, request, queryset):
        updated = queryset.update(status=Order.STATUS_CANCELED)
        self.message_user(request, f"Отменено: {updated}", level=messages.WARNING)

    make_confirmed.short_description = "Подтвердить выбранные"
    make_canceled.short_description = "Отменить выбранные"
```

---

# ✅ Шаблоны

## base.html
```html name=shop/templates/shop/base.html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Авоська</title>
  <link rel="stylesheet" href="/static/shop/css/styles.css">
</head>
<body>
<header>
  <h1>Авоська</h1>
  <nav>
    <a href="/">Каталог</a>
    <a href="/orders/">Заказы</a>
    <a href="/order/new/">Новый заказ</a>
    <a href="/register/">Регистрация</a>
    <a href="/login/">Вход</a>
  </nav>
</header>
<main>
  {% if messages %}
    <div class="messages">
      {% for message in messages %}
        <div class="msg">{{ message }}</div>
      {% endfor %}
    </div>
  {% endif %}
  {% block content %}{% endblock %}
</main>
<script src="/static/shop/js/slider.js"></script>
</body>
</html>
```

---

## register.html
```html name=shop/templates/shop/register.html
{% extends "shop/base.html" %}
{% block content %}
<h2>Регистрация</h2>
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Зарегистрироваться</button>
</form>
{% endblock %}
```

---

## login.html
```html name=shop/templates/shop/login.html
{% extends "shop/base.html" %}
{% block content %}
<h2>Авторизация</h2>
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Войти</button>
</form>
{% endblock %}
```

---

## catalog.html (слайдер на `<img>`)
```html name=shop/templates/shop/catalog.html
{% extends "shop/base.html" %}
{% block content %}
<h2>Каталог</h2>

<div class="slider">
  <button class="prev">←</button>
  <img id="slide" src="/static/shop/img/slide1.jpg" alt="Слайд">
  <button class="next">→</button>
</div>

<div class="products">
  {% for p in products %}
    <div class="card">
      <h3>{{ p.name }}</h3>
      <p>{{ p.description }}</p>
      <p><b>{{ p.price }} ₽</b></p>
      <a href="/order/new/">Оформить заказ</a>
    </div>
  {% endfor %}
</div>
{% endblock %}
```

---

## order_create.html
```html name=shop/templates/shop/order_create.html
{% extends "shop/base.html" %}
{% block content %}
<h2>Оформление заказа</h2>
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Отправить заказ</button>
</form>
{% endblock %}
```

---

## orders.html
```html name=shop/templates/shop/orders.html
{% extends "shop/base.html" %}
{% block content %}
<h2>Мои заказы</h2>
<ul>
  {% for o in orders %}
    <li>{{ o.product.name }} — {{ o.get_status_display }}</li>
  {% empty %}
    <li>Заказов нет</li>
  {% endfor %}
</ul>
{% endblock %}
```

---

# ✅ Static

## styles.css
```css name=shop/static/shop/css/styles.css
body { font-family: Arial, sans-serif; margin: 0; background: #f5f5f5; }
header { background: #2f6f4e; color: #fff; padding: 10px; }
nav a { color: #fff; margin-right: 10px; text-decoration: none; }
main { padding: 20px; }
.card { background: #fff; padding: 10px; margin: 10px 0; border-radius: 8px; }
button { background: #2f6f4e; color: #fff; border: none; padding: 8px 12px; cursor: pointer; }
.slider { display: flex; align-items: center; gap: 10px; margin: 10px 0; }
.slider img { width: 260px; height: 140px; object-fit: cover; background: #ddd; }
@media (max-width: 390px) {
  .slider img { width: 100%; height: auto; }
}
```

---

## slider.js
```javascript name=shop/static/shop/js/slider.js
const slides = [
  "/static/shop/img/slide1.jpg",
  "/static/shop/img/slide2.jpg",
  "/static/shop/img/slide3.jpg",
  "/static/shop/img/slide4.jpg",
];

let index = 0;
const img = document.getElementById("slide");

function showSlide(i) {
  index = (i + slides.length) % slides.length;
  img.src = slides[index];
}

document.querySelector(".next")?.addEventListener("click", () => showSlide(index + 1));
document.querySelector(".prev")?.addEventListener("click", () => showSlide(index - 1));

setInterval(() => showSlide(index + 1), 3000);
```

---

# ✅ Как добавить картинки
Положите **любые 4 картинки** в папку:
```
shop/static/shop/img/
```

И назовите их:
```
slide1.jpg
slide2.jpg
slide3.jpg
slide4.jpg
```

---

# ✅ Запуск
```bash
pip install -r requirements.txt
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
# логин: sklad
# пароль: 123qwe
python manage.py runserver
```

Админка:
```
http://127.0.0.1:8000/admin/
```

---

Если хотите — могу:
✅ сразу добавить тестовые товары  
✅ улучшить дизайн под ваши критерии  
✅ сделать авто-добавление пользователя sklad/123qwe  
✅ добавить фильтрацию и всплывающие уведомления в клиенте  
