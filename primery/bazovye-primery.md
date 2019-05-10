# Базовые примеры

### Hello World <a id="Hello_World"></a>

Простой Hello World:

```csharp
void main () {
    print ("hello, world\n");
}
```

#### Компиляция и запуск <a id="A.2BBBoEPgQ8BD8EOAQ7BE8ERgQ4BE8_.2BBDg_.2BBDcEMAQ.2FBEMEQQQ6-"></a>

```bash
$ valac hello.vala
$ ./hello
```

{% hint style="info" %}
Так же можно использовать просто `vala` тогда произойдет компиляция и запуск одновременно, как если бы это был Python
{% endhint %}

Если бинарник должен иметь другое имя:

```bash
$ valac hello.vala -o greeting
$ ./greeting
```

### Читая ввод пользользователя <a id="A.2BBCcEOARCBDAETw_.2BBDIEMgQ.2BBDQ_.2BBD8EPgQ7BEwENwQ.2BBDsETAQ3BD4EMgQwBEIENQQ7BE8-"></a>

```csharp
void main () {
    stdout.printf ("Введите свое имя: ");
    string name = stdin.read_line ();
    if (name != null) {
        stdout.printf ("Привет, %s!\n", name);
    }
}
```

Vala предоставляет объекты _stdin_ \(стандартный ввод\), _stdout_ \(стандартный вывод\) и _stderr_ \(стандартный поток ошибок\) для трех стандартных потоков. Метод _printf_ берет формат-строку и переменное число агрументов в качеств параметров.

### Математика <a id="A.2BBBwEMARCBDUEPAQwBEIEOAQ6BDA-"></a>

Математические функции внутри пространства имен [Math](http://valadoc.org/glib-2.0/GLib.Math.html).

```csharp
void main () {

    stdout.printf ("Please enter the radius of a circle: ");
    double radius = double.parse (stdin.read_line ()); //ввод пользователя в double
    stdout.printf ("Circumference: %g\n", 2 * Math.PI * radius);

    stdout.printf ("sin(pi/2) = %g\n", Math.sin (Math.PI / 2));

    // Random numbers

    stdout.printf ("Today's lottery results:");
    for (int i = 0; i < 6; i++) {
        stdout.printf (" %d", Random.int_range (1, 49));
    }
    stdout.printf ("\n");

    stdout.printf ("Random number between 0 and 1: %g\n", Random.next_double ());
}
```

Если вы просто ученый и вам нужен совсем мощный матан то специально для вас существует [GSL](https://www.gnu.org/software/gsl/doc/html/index.html) - GNU Scientific Library. [примеры](https://wiki.gnome.org/Projects/Vala/GSLSample) на vala, [gitbook](https://www.gnu.org/software/gsl/doc/html/index.html), [valadoc](https://valadoc.org/gsl/index.htm).

Если вам нужно посчитать математику из string, причем с длинной арифметикой, то у меня есть по этому поводу [статейка](https://gavr123456789.gitlab.io/hugo-test/post/vala/matheval/).

### Аргументы командной строки и Код Завершения <a id="A.2BBBAEQAQzBEMEPAQ1BD0EQgRL_.2BBDoEPgQ8BDAEPQQ0BD0EPgQ5_.2BBEEEQgRABD4EOgQ4_.2BBDg_.2BBBoEPgQ0_.2BBBcEMAQyBDUEQARIBDUEPQQ4BE8-"></a>

```csharp
int main (string[] args) {

    // Output the number of arguments
    stdout.printf ("%d command line argument(s):\n", args.length);

    // Enumerate all command line arguments
    foreach (string arg in args) {
        stdout.printf ("%s\n", arg);
    }

    // Exit code (0: success, 1: failure)
    return 0;
}
```

Первый аргумент командной строки \(args\[0\]\) всегда является приглашением самой программы.

### Читаем и Записываем в файл <a id="A.2BBCcEOARCBDAENQQ8_.2BBDg_.2BBBcEMAQ.2FBDgEQQRLBDIEMAQ1BDw_.2BBDI_.2BBEQEMAQ5BDs-"></a>

Это очень простая обработка текстовых файлов. Для продвинутого ввода/вывода используйте [мощные потоковые классы GIO](https://wiki.gnome.org/Vala/GIOSamples).

```csharp
void main () {
    try {
        string filename = "data.txt";

        // Writing
        string content = "hello, world";
        FileUtils.set_contents (filename, content);

        // Reading
        string read;
        FileUtils.get_contents (filename, out read);

        stdout.printf ("The content of file '%s' is:\n%s\n", filename, read);
    } catch (FileError e) {
        stderr.printf ("%s\n", e.message);
    }
}
```

### Спауним процессы <a id="A.2BBCEEPwQwBEMEPQQ4BDw_.2BBD8EQAQ.2BBEYENQRBBEEESw-"></a>

```csharp
void main () {
    try {
        // Non-blocking
        Process.spawn_command_line_async ("ls");

        // Blocking (waits for the process to finish)
        Process.spawn_command_line_sync ("ls");

        // Blocking with output
        string standard_output, standard_error;
        int exit_status;
        Process.spawn_command_line_sync ("ls", out standard_output,
                                               out standard_error,
                                               out exit_status);
    } catch (SpawnError e) {
        stderr.printf ("%s\n", e.message);
    }
}
```

### Первый класс <a id="A.2BBB8ENQRABDIESwQ5_.2BBDoEOwQwBEEEQQ-"></a>

```csharp
/* class derived from GObject */
public class BasicSample : Object {

    /* public instance method */
    public void run () {
        stdout.printf ("Hello World\n");
    }
}

/* application entry point */
int main (string[] args) {
    // instantiate this class, assigning the instance to
    // a type-inferred variable
    var sample = new BasicSample ();
    // call the run method
    sample.run ();
    // return from this main method
    return 0;
}
```

Точка входа может быть также внутри класса, если вам так угодно:

```csharp
public class BasicSample : Object {

    public void run () {
        stdout.printf ("Hello World\n");
    }

    static int main (string[] args) {
        var sample = new BasicSample ();
        sample.run ();
        return 0;
    }
}
```

В этом случае _main_ обязана быть объявлена как _static_.

