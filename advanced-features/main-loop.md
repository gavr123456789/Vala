# Главный цикл\(The Main Loop\)

GLib включает систему для работы с циклом сообщений через MainLoop. Цель этой системы позволить создавать приложения, которые ждут события и отвечают на них вместо постоянной проверки состояния. Эта модель используется и в GTK+, поэтому программа может дожидаться воздействия пользователя не выполняя при этом никакого кода.

Следующая программа создает и запускает MainLoop и затем подключает к нему источник событий. В данном случае источник - это простой таймер, который будет выполнять данный ему код по прошествии 2000мс. На самом деле метод будет просто останавливать главный цикл, что в данном случае завершит программу.

```csharp
void main() {

    var loop = new MainLoop();
    var time = new TimeoutSource(2000);

    time.set_callback(() => {
        stdout.printf("Time!\n");
        loop.quit();
        return false;
    });

    time.attach(loop.get_context());

    loop.run();
}
```

При использовании GTK+ главный цикл создается автоматически и запускается, когда вызывается метод Gtk.main\(\). Он обозначает точку, когда программа готова запуститься и начать принимать сообщения от пользователя или откуда-то еще. Код на GTK+ эквивалентен примеру выше, поэтому вы можете добавлять источники событий схожим образом. Однако для управления главным циклом вы должны использовать методы GTK+.

```csharp
void main(string[] args) {

    Gtk.init(ref args);
    var time = new TimeoutSource(2000);

    time.set_callback(() => {
        stdout.printf("Время!\n");
        Gtk.main_quit();
        return false;
    });

    time.attach(null);

    Gtk.main();
}
```

Общим требованием для GUI приложений является выполнять код как можно быстрее, но не прерывая работы пользователя. Для этого используются объекты типа IdleSource. Они посылают события в главный цикл программы, однако будут это делать только тогда, когда нет более важных задач.

Для более подробной информации о циклах сообщений смотрите документацию по GLib и GTK+.

