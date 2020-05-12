# Свойства\(Properties\)

Хорошей практикой объектно-ориентированного программирования считается скрывать внутренние детали реализации класса \(принцип сокрытия информации\), поэтому вы можете производить изменения внутри без изменений во внешнем API. Один из способов - это объявить поля приватными и написать методы доступа для получения и изменения значения этих полей \(геттеры и сеттеры\).

Если вы программировали на Java, возможно вы подумали о чем-то вроде этого:

```csharp
class Person : Object {
    private int age = 32;

    public int get_age() {
        return this.age;
    }

    public void set_age(int age) {
        this.age = age;
    }
}
```

Этот код будет скомпилирован, но в Vala можно реализовать это проще. Проблема в том, что эти методы трудоемки. Давайте предположим, что вы хотите увеличить возраст человека на один год:

```csharp
var alice = new Person();
alice.set_age(alice.get_age() + 1);
```

Вот здесь-то свойства и включаются в игру:

```csharp
class Person : Object {
    private int _age = 32;  // подчеркивание перед именем идентификатора
    // используется для того, что бы избежать конфликта имен со свойствами

    /* Свойства */
    public int age {
        get { return _age; }
        set { _age = value; }
    }
}
```

Синтаксис должен быть знаком C\# программистам. Свойство имеет блоки `get` и `set` для получения и установки его значения. `value` является ключевым словом, представляющим новое значение, которое должно быть присвоено свойству.

Теперь вы можете обратиться к свойству, как будто бы оно было public полем, но внутри будет выполняться код из блоков `get` и `set`.

```csharp
var alice = new Person();
alice.age = alice.age + 1;  //  или ещё короче:
alice.age++;
```

Если вам нужны лишь стандартные возможности, то можно записать еще короче:

```csharp
class Person : Object {
    /* Свойство с get и set, и стандартным значением */
    public int age { get; set; default = 32; }
}
```

Со свойствами вы можете поменять реализацию класса не меняя API. Например:

```csharp
static int current_year = 2525;

class Person : Object {
    private int year_of_birth = 2493;

    public int age {
        get { return current_year - year_of_birth; }
        set { year_of_birth = current_year - value; }
    }
}
```

Здесь age вычисляется на лету исходя из года рождения. Заметьте, вы можете делать не просто присваивания внутри блоков `get/set`. Вы можете обратиться к базе данных, сделать запись в лог, обновить кэш и т.д.

Если вы хотите сделать свойство только для чтения, вы должны сеттер сделать `private`:

```csharp
public int age { get; private set; default = 32; }
```

Или вы можете опустить блок `set`:

```csharp
class Person : Object {
    private int _age = 32;

    public int age {
        get { return _age; }
    }
}
```

Свойства могут иметь не только название, но и короткое описание \(называется nick\) и длинное описание \(называется blurb\). Вы можете сделать аннотацию следущим образом:

```csharp
[Description(nick = "Возраст в годах", blurb = "Это возраст человека в годах")]
    public int age { get; set; default = 32; }
```

Свойства и их дополнительные описания можно получить во время работы программы. Некоторые программы, такие как дизайнер пользовательского интерфейса Glade используют такую информацию. Таким образом Glade может показывать понятные человеку описания свойств виджетов GTK+.

Каждый объект класса, наследующего от GLib.Object имеет сигнал notify. Этот сигнал подается всякий раз, когда значение свойства изменяется, поэтому вы можете подключится к этому сигналу, если требуется получать такие уведомления:

```csharp
obj.notify.connect((s, p) => {
    stdout.printf("Свойство '%s' было изменено!\n", p.name);
});
```

`s` служит источником сигнала, `p` - информация об изменившемся свойстве типа `ParamSpec`. Если вам нужно уведомление только об определенных свойствах, используйте следующих синтаксис:

```csharp
alice.notify["age"].connect((s, p) => {
    stdout.printf("возраст изменился\n");
});
```

Заметьте, что здесь нужно использовать строковое представление свойства, где подчеркивания становятся дефисами: my\_property\_name превратится "my-property-name", что соответсвует именованию свойств в GObject.

Изменить напоминания можно с помощью атрибута CCode прямо перед объявлением свойства:

```csharp
public class MyObject : Object {
    [CCode(notify = false)]
    // сигнал уведомления НЕ подается, когда произойдет изменение значения свойства
    public int without_notification { get; set; }
    // сигнал уведомления подается при изменении свойства
    public int with_notification { get; set; }
}
```

Есть еще один тип свойств называемые свойства конструктора. Они описаны далее в разделе о конструкторах в стиле `gobject.`

Note: in case your property is type of struct, to get the property value with Object.get\(\), you have to declare your variable as example below

```csharp
struct Color
{
    public uint32 argb;

    public Color() { argb = 0x12345678; }
}

class Shape: GLib.Object
{
    public Color c { get; set; default = Color(); }
}

int main()
{
    Color? c = null;
    Shape s = new Shape();
    s.get("c", out c);
}
```

This way, c is an reference instead of an instance of Color on stack. What you passed into s.get\(\) is "Color _\*" instead of "Color_ ".

