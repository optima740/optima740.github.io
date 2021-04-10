﻿---
layout: post
title: Вселенная программирования. Ключевые концепции ч4 - Темная сторона параллелизма.
---
![](/image/post-2021-01-10/1.png)  

Продолжаем тему концепции [параллелизма](https://optima740.github.io/2021/03/25/programming-universe6/).  
На первый взгляд параллельное программирование выглядит многообещающим и эффективным способом повысить производительность программной системы.   
Но есть один существенный момент в данной концепции, который является и основной проблемой. После того, как в программной системе появились именованные состояния и параллелизм одновременно- это привело к появлению явного (наблюдаемого) **недетерминизма** (вспоминаем [статью](https://optima740.github.io/2021/01/26/programming-universe3/) про вычислительные модели и степень недетерминизма). То есть программа от вызова к вызову может выдавать разные результаты на одних и тех же входных данных. Так случается потому, что потоки могут получать доступ к именованным состояниям (переменным) в непредсказуемом порядке, который зависит от внешних условий. И причина этой изменчивости (недетерминизма) в том, что точное время, когда будет выполнена та или иная инструкция в программе теперь неизвестно потому, что потоки работают независимо и не представляют, какие инструкции выполняются в других потоках. Иначе данную ситуацию в параллельном программировании называют **проблемой конкуренции** или **race condition**.  

Рассмотрим пример кода Python ниже.  

```python
a = 0
# функция 1 потока
def thread_one():
    global a = 1
# функция 2 потока
def thread_two():
    global a = 2
```
В результате такой работы в переменной **`a`** будет либо 1, либо 2, но мы не можем заранее предсказать, какое значение именно. Причина в том, что в реальных условиях работа потоков чередуется, причём в общем случае невозможно понять как.  
Дело в том, что исходный код программы транслируется интерпретатором в двоичный машинный код (если упростить), и механизм обеспечения одновременного выполнения потоков выделяет, так называемый, квант времени поочерёдно каждому потоку. За этот квант времени может, например, выполниться всего одна машинная инструкция, то есть явную привязку квантованного выполнения к исходному коду выполнить фактически невозможно. 
Но, даже если мы будем условно считать, что за один квант выполняется одна инструкция исходного кода, ситуация не проясняется.  

В представленном коде выше, я привел самый простой пример, но в реальных программах, где смешиваются параллельные вычисления и именованные состояния, постоянно возникают значительно более сложные конфликтные ситуации.   
В истории существует печально известный пример ситуации с *race condition* - это канадский аппарат лучевой терапии [*Therac-25*](https://ru.wikipedia.org/wiki/Therac-25), который из-за подобного бага в своем софте выдавал пациентам дозы, в тысячи раз превышающие назначенные, что приводило к смертям и тяжёлым заболеваниям. Одна и та же переменная в этом аппарате использовалась сразу в двух вычислительных задачах, которые могли выполняться одновременно.  

Из всего выше изложенного следует вывод: *по возможности не использовать вместе параллелизм и именованные состояния*. Программу практически всегда можно спроектировать так, чтобы разделить эти аспекты, или, в самом крайнем случае, ограниченно и наглядно совместить их в небольшой и хорошо изолированной части проекта.  
  
Однако, существуют способы организовать программу таким образом, что одновременное использование параллелизма и именованных состояний будет давать корректный результат и в целом такая схема остается вполне применимой. Наверное, одним из самых известных и хороших способов правильно организовать программирование с параллелизмом и состоянием - это использовать **атомарные операции**.  

*Атомарная операция*, как бы, инкапсулирует внутри себя некоторые инструкции кода, и делает их неделимыми во времени. То есть, мы имеем начало атомарной операции и сразу результат, а промежуточные состояния внутри атомарной операции нам недоступны.  
Поэтому одним из способов борьбы с *race conditions* является организация потоково-безопасной (thread-safe) работы программы. Программисту предоставляется механизм атомарного выполнения- набор действий над общей переменной объявляется некоторому потоку атомарным, и пока он полностью не закончится, никакой другой поток не сможет работать с этой переменной.  
С помощью атомарных операций мы можем решить вышеприведённую проблему с неверным итоговым изменением переменной **`a`** в нескольких потоках. Идея заключается в том, чтобы обеспечить такой режим работы, когда тело каждого потока будет атомарным.  
 
Рассмотрим следующий псевдокод:  
```
thread A
a = 0
запуск thread B
начало атомарной операции
a = a + 1
конец атомарной операции
ожидание окончания работы thread B
print a

thread B
начало атомарной операции
a = a + 2
конец атомарной операции
```  
В результате на консоль всегда будет выводиться 3.  

В различных языках программирования атомарные операции представлены по-разному. Например, в Python данные операции можно реализовать с помощью *блокировок* (Lock). 
Класс **`RLock`** – это блокировка (или замок).  
*Блокировка* -  фундаментальный механизм синхронизации, который предоставлен модулем `threading` в Python.    
Замки используются для синхронизации доступа к общим ресурсам. Для каждого такого источника создается объект `Lock`. Когда нам нужно получить доступ к общему ресурсу, мы вызываем `acquire` для того, чтобы поставить блок на время работы потока с данным ресурсом, после чего вызываем `release`, тем самым открывая доступ к общему ресурсу:  

```python
lock = threading.RLock() 
lock.acquire() # Выполнит блокировку данного участка кода
... #доступ к общим ресурсам
lock.release() # Снятие блокировки 
```  

В любом случае во избежание ситуации *race condition*, при организации параллельных вычислений всегда необходимо обеспечивать атомарность операций, насколько это возможно.  
  
В следующей статье мы поговорим о еще одной ключевой концепции программирования – **абстракции данных**.  







