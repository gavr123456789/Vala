# Конструктор

Vala поддерживает две сильно отличающиеся схемы конструктора: схема в стиле Java/C\#, которую мы рассморим сейчас и в стиле GObject, которая будет рассмотрена в конце данной главы.

В Vala не поддерживается перегрузка конструктора по той же причине, что и в случае перегрузки методов, что значит класс не может иметь несколько конструкторов с одинаковыми названиями. Однако это не проблема, т.к. Vala поддерживает именованные конструкторы. Если есть необходимость в нескольких конструкторах, вы можете сделать добавления в их именах:

```csharp
public class Button : Object {

    public Button() {
    }

    public Button.with_label(string label) {
    }

    public Button.from_stock(string stock_id) {
    }
}
```

Экземпляры создаются аналогично:

```csharp
new Button();
new Button.with_label("Нажми меня");
new Button.from_stock(Gtk.STOCK_OK);
```

Вы можете собрать конструктор используя this\(\), или this.name\_extension\(\), например:

```csharp
public class Point : Object {
    public double x;
    public double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public Point.rectangular(double x, double y) {
        this(x, y);
    }

    public Point.polar(double radius, double angle) {
        this.rectangular(radius * Math.cos(angle), radius * Math.sin(angle));
    }
}

void main() {
    var p1 = new Point.rectangular(5.7, 1.2);
    var p2 = new Point.polar(5.7, 1.2);
}
```

