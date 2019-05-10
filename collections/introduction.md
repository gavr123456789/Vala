# Введение

[Gee](https://wiki.gnome.org/Projects/Libgee) - это библиотека классов коллекций, написанная на Vala. Эти классы должны быть знакомы любому, кто пользовался библиотеками типа Базовые Классы Java \(Java Foundation Classes\). Gee содержит интерфейсы и различные типы, которые по разному реализуют эти интерфейсы по-разному.

Если вы хотите использовать Gee, то придется ее устанавливать отдельно. Gee можно получить по [http://live.gnome.org/Libgee](http://live.gnome.org/Libgee). При этом ваше приложение нужно компилировать с ключом `--pkg gee-1.0`.

Основные типы коллекций следующие:

* Списки: упорядоченные коллекции элементов, доступные по индексу.
* Set: неупорядоченные коллекции не повторяющихся элементов.
* Map: неупорядоченная коллекция элементов, доступные по индексу определенного типа.

Все списки и последовательности реализуют интерфейс Collection, а все отображения - интерфейс **Map**. Списки так же реализуют **List** и последовательности **Set**. Это значит не только что все похожие коллекции реализуют одинаковый интерфейс и могут быть легка заменены между собой, но и то, что новые коллекции могут быть написаны с реализацией этих интерфейсов и использованы в существующем коде.

Так же каждый _Collection_ реализует интерфейс _Iterable_. Это значит, что любой объект из этой категории может быть итерирован стандартными методами или через `foreach`.

Все классы и интерфейсы используют обобщения. Это значит, что они должны быть созданы на основе определенного типа или группы типов, который будут содержать. Система будет проверять, что объекты только указанного типа помещаются в коллекцию, и что при получении из коллекции возвращаются объекты правильного типа.

Полная документация по Gee API, примеры Gee.

Некоторые важные классы Gee:  

