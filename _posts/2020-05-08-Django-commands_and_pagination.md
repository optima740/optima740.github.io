---
layout: post
title: Django. Пользовательские команды и Пагинация.
---
![](/image/post-2020-05-08/head.jpg)

В Django имеется функционал создания своих собственных пользовательских консольных команд для администрирования проекта.  

Давайте рассмотрим пример создания такой команды, применительно к нашему учебному проекту с автопредприятиями. Наша команда будет генерировать указанное количество автомобилей применительно к предприятию (или списка предприятий), которое также будет указываться в параметрах команды.  

Для этого создадим каталог `management` в основном каталоге проекта (на уровне каталогов `static` и `аpi`). В management создадим подкаталог `commands` и уже непосредственно в нем будем создавать наши команды. Для того чтобы Python учитывал каталог `commands`, как свой пакет, необходимо добавить пустой файл `__init__.py`. Для получения пользовательской команды достаточно создать питоновский файл с таким-же именем, как и у предполагаемой команды. Например, для команды генерирующей наши машинки – `gencar`, мы создаем `gencar.py`.  

Создадим класс команды - `Command` , наследованный от класса `BaseCommand`, который предоставляется фреймворком:
```python
from django.core.management.base import BaseCommand
from django.utils.crypto import get_random_string
from rest_framework.generics import get_object_or_404
from ...models import *

class Command(BaseCommand):
    # Задаём текст помощи, который будет отображён при выполнении команды
    # python manage.py gencars --help
    help = u'Generate cars. Создание моделей Car со случайным содержанием полей'

    def add_arguments(self, parser):
    # Указываем сколько и каких аргументов принимает команда.
    # В данном случае первый аргументом идет id предприятия, nargs='+' - минимум один аргумент, либо список.
    parser.add_argument('-e', '--enterprise_id', nargs='+', type=int,
                        help=u'Для какого предприятия(либо список enterprise_id через пробел)')
    # Вторым аргументом передаем число
    parser.add_argument('-n', '--number', type=int, help=u'Количество создаваемых Car')

    def handle(self, *args, **options):
        # Получаем аргументы:
        enterprise_id = options['enterprise_id']
        number_cars = options['number']
        # enterprise_id - возможно список, по этому итерируемся сначала по нему.
        for enterprise in enterprise_id:
            of_enterprise = get_object_or_404(Enterprise.objects.all(), pk=enterprise)
            # Создаем число машинок = number_cars
            for i in range(number_cars):
                brand = get_random_string() # случайная генерация строки для поля Car
                model = get_random_string()
                color = get_random_string()
                # Непосредственно, создание модели типа Car, с указанными полями
                Car.objects.create(brand=brand, model=model, color=color, fuel_util=10, of_enterprise=of_enterprise)
	self.stdout.write('Successfully created {0} cars!'.format(number_cars))
        self.stdout.write('Current list cars: {}'.format(list(Car.objects.all())))
```
`add_arguments()`- в данной функции мы определяем сколько и каких аргументов будет передано в команду. В нашем случае мы используем два параметра:  
   
`-e` (enterprise), где указываем id предприятия, к которому будет привязана созданная машинка, или список id предприятий.  

`-n` (number) – задает количество моделей для генерации (сколько машинок будет создано).  

`handle()` – переопределяя эту функцию, мы задаем обработку переданных параметров, и дальнейшую логику действий по реализации семантики нашей команды. Все аргументы приходят в списке (options). Также в данной функции для генерации случайного текста мы использовали метод `get_random_string()`, импортированного из `django.utils.crypto`.   
Далее, если мы введем в командной строке:  
```
python manage.py gencar –e 1 –n 10
```
у нас создастся 10 машинок прикрепленный к предприятию с id 1.  

Также мы можем создать, например, по 50 машинок для предприятия с id 1 и id 2:
```
python manage.py gencar –e 1 2 –n 50
```
Теперь у нас появилась возможность массово создавать модели Car со случайным значением полей. Но как быть с отображением? Напомню, что сейчас список объектов Car отображается вот так:  
 
![](/image/post-2020-05-08/no_paginations.jpg)   
Что будет, если мы создадим, скажем, 5000 машинок? В данном случае необходимо реализовать постраничное отображение – пагинацию.  

**Пагинация** – это разбиение на страницы или постраничная нумерация. В Django REST фреймворке есть стандартные классы для реализации данного функционала. Я опишу самый простой способ сделать это в рамках REST фреймворка.  

Для начала укажем в глобальных настройках нашего проекта (`settings.py`) класс, который будет использоваться для пагинации по умолчанию:
```python
REST_FRAMEWORK = {
        # глобальная настройка для стиля пагинации с указанием количества элементов на странице PAGE_SIZE
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 5,
}
```
`'PAGE_SIZE'`- определяет количество элементов на одной странице.  

Далее в нашем файле с представлением (api\view.py) переопределим поле `pagination_class = LimitOffsetPagination`
```python
class CarView(LoginRequiredMixin, ListCreateAPIView):
    # представление для списка авто с методами create и get
    serializer_class = CarSerializer
    pagination_class = LimitOffsetPagination

``` 
Также, необходимо добавить логику в наш метод get():
```python
def get(self, request):
    # переопределяем родительский метод для получения запроса GET
    queryset = self.get_queryset()
    serializer = CarSerializer(queryset, many=True)
    page = self.paginate_queryset(queryset) # формируем данные для отображения на одной странице
    if page is not None:
        serializer = CarSerializer(page, many=True) # сериализуем данные page
        return self.get_paginated_response(serializer.data)
    return Response({"cars": serializer.data})
```
Тут мы создаем экземпляр класса нашего пагинатора, передаем ему результат запроса (queryset). Объект `page` – будет являться набором данных для одной страницы. Далее мы через сериализацию выводим данные.  

Как можно увидеть, теперь у нас есть постраничное отображение списка Car:

![](/image/post-2020-05-08/paginations.jpg)  










