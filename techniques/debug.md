# Отладка

{% embed url="https://youtu.be/3yyDdA5IMLI" %}



Для демонстрационных целей мы создадим программу с ошибками, в которой преднамеренно разыменовывается null ссылка, что вызовет ошибку сегментирования:

```csharp
class Foo : Object {
    public int field;
}

void main() {
    Foo? foo = null;
    stdout.printf("%d\n", foo.field);
}
```

`Ошибка сегментирования`

Ну, как мы будем отлаживать эту программу? Ключ -g говорит компилятору включить информацию о строках исходников в двоичный файл, --save-temps позволяет оставить временные С файлы:

`$ valac -g --save-temps debug-demo.vala`

Программы написанные в Vala можно отлаживать с помощью GNU Debugger - gdb. Так для gdb существуют различные графические интерфейсы, напр. Nemiver.

`$ nemiver debug-demo`

Примерный сеанс c gdb:

```text
$ gdb debug-demo
(gdb) run
Запуск программы:
/home/valacoder/debug-demo
Программа получает сигнал SIGSEGV,
Ошибка сегментирования.
0x0804881f in _main () at debug-demo.vala:7
7 stdout.printf("%d\n", foo.field);
(gdb)
```

