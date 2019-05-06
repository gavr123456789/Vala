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

 Чтобы определить массив структур, смотрите [FAQ](https://wiki.gnome.org/Projects/Vala/Tutorial/FAQ#How_do_I_create_an_array_of_structs.3F)

