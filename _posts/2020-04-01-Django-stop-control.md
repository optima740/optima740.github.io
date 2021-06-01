---
layout: post
title: Django. Стоп, контроль!
category: django
---
![](/image/post-2020-04-01/stop-control2.jpg)

Для большинства современных веб-приложений наличие механизма аутентификации пользователя является обязательным условием. 
Но, для начала давайте поймем разницу между такими, казалось бы, похожими понятиями, как идентификация, аутентификация и авторизация.  
Итак, **Идентификация** — это процесс определения, что за пользователь перед нами. **Аутентификация** — это процесс подтверждения, что этот пользователь именно тот, за кого себя выдает, то есть, процесс сопоставления переданных данных от пользователя с существующими учетными данными в БД. **Авторизация** — это процесс принятия решения о том, что именно этой аутентифицированной персоне разрешается делать, то есть, назначение прав пользователю для разграничения доступа к ресурсам.  
В Django имеется встроенный функционал для реализации процесса аутентификации и авторизации. Этим самым "готовым кубиком" служит приложение `auth` из `django.contrib.auth`. 
Что бы работал сам механизм аутентификации и авторизации пользователя, необходимо изначально в моделях (models) использовать для создания пользователя встроенный в Django класс **User**, и уже при необходимости наследовать от него свои пользовательские классы. Именно в стандартном встроенном классе User реализована механика взаимодействия при аутентификации и авторизации.   
Приложение аутентификации Django предоставляет следующие функциональные возможности из «коробки»:  

•	Класс авторизации `LoginView`  
•	Класс выхода `LogoutView`  
•	Сброс пароля `PasswordResetView`  
•	Смена пароля `PasswordChangeView`  

Нам нужно только предоставить шаблоны для реализации этих функций в нашем приложении.
Все примеры, описанные ниже, относятся к версии Django 3 и выше.  

Для начала, подготовим необходимый шаблон для прохождения процесса аутентификации. Проще говоря, форму, куда будут вноситься пользовательские данные username и password. Создадим директорию в корне нашего проекта: `templates/registration`, а внутри `registration` создаем два шаблона `login.html` и `logged_out.html` для "залогинивания" и "разлогинивания" соответственно.  
Для того, чтобы все корректно работало, внутри директории `templates/`(если он еще не создан) добавим еще один файл `base_generic.html` с содержимым:
```html
{% raw %}
<!DOCTYPE html>
<html lang="en">
<head>
  {% block title %}<title>Local Library</title>{% endblock %}
</head>
<body>
  {% block sidebar %}
  {% if user.is_authenticated %}
     <li>User: {{ user.get_username }}</li>
     <li><a href="{% url 'logout'%}?next={{request.path}}">Logout</a></li>
   {% else %}
     <li><a href="{% url 'login'%}?next={{request.path}}">Login</a></li>
   {% endif %}
  {% endblock %}
  {% block content %}<!-- default content text (typically empty) -->{% endblock %}
</body>
</html>
{% endraw %}
```
Далее, с учетом расширения `base_generic.html`, создадим `login.html`:
```html
{% raw %}
{% extends "base_generic.html" %}
{% block content %}
  {% if form.errors %}
    <p>Your username and password didn't match. Please try again.</p>
  {% endif %}  
  {% if next %}
    {% if user.is_authenticated %}
      <p>Your account doesn't have access to this page. To proceed,
      please login with an account that has access.</p>
    {% else %}
      <p>Please login to see this page.</p>
    {% endif %}
  {% endif %}  
  <form method="post" action="{% url 'login' %}">
    {% csrf_token %}
    <table>
      <tr>
        <td>{{ form.username.label_tag }}</td>
        <td>{{ form.username }}</td>
      </tr>
      <tr>
        <td>{{ form.password.label_tag }}</td>
        <td>{{ form.password }}</td>
      </tr>
    </table>
    <input type="submit" value="login" />
    <input type="hidden" name="next" value="{{ next }}" />
  </form>  
  {# Assumes you setup the password_reset view in your URLconf #}
  <p><a href="{% url 'password_reset' %}">Lost password?</a></p> 
{% endblock %}
{% endraw %}
```
И, соответственно, `logged_out.html`:
```html
{% raw %}
{% extends "base_generic.html" %}
{% block content %}
  <p>Logged out!</p>  
  <a href="{% url 'login'%}">Click here to login again.</a>
{% endblock %}
{% endraw %}
```
Далее установим параметр редиректа (перенаправления) после удачной аутентификации пользователя. Для этого в файле `settings.py` добавим параметр:   
```
LOGIN_REDIRECT_URL = '/'
```
`'/'` – означает, что будет произведен редирект на главную страницу сайта.
Затем, делаем связывание пути и представления (вьюхи), которое будет отрабатывать саму логику аутентификации. В основном файле `rls.py`, который находится на одном уровне с  `settings.py`. свяжем адрес `accounts/`, и встроенное представление для аутентификации - `django.contrib.auth.urls`, которое и будет отвечать за всю дальнейшую логику, своего рода "черный ящик":
```python
path('accounts/', include('django.contrib.auth.urls'))
```
Тем самым, пройдя по ссылке ..`/accounts/login/` , `django.contrib.auth.urls` обратится в директорию `templates/registration` за поиском шаблона `login.html` для отображения, который мы предусмотрительно заранее создали. 
Запускаем наш локальный сервер (manage.py runserver) и проверяем работу.  
Аналогичным образом можно реализовать возможности выхода (logged_out) и смены пароля (Password_Change).














