# Создание объектов в стиле GObject

Как указывалось ранее, Vala поддерживает альтернативный способ создания объектов, в значительной степени отличающийся от того, что описывалось до этого, но очень похожий на работу по типу GObject. Предпочтения какой использовать зависят от того с чем вы до этого работали,с GObject или Java и C\#. Стиль gobject имеет новые синтаксические элементы, специальный вызов Object\(...\) и блок конструктора. Давайте посмотрим, как это работает:

```csharp
public class Person : Object {

    /* Свойства конструктора */
    public string name { get; construct; }
    public int age { get; construct set; }

    public Person(string name) {
        Object(name: name);
    }

    public Person.with_age(string name, int years) {
        Object(name: name, age: years);
    }

    construct {
        // сделать что-нибудь еще
        stdout.printf("Welcome %s\n", this.name);
    }
}
```

В конструкторе в стиле gobject содержится только вызов `Object(...)` для установки свойств конструктора. `Object(...)` принимает переменное число аргументов в виде пар свойство:значение. Эти свойства должны объявлятся как свойства конструктора. Переданные значения будут присвоены в соответствующие поля и затем по иерархической цепочке `GLib.Object` будут вызваны все блоки construct {}.

Блок construct выполняется гарантированно, даже если он описан в подклассе. Он не имеет параметров и возвращаемого значения. Внутри блока вы можете вызывать другие методы и устанавливать значения полей.

Свойства сonstruct определяются как обычные свойства с `get` и `set`, поэтому в них можно встраивать собственный код. Если вам нужна инициализация, основанная на только на одном свойстве констракта, то возможно написать собственный констракт блок для данного свойства, который будет выполняться непосредственно при присвоении значения, до какого-либо другого кода внутри блока констракт.

Если свойство констракта объявлено без сета, то это ему значение можно присвоить только при инициализации, и больше никогда. В примере выше name как раз является таким свойством.

Вот перечисление различных типов свойств с номенклатурой, которую обычно можно встретить в библиотеках, основанных на `gobject`.

```csharp
    public int a { get; private set; }    // Чтение
    public int b { private get; set; }    // Запись
    public int c { get; set; }            // Чтение / Запись
    public int d { get; set construct; }  // Чтение / Запись / Construct
    public int e { get; construct; }      // Read / Чтение / Запись только в construct
```



In some cases you may also want to perform some action - not when instances of a class is created - but when the class itself is created by the GObject runtime. In GObject terminology we are talking about a snippet of code run inside the class\_init function for the class in question. In Java this is known as _static initializer blocks_. In Vala this looks like:

```csharp
    /* This snippet of code is run when the class
     * is registered with the type system */
    static construct {
      ...
    }
```

