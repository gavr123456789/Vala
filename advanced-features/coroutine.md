# Асинхронные методы

Асинхронные методы позволяют программировать вообще без блокировок. Начиная с версии 0.7.6 Vala предоставляет специальный синтаксис для асинхронного программирования.

## Синтаксис и Примеры

Асинхронный метод определяется модификатором async. Вы можете вызывать async метод из синхронного через вызов async\_method\_name.begin\(\).

Из async метода другие async методы могут быть вызваны ключевым словом yield. Это позволит приостановить первый async метод, пока другие на завершатся \(см. пример\).

Все это делается неявно через обратные вызовы с AsyncResult. Следовательно все асинхронные методы зависят от GIO, и вы должны компилировать ключом --pkg gio-2.0.

// Пример с асинхронными методами GIO.

```csharp
  async void display_jpeg(string fnam) {
     // Load JPEG in a background thread and display it when loaded
     [...]
  }
```

или:

```csharp
  async int fetch_webpage(string url, out string text) throws IOError {
     // Fetch a webpage asynchronously and when ready return the 
     // HTTP status code and put the page contents in 'text'
     [...]
     text = result;
     return status;
  }
```

Метод может принимать аргументы и возвращать значение, как и любой другой метод. Он может использовать оператор yield в любое время, чтобы вернуть контроль над процессором своему вызывающему объекту.

Асинхронный метод может быть вызван с любой из этих двух форм:

```csharp
  display_jpeg.begin("test.jpg");
  display_jpeg.begin("test.jpg", (obj, res) => {
      display_jpeg.end(res);
  });
```



`list_dir()` не будет блокирован. Внутри него вызываются методы `enumerate_children_async()` и next\_files\_async\(\) через ключевое слово yield. `list_dir()` продолжает выполнение, как только те вернут результат.

Создание асинхронных методов

В предыдущем примере для демонстрации применения `begin()` и yield использовались асинхронные методы GIO для демонстрации. Однако возможно создание и своих асинхронных методов.

`.callback` используется для явной регистрации метода `_finish` в качестве асинхронного. Он используется через `yield`.

// добавьте обратный вызов для вашего метода

```csharp
Idle.add(fetch_webpage.callback);
  yield;
```

// вернуть результат

```csharp
return a_result_of_some_type;
```

После yield может быть возвращен результат. Явно это выполняется в AsyncResult в методе обратного вызова. .callback во многом похож на концепцию пролонгации в определенных языках программирования \(напр. Scheme\), за исключением того, что в Vala он немедленно предоставляет контекст следующий за ближайшим выражением yield.

`end()` служит синт сахаром для метода `*_finish`. Он берет `AsyncResult` и возвращает действительный результат или кидает ошибку \(если это делает асинхронный метод\). Он вызывается как: `async_method.end(result)`в асинхронном обратном вызове.

**Examples**

See [Async Method Samples](https://wiki.gnome.org/Projects/Vala/AsyncSamples) for examples of different ways that async may be used.

```csharp
using Gtk;

using Gtk;
Button button;

public static int main (string[] args) {
    Gtk.init (ref args);

    var window = new Window ();
    window.title = "Count without blocking the UI";
    window.border_width = 10;
    window.window_position = WindowPosition.CENTER;
    window.set_default_size (350, 70);
    window.destroy.connect (Gtk.main_quit);
    button = new Button.with_label ("Start counting");
    button.clicked.connect (() => {
        count.begin();
    });

    window.add (button);
    window.show_all ();

    Gtk.main ();

    return 0;
}

async void count(){
    for(int i = 0; i < 10000; i++){
        button.label = i.to_string();
        Idle.add (count.callback);
        yield;
    }
}
```

