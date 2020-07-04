﻿---
layout: post
title: Django. Немного о SCRF.
---
![](/image/post-2020-04-20/csrf.jpg)


**Cross-Site Request Forgery** – если адаптировать на русский язык, то получается что-то вроде межсайтовая подделка запроса. Суть данной атаки в том, что браузер не может различить, было ли действие выполнено пользователем умышленно (например, переход по ссылке или нажатие на кнопку отправки формы на целевой ресурс), или пользователь выполнил действие не умышленно (например, при посещении ресурса-злоумышленника, этот ресурс отправил запрос от имени пользователя на целевой ресурс, на котором пользователь остался «залогиненым»).   
Получается, что ресурс-злоумышленник, вклинившись между пользователем и целевым ресурсом (**cross-site**), может отправить HTTP запрос (**request**), используя авторизированные данные пользователя, и совершить какие-либо действия от имени пользователя на целевом ресурсе (**forgery**).  
  
![](/image/post-2020-04-20/csrf-attack.jpg)  
На сегодня общепринятым методом защиты от подобного рода атак, является использование csrf-токенов. **Токен** - сгенерированный случайным образом сервером ключ, который передается клиенту и возвращается обратно серверу для проверки.
Своеобразная система «свой» - «чужой».

В Django также реализован данный функционал. CSRF защита является частью ядра Django. Кроме того, мы можем использовать декоратор `django.views.decorators.csrf.csrf_protect` для обеспечения защиты для конкретных вьюх.   
Django использует механизм csrf protect. Он заключается в том, чтобы отдать токен клиенту двумя методами: в куках и в одном из параметров ответа (header или внутри HTML).  
При запросе клиента на стороне сервера генерится токен. В ответе сервера, токен возвращается в cookie (`X-CSRF-Token`) и в одном из параметров ответа (в header или внутри HTML).  
В последующих запросах клиент обязан передать оба,  полученных ранее токена. Один как cookie, другой либо как header, либо внутри POST данных формы.   
При получении запроса от клиента небезопасным методом (POST, PUT, DELETE, PATCH) сервер проверяет на идентичность токен из cookie и токен, который явно прислал клиент.  

Давайте проверим данный механизм при помощи curl.   
**Curl** – кроссплатформенная программа для командной строки. При помощи Curl мы можем взаимодействовать с сервером по различным протоколам. Нас, в частности, интересуют HTTP-запросы. 
Мы авторизуемся на сайте и попробуем обратиться, используя HTTP-запросы, чтобы получить список автомобилей, ограничение доступа к которому я описывал в предыдущих статьях.  
Для Windows, как для наиболее распространенной платформы, запускаем `cmd.exe` от имени админа. Итак, если мы просто отправим POST запрос с данными для авторизации:
```
curl -v -d "username=manager&password=1234567" http://127.0.0.1:8000/accounts/login/
```
В ответ получаем сообщение сервера с кодом 403, а именно `"In general, this can occur when there is a genuine Cross Site Request Forgery, or when"`, что говорит нам о не совпадении csrf токена.  
Для того, чтобы сервер понял, что мы «свои» сымитируем действия браузера. Сделаем сначала предварительный GET-запрос, затем сохраним полученный cookie файл, вытащим из него csrf-token и передадим его в повторном POST запросе c данными для авторизации:
```
curl -c cookie.txt http://127.0.0.1:8000/accounts/login/
```
Подставляем полученный токен из cookie.txt в следующий запрос. В нем мы передаем серверу, как сам cookie, так и поле c токеном: 
```
curl --cookie cookie.txt http://127.0.0.1:8000/accounts/login/ -H "X-CSRFToken: owPT8worwO0Bd8AEfqUg1FB2NIPkEQ8DHoJ0kAVPUm1hsSrMS0y0AT7T6qsUNqXa" -b "csrftoken=owPT8worwO0Bd8AEfqUg1FB2NIPkEQ8DHoJ0kAVPUm1hsSrMS0y0AT7T6qsUNqXa" -d "username=manager1&password=Qwertyu_123&next=" -c cookie.txt
```
В этот раз, сервер отвечает уже сообщением 302, что означает редирект. А мы помним, что в механизме авторизации редирект указывается в настройках, как адрес куда нас перенаправят при успешной авторизации.   
Итак, мы успешно авторизовались. Осталось получить список автомобилей. Для этого отправим GET-запрос вместе с cookie:
```
curl --cookie cookie.txt -v http://127.0.0.1:8000/api/cars/
```
Сервер ответил кодом 200 ОК. А ниже мы видим наш список в JSON формате:
![](/image/post-2020-04-20/curl-cmd.JPG)  









