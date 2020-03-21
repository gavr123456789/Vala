# For Big Boys

Я решил создать этот раздел для тех кто уже хорошо знаком с основными понятиями программирования вроде ООП, лямбд, делегатов и тд. Здесь будут описаны только ключевые особенности Vala в отличии от других языком.  


## Оссобенности синтаксиса

?? null-сливающий оператор: a ?? b  эквивалентно  a != null ? a : b. Этот оператор особенно полезен, например, для предоставление дефолтного значения в том случае, если ссылка равна _null_:

```csharp
print("Hello, %s!\n", name ?? "unknown person");
```

PR ПО ДОБАВЛЕНИЮ ОПЕРАТОРОВ .? !in ...

Модификаторы для ссылок unowned/owned, weak/strong \(подробнее будет ниже\)

### Массивы

Для стандартных массивов в Vala генерируются следующие методы

СИ МЕТОДЫ

Как можно заметить вы можете добавлять элементы динамически с помощью +=. Однако, это работает только для локально определенных или private массивов. Массив будет автоматически переразмещен, если потребуется. Внутренне, это переразмещение происходит, когда размер массива переваливает через еще одну степень двойки, это сделано по причинам эффективности.  .length содержит реальное число элементов массива, а не внутренний размер.

```csharp
int[] e = {};
e += 12;
e += 5;
e += 37;
```

#### Слайсы как в Python

```csharp
int[] b = { 2, 4, 6, 8 };
int[] c = b[1:3]; // => { 4, 6 }
```

Результатом обрезания массива будет ссылка на запрашиваемые данные, а не копия. Однако, присвоение обрезания owned-переменной \(как показано ниже\) будет приводить к копированию. Если вы хотите избежать копирования, вы обязаны либо присвоить срез какой-нибудь unowned-переменной или передать его прямо как аргумент \(аргументы по умолчанию unowned\):

```csharp
unowned int[] c = b[1:3];     // => { 4, 6 } 
```

#### Методы

Vala хранит длинну для всех массивов отдельной переменной\(если вы не укажете обратного с помощью атрибутов кода\) поэтому у каждого массива\(и строки\) есть свойство `length`

Для получения длинны в многомерных массивах используйте следующий синтаксис

```csharp
int[,] arr = new int[4,5];
int r = arr.length[0];
int c = arr.length[1];
```

ТУТ ПРИМЕР СГЕНЕРИРОВАННОГО С КОДА  
ЕЩЕ ПРИМЕР С АТРИБУТОМ ДЛЯ БЕЗ ДЛИННЫ



resize\(int\) - ресайзит

```csharp
int[] a = new int[5];
a.resize(12);
```

move\(_src, dest, lenght\)_ мувит

```csharp
uint8[] chars = "hello world".data;
chars.move(6,0,5);
print((string) chars); //"world "
```

Если вы поставите квадратные скобки после идентификатора вместе с указанием размера, вы получите массив фиксированного размера. Массивы фиксированного размера размещаются в стеке \(если используются в качестве локальных переменных\) или размещаются в строке \(если используются в качестве полей\), и вы не сможете перераспределить их позже.

```text
int f[10];
```

Vala не выполняет никаких проверок границ для доступа к массиву во время выполнения. Если вам нужно больше безопасности, вы должны использовать более сложную структуру данных, такую как [ArrayList](https://wiki.gnome.org/ArrayList). Вы узнаете больше об этом позже в разделе о коллекциях.

Однако массивы в Vala все же более безопасны чем в си, изза некоторых хитростей кодогенерации:

{% embed url="https://twitter.com/gavr123456789/status/1240625085448359937" %}

Оператор in проверяет, содержит ли правый операнд левый. Этот оператор работает на массивах, строках, коллекциях и любых других типах, которые имеют соответствующий _contains\(\)_ метод. Для строк он выполняет поиск подстроки.

## Компилятор

### Vala scripting

Вопервых, как уже упоминалось в предисловии вы можете использовать Vala в качестве скриптового языка. Сделав следующий файл исполнительным, это будет работать. 

```csharp
#!/usr/bin/vala
var a = 5;
print(@"a = $a\n");
print(Environment.get_current_dir() + "\n");
print(Environment.get_host_name() + "\n");
```

```csharp
❯❯❯ ./v.vala
a = 5
/home/gavr/bin/temp
arch
```

### Debug

Компилируя с -g --save-temp Vala подставит в Си код прагмы позволяющие GDB распознавать каким Vala строкам какие Си строки соответствуют.

Таким образом Vala можно замечательно дебажить любыми средствами.   
Видео демка:

{% embed url="https://youtu.be/3yyDdA5IMLI" %}

Пример Taskа для VSc под видео.

### Компиляция

{% hint style="info" %}
Только самые важные/интересные, я не буду описывать флаги компиляции shared библиотек и gresource потому что это за вас делает Meson
{% endhint %}

* Флаг -C создаст С файл вместо бинарника, многие флаги специфичные для бинарников при этом не работают \(логично\)
* -H или --header=FILE создаст хедер, если вы хотите слинковаться с либой\(хотя опять таки, лучше предоставить это дело Meson\)
* -X -ANY\_C\_FLAG — передача флага в компилятор С, например `-X -O3`
* -k --keep-going Продолжать анализ файла, даже если были встречены ошибки\(на случай если вы хотите увидеть все ошибки, и почему то не используете Langue Server, вообще для него флаг и предназначен\)
* --pkg PKGNAME — использовать пакет через pgk-config, пример `vala v.vala --pkg gee-0.8`
* --abi-stability — ?
* --enable-experimental-non-null — об этом позже, в разделе nullabylity
* --fatal-warnings — вылетать при warning, подробнее в разделе про логирование
* --cc COMMAND — использовать команду в качестве компилятора С, например valac --cc tcc v.vala ускорит компиляцию в ~2 раза, хороший вариант для циклов разработки

## ARC и GC

{% embed url="https://oxozle.com/2017/05/10/sravnenie-arc-i-garbage-collector/" %}



## 5 типа ссылок

{% embed url="https://wiki.gnome.org/Projects/Vala/Ownership" %}

### Owned/Unowned

### Weak/Strong

### Pure

Pointers are Vala's way of allowing manual memory management. Normally when you create an instance of a type you receive a reference to it, and Vala will take care of destroying the instance when there are no more references left to it. By requesting instead a pointer to an instance, you take responsibility for destroying the instance when it is no longer wanted, and therefore get greater control over how much memory is used.

This functionality is not necessarily needed most of the time, as modern computers are usually fast enough to handle reference counting and have enough memory that small inefficiencies are not important. The times when you might resort to manual memory management are:

* When you specifically want to optimise part of a program.
* When you are dealing with an external library that does not implement reference counting for memory management \(probably meaning one not based on gobject.\)

In order to create an instance of a type, and receive a pointer to it:

```text
Object* o = new Object();
```

In order to access members of that instance:

```text
o->method_1();
o->data_1;
```

In order to free the memory pointed to:

```text
delete o;
```

Vala also supports the _address-of_ \(&\) and _indirection_ \(\*\) operators known from C:

```text
int i = 42;
int* i_ptr = &i;    // address-of
int j = *i_ptr;     // indirection

```

The behavior is a bit different with reference types, you can omit the address-of and indirection operator on assignment:

```text
Foo f = new Foo();
Foo* f_ptr = f;    // address-of
Foo g = f_ptr;     // indirection

unowned Foo f_weak = f;  // equivalent to the second line

```

The usage of reference-type pointers is equivalent to the use of unowned references.



References and ownership \([https://wiki.gnome.org/Projects/Vala/Manual/Concepts\#References\_and\_ownership](https://wiki.gnome.org/Projects/Vala/Manual/Concepts#References_and_ownership)\)

Ownership transfer expressions \([https://wiki.gnome.org/Projects/Vala/Manual/Expressions\#Ownership\_transfer\_expressions](https://wiki.gnome.org/Projects/Vala/Manual/Expressions#Ownership_transfer_expressions)\)

Перевести, обновить \(инфа из 2013\)

| feature | status |
| :--- | :--- |
| Handling Ownership in simple collections | no |
| Panic for weak reference for referable objects | yes |
| Weak reference for InitiallyUnowned | not tested yet |
| References as entities | state unknown |
| Casting strong to weak | yes |
| Casting strong to weak ownership transferring at formal parameter | yes |
| Implicit strong keyword | yes, but not purposely |
| Implicit ownership keyword | yes, but not purposely |
| Explicit strong keyword | no |
| Explicit ownership keyword | no |
| Strict ownership transferring grammar | no |
| Weak reference as entity type in generics | yes, but not purposely |
| Panic for strong reference as entity type in generics | N/A because no explicit strong keyword, they are inferred to be ownerships |

## 3 вида классов

{% embed url="https://stackoverflow.com/questions/33003942/vala-different-type-of-constructors" %}

{% embed url="https://vala.gitbook.io/vala/examples/basic\#gobject" %}



## Строки

string = Си строки \(это сделано для более простой совместимости, принять из Си char\* все равно что string\)  
StringBuilder = GString

По дефолту UTF-8 из-за чего есть особенности [https://wiki.gnome.org/Projects/Vala/StringSample](https://wiki.gnome.org/Projects/Vala/StringSample)

## Атрибуты кода

{% embed url="https://wiki.gnome.org/Projects/Vala/Manual/Attributes" %}

Выделить самые полезные

## GIR

{% embed url="https://gitlab.com/gavr123456789/call-vala-from-all-languages" %}

{% embed url="https://gi.readthedocs.io/en/latest/" %}

![](.gitbook/assets/image%20%286%29.png)

## Nullability



## How to do X in Vala

