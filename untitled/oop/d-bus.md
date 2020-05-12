# Интеграция с D-Bus

D-Bus тесно интегрирован в язык и его использование в Vala как никогда просто.

Для того, чтобы экспортировать класс как сервис D-Bus нужно просто написать аннотацию с атрибутами D-Bus и зарегистрировать экземпляр этого класса с локальной сессией D-Bus.

```csharp
[DBus(name = "org.example.DemoService")]
public class DemoService : Object {
    /* Private поле, не экспортируется через D-Bus */
    int counter;

    /* Public поле, не экспортируется через D-Bus */
    public int status;

    /* Public свойство, экспортируется через D-Bus */
    public int something { get; set; }

    /* Public сигнал, экспортируется через D-Bus
     * Может быть выброшен на стороне сервера и присоединен на стороне клиента.
     */
    public signal void sig1();

    /* Public метод, экспортируется через D-Bus */
    public void some_method() {
        counter++;
        stdout.printf("Эврика! counter = %d\n", counter);
        sig1();  // генерация сигнала
    }

    /* Public method, exported via D-Bus and showing the sender who is
       is calling the method (not exported in the D-Bus interface) */
    public void some_method_sender(string message, GLib.BusName sender) {
        counter++;
        stdout.printf("Эврика! counter = %d, '%s' message from sender %s\n",
                      counter, message, sender);
    }
}
```

Регистрируем сервис и начинаем главный цикл:

```csharp
void on_bus_aquired (DBusConnection conn) {
    try {
        // запуск сервиса и регистрация его как dbus объекта
        var service = new DemoService();
        conn.register_object ("/org/example/demo", service);
    } catch (IOError e) {
        stderr.printf ("Could not register service: %s\n", e.message);
    }
}

void main () {
    // пробуем зарегистрировать сервис в сессии
    Bus.own_name (BusType.SESSION, "org.example.DemoService", /* name to register */
                  BusNameOwnerFlags.NONE, /* flags */
                  on_bus_aquired, /* callback function on registration succeeded */
                  () => {}, /* callback on name register succeeded */
                  () => stderr.printf ("Could not acquire name\n"));
                                                     /* callback on name lost */

    // запуск главного цикла
    new MainLoop ().run ();
}
```

Компилировать нужно с пакетом dbus-glib-1:

```text
$ valac --pkg gio-2.0 dbus-demo-service.vala
$ ./dbus-demo-service
```

Все названия в Vala, имеющие вид lower\_case\_with\_underscores автоматически транслируются к виду CamelCase как принято в D-Bus. Экспортируемый D-Bus интерфейс указанного примера будет иметь свойство Something, сигнал Sig1 и метод SomeMethod. Вы можете открыть новое окно терминала и вызвать метод командой:

```text
$ dbus-send --type=method_call                   \
            --dest=org.example.DemoService       \
            /org/example/demo                    \
            org.example.DemoService.SomeMethod
```

or

```text
$ dbus-send --type=method_call                   \
            --dest=org.example.DemoService       \
            /org/example/demo                    \
            org.example.DemoService.SomeMethodSender \
            string:'hello world'
```

Вы можете так же использовать графический отладчик D-Bus типа [D-Feet](https://wiki.gnome.org/Apps/DFeet), чтобы смотреть интерфейс D-Bus и вызываемые методы.

Более продвинутые примеры:  
[Vala/DBusClientSamples](https://wiki.gnome.org/Projects/Vala/DBusClientSamples) и[ Vala/DBusServerSample](https://wiki.gnome.org/Projects/Vala/DBusServerSample)  


