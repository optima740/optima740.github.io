﻿---
layout: post
title: Вселенная программирования. Глобальный взгляд на проектирование.
category: universe
---
![](/image/post-2021-01-10/1.png)  

Любая программная система характеризуется двумя ключевыми характеристиками:  

- **Сложность.**
- **Хаотичность.**  

*Сложность* определяется количеством базовых сущностей и связей между ними в системе (например, классы в проекте). Соответственно, чем их меньше, тем проще проект.  

*Хаотичность* говорит нам о том, насколько не детерминирована (вспоминаем [первую статью](http://127.0.0.1:4000/universe/2020/12/25/programming-universe1/) данного цикла :) система в целом. Насколько она непредсказуема, если говорить иначе. 

Каждый язык программирования создавался для решения определенной сферы задач. Но для любого множества задач можно найти такую парадигму программирования, в которой эти задачи будут решаться продуктивнее и эффективнее. Однако, стоит помнить, что не существует универсальной парадигмы для решения всех классов задач.  
При проектировании программной системы, выбор парадигмы является выбором той математической системы или системы логических принципов, содержащей концепции программирования, которые более точно охватили бы тот круг задач, который необходимо решить. В крупных проектах возможны применения нескольких парадигм, каждая из которых решала бы поставленные задачи на своем уровне (слое).  
Например, **объектно-ориентированное программирование** лучше всего подходит для создания систем, где подразумевается большое число взаимосвязанных абстракций данных, организованных в иерархии.   
**Логическое программирование** лучше всего подходит для анализа и преобразования сложных символических структур в соответствии с наборами логических правил.    
**Дискретное синхронное программирование** лучше всего подходит для "реактивных" задач, когда в системе постоянно происходят реакции на последовательности внешних событий, которые (реакции) после некоторых вычислений генерируют выходные события.    
Популярное сегодня **функционального реактивного программирования** отличается от **FRP** тем, что тут "время" в системе прерывно (шаг времени -- это, как правило, произвольный период между двумя входными событиями), в отличие от **FRP**, где время непрерывно. По этой причине **FRP** также называют **непрерывным синхронным программированием**.  

Большие и сложные системы, подсистемы которых практически всегда реализуются в разных парадигмах (например, классическая система, состоящая из базы данных, бизнес-логики, сетевого движка и клиентской части), могут быть представлены в виде таких слоёв, как, например, реляционно-логическое программирование, последовательное и параллельное ООП, функциональное реактивное программирование. Уже исходя из такой архитектуры, можно сознательно подбирать наиболее подходящие фреймворки, отбирая их по формальным критериям совместимости фундаментальных концепций, и получить существенный выигрыш в сравнении с несознательным механическим выбором популярного стека.  

![](/image/post-2021-01-10/pic1.jpg)  

Многие языки программирования поддерживают несколько парадигм, которые можно разделить на две группы: 

- Парадигмы, поддерживающие **programming in small**. Непосредственное кодирование для класса задач, наиболее часто решаемых данным языком.
- Парадигмы, поддерживающие **programming in large**. Проектирование и создание больших программных систем с поддержкой абстракции и модульности.  

Таким образом, мы видим, что при правильном проектировании сложных программных систем и проектов, конкретные задачи лучше всего выделять более выразительно, чтобы можно было подобрать более продуктивные языки и фреймворки, точно отвечающие наиболее подходящей парадигме для решения выделенного класса задач. И в результате, получить выигрыш в трудоёмкости в сотни, а то и тысячи раз. Но для этого надо научиться правильно смотреть на слои архитектуры по-научному - глобально, через спектр всех доступных парадигм программирования.







 






