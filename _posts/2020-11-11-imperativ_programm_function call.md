﻿---
layout: post
title: Императивное программирование. Вызов функций, нити исполнения кода.
---
![](/image/post-2020-11-11/logo.JPG)

В данной статье я хотел бы рассмотреть процесс вызова подпрограмм (функций) в императивной парадигме программирования, на примере изучаемого мной Питона. 
А также немного затронуть основную суть синхронного (классического) и асинхронного вызова функций.  

В 1946 г Джон фон Нейман, Герман Голдстайн и Артур Беркс в своей совместной статье изложили принципы построения и функционирования ЭВМ. Впоследствии на основе этих принципов производились первые два поколения компьютеров. 
В более поздних поколениях происходили некоторые изменения, хотя принципы Неймана актуальны и сегодня.  
По одному из указанных принципов фон Неймана программа состоит из набора команд, которые выполняются процессором друг за другом в определенной последовательности. Выполнение инструкций (или программного кода) применительно к императивной парадигме программирования происходит последовательно или сверху вниз по мере чтения кода. 
Имея в своем арсенале такие элементарные конструкции как переменные, ветвления и циклы можно запрограммировать (описать на языке программирования) практически любую задачу.  

В ходе написания программы мы неизбежно сталкиваемся с ситуацией многократного повторного использования одного и того же кода. Эти повторяющиеся участки программного кода, согласно принципу **DRY** - Don’t repeat yourself (не повторяй себя), принято локализовать и выделять в отдельные подпрограммы (или функции).  

Давайте рассмотрим процесс вызова функций из главной программы. Как я уже ранее отмечал в своей [статье](https://optima740.github.io/2020/02/17/go-to-stage-down/) , при запуске программы, операционная система генерирует процесс. Процесс, в свою очередь, получает свой собственный набор (пространство) адресов памяти. Это такой способ разграничить ресурсы одной программы от другой, чтобы избежать использование памяти одного процесса другим: все адреса памяти, к которым процесс может обращаться принадлежат только ему.  

Пространство памяти внутри процесса, если описать очень упрощенно, делится на несколько сегментов:  

- **Сегмент данных (обрабатываемых значений).**
- **Сегмент команд (кода).**
- **Стек вызова.**

Сегмент данных - это динамическая память (куча - heap), в которой содержатся данные (объекты) для вычисления в коде.  


![](/image/post-2020-11-11/steck.jpg)

На схеме выше, я постарался отобразить ход выполнения программы, в которой используется вызов функций. Весь исполняемый код располагается в сегменте команд. 
При выполнении главной программы **`__main__`** интерпретатор языка доходит до точки вызова функции **`foo()`**, после чего, в стек вызова заносится адрес возврата. Это та точка в основной программе, куда будет возвращено управление после выполнения функции **`foo()`** посредствам оператора **`return`**. Формально, если коснуться более низкого уровня – ассемблера, после выполнения любой функции происходит вызов инструкции **`RET`**, которая уже использует стек с адресами возврата для передачи управления обратно в точку первоначального вызова. Точно также происходит и в случае еще одной вложенной функции **`bar()`**. После ее вызова в стек добавляется адрес возврата в функцию **`foo()`**. И, как только выполнится **`bar()`**, ее **`return`** вернет ход программы в **`foo()`**, используя адрес возврата из стека. А затем, после выполнения **`foo()`**, вытащит из стека самый нижний элемент – адрес возврата в **`__main__`**. 

И вот тут мы плавно подошли к теме нитей выполнения. Дело в том, что во время выполнения всех вложенных функций главная программа **`__main__`** , как-бы, ожидает пока выполнится **`foo()`**. А **`foo()`**, в свою очередь, ожидает выполнения **`bar()`**. Такая модель вызовов функций называется синхронными вызовами (или классическими вызовами).

![](/image/post-2020-11-11/sinchronic.jpg)

Существует еще одна модель выполнения кода – асинхронная. В этом случае вызванная функция выполняется параллельно основной программе. Но, в таком случае, сразу встает вопрос о том, в какой момент, и в какую точку необходимо вернуть значение вызванной функции **`foo()`** в параллельно выполняющуюся **`__main__`**. И что делать, если **`foo()`** выполнилась раньше или позже от ожидаемого момента в основной программе.  
Но асинхронная модель потому и названа таковой, что обе нити периодически обмениваются состоянием своего выполнения. И одна из нитей имеет возможность подстроиться под ход выполнения другой, ожидая ее в нужных моментах.  
На схеме ниже я постарался привести такой пример выполнения.  

![](/image/post-2020-11-11/asinchronic.jpg)

Таким образом, я постарался наглядно описать (пусть и в очень упрощенной форме), ту механику, которая происходит при вызове функций, или в классическом понимании – подпрограмм.

[`В статье использовались материалы из курса лекций преподавателя кафедры информатики МФТИ Тимофея Хирьянова.`](https://www.youtube.com/c/%D0%A2%D0%B8%D0%BC%D0%BE%D1%84%D0%B5%D0%B9%D0%A5%D0%B8%D1%80%D1%8C%D1%8F%D0%BD%D0%BE%D0%B2/featured)  





 





