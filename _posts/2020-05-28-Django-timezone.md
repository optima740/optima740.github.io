﻿---
layout: post
title: Django. Время поговорить о времени.
---
![](/image/post-2020-05-28/time2.jpg)
Сегодня я хотел бы сделать небольшие заметки относительно тайм-зон и о том, как я столкнулся с их применением в Django.  
**Тайм-зона** – это привязка к часовому поясу. Как мы знаем, часовой пояс это, по сути, поправка, на которую корректируют время по географическому расположению относительно UTC. В свою очередь, **UTC** (Universal Time Coordinated) – это, своего рода, абсолютное время, относительно которого уже идет смещение поправки по часовым поясам в зависимости от географического расположения пользователя.  

В Django настройки тайм-зоны выставляются глобально в файле `settings.py`  


```python
TIME_ZONE = 'UTC'
USE_TZ = True
```

Полем `TIME_ZONE` (для БД не поддерживающих часовые пояса) мы определяем, непосредственно, сам часовой пояс. В данном случае UTC. 
`USE_TZ` – этим полем мы определяем для Django, что значение времени (например, поле модели типа `DateTimeField`) будет записано в базу данных в той тайм-зоне, которая указана в настройке проекта Django в поле `TIME_ZONE`.   
Тут сразу следует сделать оговорку. Не все базы данных поддерживают формат времени с учетом тайм-зоны. Например, SQLite, MySQL, Oracle не поддерживают часовые пояса (а как раз SQLite, идет как стандартная БД для Django “из коробки”) – и в данном случае , при `USE_TZ = True`, фреймворк запишет время с учетом установки `TIME_ZONE`. 
В случае, если в проекте Django используется БД, поддерживающая работу с часовыми поясами (например, PostgreSQL), то опция `USE_TZ` должна быть установлена в `False`, тем самым позволяя БД работать с часовым поясом, переданным в пользовательских данных без учета настройки проекта Django `TIME_ZONE`.  

Давайте рассмотрим следующий случай. В нашем приложении по учету автомобилей есть несколько предприятий. Для каждого предприятия нам необходимо реализовать возможность установки тайм-зоны путем выбора из предложенных вариантов часовых поясов, чтобы каждое предприятие могло работать в своем часовом поясе. Если при создании предприятия тайм-зона не выбрана, то это значение соответствует UTC. В `models.py` добавим поле для нашей модели `Enterprise:`

```python
class Enterprise(models.Model):
    TIMEZONES = tuple(zip(pytz.all_timezones, pytz.all_timezones))
    title = models.CharField(max_length=32)
    address = models.CharField(max_length=64)
    time_zone = models.CharField(max_length=32, choices=TIMEZONES, default='UTC')
```

Далее, для взаимодействия с моделями автомобилей добавим в модель `Car` поле, отвечающее за дату и время приобретения автомобиля, или постановки на учет в данном предприятии - как угодно. Данные о приобретении автомобиля заносятся автоматически и значение берется из текущего времени пользователя (`auto_now=True`) с поправкой на настройку в `settings.py` (мы используем стандартную БД SQLite). Т.о. время в поле модели `Enterprise` запишется в абсолютном значении UTC.

```python
class Car(models.Model):
    brand = models.CharField(max_length=32)
    model = models.CharField(max_length=32)
    color = models.CharField(max_length=16)
    fuel_util = models.CharField(max_length=5)
    date_of_buy = models.DateTimeField(auto_now=True)
    of_enterprise = models.ForeignKey('Enterprise', on_delete=models.SET_NULL, null=True)
```

Т.к. в наш проект ранее мы интегрировали REST API, то изменять (делать поправку на часовой пояс предприятия) необходимо не в HTML шаблоне, а на стороне бэкенда для того, чтобы API сформировал уже готовые данные с учетом поправки на тайм-зону, и отдал эти данные (например, словарем) во фронтенд. Тем самым мы придерживаемся концепции API и оставляем логику на стороне бэкенда в нашем `views.py`: 

```python
def get_queryset(self):
    # переопределяем родительский метод для запроса из БД с фильтром для наших условий
    user_name = self.request.user.username  # получаем данные о имени пользователя из request
    list_car = Car.objects.all()            # формируем список всех автомобилей
    enterprise = Enterprise.objects.get(manager__username__contains=user_name)  # получаем предприятие по пользователю
    list_car = Car.objects.filter(of_enterprise=enterprise)   # получаем список авто по конкретному предприятию
    tz = self.get_time_zone()                                 # получаем таймзону из предприятия текущего менеджера
    for car in list_car:   # проходимся по всем автомобилям из списка запроса
        car_of_db = Car.objects.get(id=car.id)        # получаем из БД  машину по id
        datatime_of_db = car_of_db.date_of_buy        # получаем из поля значение даты и времени покупки машины
        datatime_tz = datatime_of_db.replace(tzinfo=tz)       # корректировка даты и времени относительно другой tz
        car.date_of_buy = datatime_tz                 # записываем откорректированное время в список для отображения в шаблоне
    return list_car    

def get_time_zone(self):
    user_name = self.request.user.username
    enterprise = Enterprise.objects.get(manager__username__contains=user_name)   # получаем предприятие по пользователю
    tz = pytz.timezone(enterprise.time_zone)     # получаем таймзону из предприятия текущего менеджера
    return tz
```
Я постарался максимально подробно в коде расписать каждое действие пошагово, в целях бОльшей понятности.  
  
В итоге, наш API будет отдавать словарь с уже исправленным временем, применительно к часовому поясу того предприятия, за которым закреплены автомобиль и авторизованный сотрудник, который делает запрос на просмотр списка доступных авто.



