# Финальный ультракороткий гайд:  Корочки.есть

## Шаг 1: Создание проекта

```bash
mkdir korochki && cd korochki
python -m venv venv
venv\Scripts\activate
pip install django
django-admin startproject config .
python manage.py startapp main
mkdir main\templates
```

---

## Шаг 2: Структура

```
korochki/
├── config/
│   ├── settings.py
│   └── urls.py
├── main/
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   └── templates/
│       ├── base.html
│       ├── reg.html
│       ├── auth.html
│       ├── apps.html
│       ├── create.html
│       └── admin.html
└── manage.py
```

---

## Шаг 3: Файлы

### 3.1 config/settings. py

Найди `INSTALLED_APPS` и добавь `'main'`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django. contrib.messages',
    'django.contrib.staticfiles',
    'main',
]
```

---

### 3.2 config/urls.py

```python
from django.urls import path, include
urlpatterns = [path('', include('main.urls'))]
```

---

### 3.3 main/models. py

```python
from django.db import models
from django.contrib.auth.models import User

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    fio = models.CharField(max_length=100)
    phone = models.CharField(max_length=20)

class App(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    course = models.CharField(max_length=100)
    date = models.CharField(max_length=10)
    pay = models.CharField(max_length=30)
    status = models.CharField(max_length=30, default='Новая')
    review = models.TextField(blank=True)
```

---

### 3.4 main/urls.py

```python
from django.urls import path
from . import views
urlpatterns = [
    path('', views.auth, name='auth'),
    path('reg/', views.reg, name='reg'),
    path('apps/', views. apps, name='apps'),
    path('create/', views.create, name='create'),
    path('panel/', views.panel, name='panel'),
    path('out/', views.out, name='out'),
]
```

---

### 3.5 main/views.py

```python
from django.shortcuts import render, redirect
from django.contrib.auth import login, authenticate, logout
from django.contrib.auth.models import User
from . models import Profile, App
import re

try:
    if not User.objects.filter(username='Admin').exists():
        User.objects.create_user('Admin', '', 'KorokNET')
except:  pass

def reg(request):
    e = {}
    if request.method == 'POST':
        lo, pw = request.POST['login'], request.POST['pw']
        fio, phone, email = request.POST['fio'], request.POST['phone'], request.POST['email']
        if len(lo) < 6 or not re.match(r'^[a-zA-Z0-9]+$', lo): e['login'] = 'Мин 6, латиница'
        elif User.objects.filter(username=lo).exists(): e['login'] = 'Занят'
        if len(pw) < 8: e['pw'] = 'Мин 8 символов'
        if not re. match(r'^[а-яА-ЯёЁ\s]+$', fio): e['fio'] = 'Только кириллица'
        if not re.match(r'^8\(\d{3}\)\d{3}-\d{2}-\d{2}$', phone): e['phone'] = '8(XXX)XXX-XX-XX'
        if not re.match(r'^[^@]+@[^@]+\.[^@]+$', email): e['email'] = 'Неверный email'
        if not e:
            u = User.objects.create_user(lo, email, pw)
            Profile.objects.create(user=u, fio=fio, phone=phone)
            login(request, u); return redirect('apps')
    return render(request, 'reg.html', {'e': e})

def auth(request):
    e = ''
    if request.method == 'POST':
        u = authenticate(username=request.POST['login'], password=request.POST['pw'])
        if u:  login(request, u); return redirect('panel' if u.username == 'Admin' else 'apps')
        e = 'Неверные данные'
    return render(request, 'auth.html', {'e': e})

def apps(request):
    if not request.user.is_authenticated: return redirect('auth')
    if request.method == 'POST' and 'review' in request.POST:
        a = App.objects.get(id=request.POST['id'], user=request.user)
        if a.status == 'Завершено': a.review = request.POST['review']; a.save()
    return render(request, 'apps.html', {'apps': App. objects.filter(user=request. user)})

def create(request):
    if not request.user.is_authenticated: return redirect('auth')
    err = ''
    if request.method == 'POST':
        if re.match(r'^\d{2}\.\d{2}\.\d{4}$', request.POST['date']):
            App.objects.create(user=request.user, course=request.POST['course'], date=request.POST['date'], pay=request.POST['pay'])
            return redirect('apps')
        err = 'Формат ДД.ММ. ГГГГ'
    return render(request, 'create.html', {'err': err})

def panel(request):
    if not request.user.is_authenticated or request.user.username != 'Admin': return redirect('auth')
    if request.method == 'POST': 
        a = App.objects. get(id=request.POST['id']); a.status = request.POST['st']; a.save()
    apps = App.objects.all()
    st = request.GET.get('st')
    if st:  apps = apps.filter(status=st)
    return render(request, 'admin.html', {'apps': apps})

def out(request): logout(request); return redirect('auth')
```

---

### 3.6 main/templates/base.html

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Корочки.есть</title>
<style>
*{margin: 0;padding:0;box-sizing:border-box}
body{font-family:Arial;background: linear-gradient(135deg,#667eea,#764ba2);min-height:100vh}
nav{background:#2d3436;padding:15px;display:flex;gap:15px}
nav a{color:#fff;text-decoration:none}
main{max-width:700px;margin:20px auto;background:#fff;padding:25px;border-radius:12px}
h1{text-align:center;margin-bottom:20px}
input,select{width:100%;padding:10px;margin: 5px 0;border:1px solid #ccc;border-radius:5px}
button,. btn{background:#667eea;color:#fff;padding:10px 20px;border:none;border-radius:5px;cursor:pointer;text-decoration:none}
. e{color: red;font-size:12px}
table{width:100%;border-collapse:collapse}th,td{padding:10px;border-bottom:1px solid #eee;text-align:left}
. f a{margin-right:10px;padding:5px 10px;background:#eee;border-radius:10px;text-decoration:none}
.slider{display:flex;justify-content:center;align-items:center;padding:10px;gap:8px}
.slider img{width:500px;height:120px;object-fit:cover;border-radius:8px}
. slider button{padding:5px 10px;border:none;background:#fff;border-radius:50%;cursor:pointer}
@media(max-width:430px){nav{flex-direction:column;align-items:center}main{margin:10px;padding:15px}. slider img{width:300px;height:80px}}
</style>
</head>
<body>
<nav>
<a href="/"><b>Корочки.есть</b></a>
{% if user.is_authenticated %}<a href="/apps/">Заявки</a><a href="/create/">Новая</a><a href="/out/">Выход</a>{% endif %}
</nav>
<div class="slider">
<button onclick="s(-1)">❮</button>
<img id="si" src="https://picsum.photos/500/120?1">
<button onclick="s(1)">❯</button>
</div>
<main>{% block c %}{% endblock %}</main>
<script>
let i=0,imgs=['https://picsum.photos/500/120?1','https://picsum.photos/500/120? 2','https://picsum.photos/500/120?3','https://picsum.photos/500/120?4'];
function s(d){i=(i+d+4)%4;document.getElementById('si').src=imgs[i]}
setInterval(function(){s(1)},3000);
</script>
</body>
</html>
```

---

### 3.7 main/templates/auth.html

```html
{% extends 'base.html' %}
{% block c %}
<h1>Вход</h1>
<form method="post">{% csrf_token %}
<input name="login" placeholder="Логин" required>
<input name="pw" type="password" placeholder="Пароль" required>
{% if e %}<p class="e">{{e}}</p>{% endif %}
<button>Войти</button>
</form>
<p style="margin-top:15px"><a href="/reg/">Нет аккаунта?  Регистрация</a></p>
{% endblock %}
```

---

### 3.8 main/templates/reg.html

```html
{% extends 'base.html' %}
{% block c %}
<h1>Регистрация</h1>
<form method="post">{% csrf_token %}
<input name="login" placeholder="Логин (6+ симв, латиница)" required>{% if e. login %}<p class="e">{{e.login}}</p>{% endif %}
<input name="pw" type="password" placeholder="Пароль (8+ симв)" required>{% if e.pw %}<p class="e">{{e. pw}}</p>{% endif %}
<input name="fio" placeholder="ФИО (кириллица)" required>{% if e.fio %}<p class="e">{{e.fio}}</p>{% endif %}
<input name="phone" placeholder="8(999)123-45-67" required>{% if e.phone %}<p class="e">{{e.phone}}</p>{% endif %}
<input name="email" placeholder="Email" required>{% if e.email %}<p class="e">{{e.email}}</p>{% endif %}
<button>Зарегистрироваться</button>
</form>
<p style="margin-top:15px"><a href="/">Есть аккаунт? Войти</a></p>
{% endblock %}
```

---

### 3.9 main/templates/apps.html

```html
{% extends 'base.html' %}
{% block c %}
<h1>Мои заявки</h1>
<a href="/create/" class="btn">+ Новая</a>
<table>
<tr><th>Курс</th><th>Дата</th><th>Оплата</th><th>Статус</th><th>Отзыв</th></tr>
{% for a in apps %}
<tr>
<td>{{a.course}}</td><td>{{a.date}}</td><td>{{a.pay}}</td><td>{{a.status}}</td>
<td>{% if a.status == 'Завершено' %}{% if a.review %}{{a.review}}{% else %}
<form method="post" style="display:inline">{% csrf_token %}<input name="id" value="{{a.id}}" type="hidden">
<input name="review" placeholder="Отзыв" style="width:80px"><button>OK</button></form>{% endif %}{% else %}—{% endif %}</td>
</tr>
{% empty %}<tr><td colspan="5">Заявок нет</td></tr>{% endfor %}
</table>
{% endblock %}
```

---

### 3.10 main/templates/create.html

```html
{% extends 'base.html' %}
{% block c %}
<h1>Новая заявка</h1>
<form method="post">{% csrf_token %}
<select name="course">
<option>Основы алгоритмизации и программирования</option>
<option>Основы веб-дизайна</option>
<option>Основы проектирования баз данных</option>
</select>
<input name="date" placeholder="ДД.ММ.ГГГГ" required>{% if err %}<p class="e">{{err}}</p>{% endif %}
<p style="margin: 10px 0">Оплата: </p>
<label><input type="radio" name="pay" value="Наличными" checked> Наличными</label><br>
<label><input type="radio" name="pay" value="Перевод"> Перевод по телефону</label><br><br>
<button>Отправить</button>
</form>
{% endblock %}
```

---

### 3.11 main/templates/admin.html

```html
{% extends 'base.html' %}
{% block c %}
<h1>Админ-панель</h1>
<div class="f"><a href="/panel/">Все</a><a href="/panel/? st=Новая">Новые</a><a href="/panel/? st=Идет обучение">В процессе</a><a href="/panel/?st=Завершено">Готово</a></div>
<table>
<tr><th>User</th><th>Курс</th><th>Дата</th><th>Оплата</th><th>Статус</th></tr>
{% for a in apps %}
<tr>
<td>{{a.user}}</td><td>{{a.course}}</td><td>{{a. date}}</td><td>{{a.pay}}</td>
<td><form method="post">{% csrf_token %}<input name="id" value="{{a. id}}" type="hidden">
<select name="st" onchange="this.form.submit()">
<option {% if a.status == 'Новая' %}selected{% endif %}>Новая</option>
<option {% if a.status == 'Идет обучение' %}selected{% endif %}>Идет обучение</option>
<option {% if a.status == 'Завершено' %}selected{% endif %}>Завершено</option>
</select></form></td>
</tr>
{% empty %}<tr><td colspan="5">Заявок нет</td></tr>{% endfor %}
</table>
{% endblock %}
```

---

## Шаг 4: Запуск

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

**Админ создастся автоматически! ** (Admin / KorokNET)

---

## Шаг 5: Проверка

| URL | Описание |
|-----|----------|
| http://127.0.0.1:8000/ | Вход |
| http://127.0.0.1:8000/reg/ | Регистрация |
| http://127.0.0.1:8000/apps/ | Заявки |
| http://127.0.0.1:8000/create/ | Новая заявка |
| http://127.0.0.1:8000/panel/ | Админка |

---

## Итого:  ~200 строк 🚀

| Файл             | Строк    |
| ---------------- | -------- |
| urls.py (config) | 2        |
| models.py        | 13       |
| urls.py (main)   | 9        |
| views.py         | 55       |
| base.html        | 28       |
| auth.html        | 10       |
| reg.html         | 12       |
| apps.html        | 14       |
| create.html      | 14       |
| admin.html       | 16       |
| **ИТОГО**        | **~173** |