﻿---
layout: post
title: Типы данных в Python.
category: "imperative"
---
![](/image/post-2021-06-29/logo1.jpg)  

Язык программирования Python из года в год набирает все большую популярность. Данный язык используется во множестве различных сфер, например, AI/ML, Big Data, Data Science, Computer Science. Кроме того, питон имеет достаточно невысокий порог входа для новичков и пропагандируется многими IT-лидерами, что также добавляет дров в печь толкающую популярность вверх.  

Сегодня я хотел бы сделать небольшой обзор встроенных типов данных, которые используются внутри языка.  
Для начала я постараюсь своими словами объяснить, что такое тип данных.  

**Тип данных** – это множество допустимых (возможных) значений, для какой-либо программной сущности. Если очень примитивно говорить (для более простого понимания), тип данных необходим для того, чтобы интерпретатор понимал сколько памяти необходимо выделить или зарезервировать под ту или иную сущность, описанную в программе.   

Python имеет **динамическую типизацию** (позднее связывание). Данный вид типизации еще принято называть «утиной типизацией», при которой конкретный тип объекта не важен, а важны лишь свойства и методы, которыми этот объект обладает. Другими словами, если объект плавает и крякает как утка, то вероятнее всего, это и есть утка!  
При динамической типизации, тип объекта (сущности) определяется не в момент компиляции (трансляции) программы (т.е. до ее запуска), а в момент исполнения программы (т.е. после ее запуска). Типы переменных неизвестны до того момента, пока у них не появляются конкретные значения при запуске программы.  

Наряду с динамической типизацией, питон **сильно типизирован**. Это означает, что неявное приведение типов исключено. Например, мы можем определить две переменные без указания типа, но мы не можем сложить эти две переменные, если в одной окажется строка, а в другой число. Компилятор в данном случае вмешается с уточнением.  
Кроме того, питон использует ссылочную модель данных, то есть оперирует не значениями, а ссылками (безопасными указателями) на значения (объекты). Я немного писал уже об этом вот в этой своей [статье](https://optima740.github.io/universe/2021/02/02/programming-universe4/).  

Также, я хотел бы отметить еще одну характеристику программных сущностей – способность к изменению. Сущность, которая позволяет себя изменить, после того как была определена в программе, называется **мутабельной**. Соответственно, ту сущность которую изменить нельзя, называют **иммутабельной**.  
Но, несмотря на «утиную типизацию», встроенные типы данных в питоне все же есть. Ниже я привел типы данных Python, для некоторых из которых будут доступны ссылки на мой GitHub, где я реализовывал их в целях лучшего понимания.  

**Числовой тип (иммутабельный)**.

- **int.** – целые числа (числа без дробной части). По сути неограниченной длины. Интерпретатор сам определит необходимость использования объектов неограниченной длины и сделает нужные для этого преобразования.  
- **float** – числа с плавающей точкой с точностью до 15 десятичных знаков. В памяти представлены как совокупность двух частей: *мантиссы* и *экспоненты* (n, e). Мантисса – дробная часть без учета порядка. Экспоненту можно рассматривать как количество цифр перед запятой, отделяющей дробную часть числа. Например: `1.2 E3 = 1.2 * 1000 = 1200` (1.2 – мантисса E3 - экспонента)  
- **complex** – комплексные числа. (состоящие из вещественной и мнимой части как из двух координат). Показательной особенностью комплексных чисел является возможность извлечения корня из отрицательных значений. В Python комплексные числа задаются с помощью функции `complex()`:
```python
z = complex(1, 2)
print(z)
> (1+2j)
```  

Еще есть «лонги»:  

- **long** – содержит длинные целые числа, которые могут быть представлены в восьмеричной или шестнадцатеричной системе счисления 
(существует в Python 2.x. В Python 3.x данный функционал взяли на себя `int`).


**Логический тип данных (иммутабельный)**.  

- **bool** – True/False (1/0). Тип данных, который принимает 2 значения – истина или ложь. `True` и `False` в Python являются экземплярами класса `bool`, который в свою очередь является подклассом `int`. Поэтому `True` и `False` в Python ведут себя как числа `1` и `0`. Отличие только в том, как они выводятся на экран.  

**Строки (иммутабельны)**.  

- **str** - cтроки в Python являются экземплярами класса `str`. Строка представляет собой последовательность (массив) символов Юникода. Строки неизменяемы.
К элементам строки можно получать доступ по их индексам, как в массиве, а также использовать срезы.  

**Двоичные последовательности**:  

- **bytes (иммутабельны)** – по сути это те же строки, только в двоичном представлении (целое число от 0 до 255), `bytes` можно получить из `str`, если известна кодировка.
- **bytearray (мутабельны)** - массив байт. От типа `bytes` отличается только тем, что является изменяемым.  

**Кортежи (иммутабельны)**.  

- **tuple**. Кортеж — это упорядоченная, неизменяемая коллекция объектов произвольных типов. Кортеж является иммутабельным объектом в Python. После определения кортежа, его нельзя изменить. Т.е. кортеж не позволяет как-либо модифицировать свои элементы, обращаясь к ним напрямую как к элементам коллекции. Кроме того, иммутабельность кортежей дает некотрую выгоду при использовании памяти, т.к. кортеж не может изменяться динамически, в отличие от списков. Однако, не стоит забывать, что в питоне ссылочная модель данных, и если сама структура `tuple` не изменяема, то это не значит, что не может изменяться тот объект на который указывает ссылка из кортежа.

**Cписки (мутабельны)**.  
[`Мой пример реализации на Python как связный список`](https://github.com/optima740/Data_structures-Python-/blob/master/level_1/My_new_list.py)  
[`Мой пример реализации на Python как динамический массив`](https://github.com/optima740/Data_structures-Python-/blob/master/level_1/My_dyn_array.py)

- **list**. Cписок можно описать, как упорядоченную и изменяемую коллекцию (набор), состоящую из объектов произвольных типов. Несмторя на свое название (которое подразумевает использование двухсвязного списка в основе структуры), в реализации `CPython`, список – это динамический массив ссылок на объекты (PyObject), размер которого может быть изменен (реалоцирован в памяти). В отдельном модуле питона есть структура данных, которая реализована внутри именно как двухсвязный список (где каждый элемент указывает на соседа, и перемещаться по таким указателям можно в обе стороны). Это `queue` - очередь.  

**Словари (мутабельны)**  
[`Мой пример реализации на Python`](https://github.com/optima740/Data_structures-Python-/blob/master/level_1/NativeArray.py)       

- **dict**. Словарь или ассоциативный массив - представляет собой неупорядоченный набор (не последовательность) пар `«ключ»: «значение»`. Если просто описать суть данной структуры данных (не вникая в нюансы коллизий и тд), то она заключается в том, что по значению «ключа» высчитывается хэш, который служит, своего рода индексом для «значения» (структура хэш-таблицы).  
Т.к. хэши CPU считает очень быстро, то и доступ к элементам словаря практически мгновенный. Кроме того, возможность посчитать хэш есть только для иммутабельных объктов. Потому, ключами для словаря могут стать только неизменяемые объекты, и ключи всегда уникальны. Но само значение хэша от ключа может быть и не уникальным, что приводит к проблеме коллизии и, соответственно, к алгоритмам минимизации данной проблемы.  

**Множества**.  
[`Мой пример реализации на Python`](https://github.com/optima740/Data_structures-Python-/blob/master/level_1/My_Set-new.py)

- **set (мутабельны)**. Набор (не последовательность) уникальных элементов. Элементами множества могут быть любые неизменяемые объекты. Поскольку множество основано на хэш-таблице, элементы множества должны быть хешируемыми, то есть функция `hash()` должна работать с ними, а это возможно только с иммутабельными объектами. Для множества доступны основные математические операции (из теории множеств): объединение, пересечение, разность, симметрическая разность.  
Одними из часто используемых "полезностей" множества является, например, то что можно быстро и эффективно определить входит ли элемент(или несколько элементов) в состав набора. Или отсеять из коллекции (например, списка) повторяющиеся элементы, оставивив из них только уникальные.  
Кроме того, множества позволяют использовать себя как мини-БД, при помощи поддержки операций из теории множеств (а SQL это как раз и есть отсылка к реляционной алгебре), мы можем сделать простые элементарные аналоги запросов к БД, используя вместо этого встроенный тип данных `set`.

- **frozenset (иммутабельны)**. Отличаются от обычного `set` лишь тем, что является неизменяемым типом данных.  

Это был мой краткий обзор основных "коробочных" типов данных в Python. Для ясности и простоты восприятия я решил отобразить все в графическом виде:  

![](/image/post-2021-06-29/python_type_table.jpg)  


Ну и еще можно конечно сюда отнести Классы (**class**). Ведь класс - это, по сути, возможность создавать пользовательский тип данных, и в питоне класс также является объектом (который реализован, если утрированно сказать, как словарь методов и атрибутов, обернутый в синтаксический сахар), порожденным от мета-класса, но об этом в отдельной статье :)  






 




















