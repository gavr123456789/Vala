# Структуры

```csharp
struct StructName {
    public int a;
}
```

определяет структурный тип, т. е. составной значимый тип. Структура в Vala может в некоторых случаях иметь методы и закрытые члены, значит надо явно использовать модификатор доступа public.

```csharp
struct Color {
    public double red;
    public double green;
    public double blue;
}
```

Так вы можете инициализировать структуру:

```csharp
// без вывода типов
Color c1 = Color();
Color c2 = { 0.5, 0.5, 1.0 };
Color c3 = Color() {
    red = 0.5,
    green = 0.5,
    blue = 1.0
};

// с выводом типов
var c4 = Color();
var c5 = Color() {
    red = 0.5,
    green = 0.5,
    blue = 1.0
};
```

Структуры - значимые типы \(выделяются в стеке/встраиваются и копируются при присваивании\).

-----------------------------

### SimpleType

Если перед объявлением структуры написать шаблон `[SimpleType]`то данная структура будет передаваться по значению.

![](../.gitbook/assets/image%20%281%29.png)

## Массив структур

Если массив константен:

```csharp
const YourStruct[] s = { { value1, value2, ... }, ...};
```

Иначе:

```text
YourStruct[] s = { YourStruct() { field1=value1, field2=value2, ... }, ...};
```

В качестве альтернативы, если структуру YourStruct создали вы \(она не является внешней\), вы можете предоставить конструктор, чтобы упростить приведенное выше выражение:

```csharp
public struct YourStruct {
  public int field1;
  public string field2;
  ...
  public YourStruct (int field1, string field2, ...) {
    this.field1 = field1;
    this.field2 = field2;
    ...
  }
}
```

Тогда вы сможете инициализировать массив структур так:

```csharp
YourStruct[] s = { YourStruct (field1, field2, ...), ...};
```

