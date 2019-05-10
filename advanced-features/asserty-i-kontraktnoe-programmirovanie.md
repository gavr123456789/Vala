# Ассерты и контрактное программирование

С ассертами программист может проверять допущения во время работы программы. Синтакс при этом - assert\(условие\). Если ассерт не верен, то выполнение программы прекратиться с соответствующим сообщением об ошибке. Существует еще несколько видов ассертов в стандартном пространстве имен GLib:

| assert\_not\_reached\(\) |
| :--- |
| return\_if\_fail\(bool expr\) |
| return\_if\_reached\(\) |
| warn\_if\_fail\(bool expr\) |
| warn\_if\_reached\(\) |

Вы можете попытаться использовать ассерты для проверки аргументов метода на null. Однако это вовсе не обязательно, т.к. Vala неявно проверяет все методы, которые не помечены ?.

```csharp
void method_name(Foo foo, Bar bar) {
    /* Не обязательно, Vala делает это за вас:
    return_if_fail(foo != null);
    return_if_fail(bar != null);
    */
}
```

Vala поддерживает базовые возможности контрактного программирования. Метод может имет предусловия\(требования\) и постусловия \(удостоверение\), которые должны пройти до и после выполнения метода соответственно:

```csharp
double method_name(int x, double d)
        requires (x > 0 && x < 10)
        requires (d >= 0.0 && d <= 1.0)
        ensures (result >= 0.0 && result <= 10.0)
{
    return d * x;
}
```

`результатом` является специальная переменная, представляющая возвращаемое значение.  

