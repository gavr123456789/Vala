# Gpseq

* Что такое Gpseq?
* Ссылки

### Что такое Gpseq? <a id="user-content-what-is-gpseq"></a>

Gpseq - это библиотека предоставляющая параллелизм для Vala и GObject.

Она содержит следующий функционал:

* \*\*\*\*[**Work-stealing**](https://habr.com/ru/post/333654/) **и** managed blocking планировщик задач аналогичные планировщику Go.
* Обработка данных в стиле функциональной парадигмы с поддержкой параллельного выполнения: эквивалент [потоков](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html) Java
* [Fork-Join](https://habr.com/ru/post/128985/) параллелизм
* Параллельная сортировка
* [Фьючерсы и промисы](https://en.wikipedia.org/wiki/Futures_and_promises)
* 64-битные атомарные операции
* Переполнение безопасных арифметических функций для целых чисел со знаком
* ...

### Ссылки <a id="user-content-references"></a>

* Еще не зарегистрирован на [Valadoc.org](https://valadoc.org)
* Локальная версия[ Valadoc ](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/index.html?job=build)
* [GtkDoc \(C API\)](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/gtkdoc/html/index.html?job=build)
* \(TODO\) pgi-docs \(Python API\)
* \(TODO\) gjs-docs \(JavaScript API\)

## Futures и promises

Содержание

* Future.of\(\)
* Future.err\(\)
* wait\(\) and wait\_until\(\)
* value
* map\(\)
* flat\_map\(\)
* light\_map\(\)
* map\_err\(\)
* then\(\)
* and\_then\(\)
* zip\(\)
* transform\(\)
* Цепочка фьючерсов
* Зачем использовать Gpseq.Future вместо Gee.Future?

Библиотека Gpseq предоставляет два вида объектов [Future](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Future.html?job=build) и [Promise](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Promise.html?job=build).

Future - это read-only значение, которое становиться доступным в определенный момент. Значение \(или исключение\) обычно устанавливается с помощью Promise.

метод main\(\) и `using Gpseq;`опущены

Пример 1. Futures и promises

```csharp
Promise<int> promise = new Promise<int>();
Future<int> future = promise.future;
promise.set_value(123);
print("%d\n", future.value);
```

Вывод: `123`

```csharp
Promise<int> promise = new Promise<int>();
Future<int> future = promise.future;
promise.set_exception( new OptionalError.NOT_PRESENT("Oops!") );
print("%s\n", future.exception.message);
```

Вывод: `Oops!`

Операции [Seq](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Seq.html?job=build) не блокируют текущий поток. Вместо этого большинство из них возвращают Future как результат. Он будет вычислен в качестве значения или исключения.



### Примеры методов <a id="user-content-method-examples"></a>

####  Future.of \(\) <a id="user-content-future-of"></a>

```csharp
public static Future<G> of<G> (owned G value)
```

Future.of \(\) - это вспомогательный метод, который возвращает `Future`, заполненный заданным значением.

```csharp
Future<int> future = Future.of<int>(123);
print("%d\n", future.value);
```

Вывод: `123`

```text
123
```

####  Future.err \(\) <a id="user-content-future-err"></a>

```csharp
public static Future<G> err<G> (owned Error exception)
```

Future.err \(\) - это вспомогательный метод, который возвращает `Future`, заполненный заданным исключением.

#### wait\(\) и wait\_until\(\) <a id="user-content-wait-and-wait_until"></a>

```csharp
public abstract weak G wait () throws Error
public abstract bool wait_until (int64 end_time, out G value = null) throws Error
```

Пример 2. wait\(\)

```csharp
const ulong SEC = 1000000;

Promise<void*> promise = new Promise<void*>();
new Thread<void*>("test", () => {
    Thread.usleep(5 * SEC);
    promise.set_value(null);
    return null;
});
promise.future.wait();
assert(promise.future.ready);
```

Пример 3. wait\_until\(\)

```csharp
const ulong SEC = 1000000;

Promise<void*> promise = new Promise<void*>();
new Thread<void*>("test", () => {
    Thread.usleep(5 * SEC);
    promise.set_value(null);
    return null;
});
bool result = promise.future.wait_until( get_monotonic_time() + 1*SEC );
assert( !promise.future.ready );
assert( !result );
```

####  value <a id="user-content-value"></a>

Получает значение `Future`

Если значение не ещё готово, "получающая" значение сторона будет находиться в заблокированном состоянии, пока значение не будет вычислено. Если `Future` завершается исключением, получение значения вызовет [GLib.error](https://valadoc.org/glib-2.0/GLib.error.html).

```csharp
const ulong SEC = 1000000;

Promise<int> promise = new Promise<int>();
new Thread<void*>("test", () => {
    Thread.usleep(5 * SEC);
    promise.set_value(123);
    return null;
});
int val = promise.future.value;
assert(promise.future.ready);
assert(val == 123);
```

```csharp
Promise<int> promise = new Promise<int>();
promise.set_exception( new OptionalError.NOT_PRESENT("Oops!") );
int val = promise.future.value;
```

Вывод: `ERROR: Oops!`

####  map\(\) <a id="user-content-map"></a>

map - применяет функцию к каждомы елементу коллекции:

```csharp
Future<int> future = Future.of<string>("123").map<int>(g => int.parse(g));
assert(future.value == 123);
```

####  flat\_map\(\) <a id="user-content-flat_map"></a>

Работает как map, но с одним отличием — можно преобразовать один элемент в ноль, один или множество других.

Для того, чтобы один элемент преобразовать в ноль элементов, нужно вернуть null, либо пустой стрим. Чтобы преобразовать в один элемент, нужно вернуть Future из одного элемента, например, через Future.of\(x\). Для возвращения нескольких элементов, можно любыми способами создать Future с этими элементами.

```csharp
Future<int> future = Future.of<string>("123")
                           .flat_map<int>(f => {
                                int val = int.parse(f.value);
                                return Future.of<int>(val);
                           });
assert(future.value == 123);
```

####  light\_map\(\) <a id="user-content-light_map"></a>

light\_map\(\) - это облегченная версия map\(\). Данная функция может быть вычислена повторно в любое время.

Пример 4. map \(\) против light\_map \(\)

```csharp
Future<int> future = Future.of<string>("123")
                           .map<int>(g => {
                                print("map()\n");
                                return int.parse(g);
                           });
assert(future.value == 123);
assert(future.value == 123);

future = Future.of<string>("123")
               .light_map<int>(g => {
                    print("light_map()\n");
                    return int.parse(g);
               });
assert(future.value == 123);
assert(future.value == 123);
```

Вывод:

```csharp
map()
light_map()
light_map()
```

####  map\_err\(\) <a id="user-content-map_err"></a>

```csharp
Future<int> future = Future.of<int>(123)
                           .map_err(err => {
                               return new OptionalError.NOT_PRESENT("Hello, " + err.message);
                           });
assert(future.value == 123);
```

```csharp
Future<int> future = Future.err<int>( new OptionalError.NOT_PRESENT("Oops!") )
                           .map_err(err => {
                               return new OptionalError.NOT_PRESENT("Hello, " + err.message);
                           });
assert(future.exception != null);
assert(future.exception.message == "Hello, Oops!");
```

####  then\(\) <a id="user-content-then"></a>

Функция `then()` отложено запускает переданную в качестве аргумента функцию \(принимающую future\) - после вычисления текущего future, результатом чего может является **значение** или __**исключение**.

Примечание автора перевода: Если по человечески -- С помощью функции then\(\) можно создать цепочку ленивых вычислений, то есть поставить их в очередь, получая гарантию что следующие будут вычислены только после предыдущих. 

```csharp
Promise<void*> promise = new Promise<void*>();
promise.set_value(null);
promise.future.then(future => {
    if (future.exception == null) {
        print("then() with a value\n");
    } else {
        print("then() with an exception\n");
    }
});
```

Вывод:

```csharp
then() with a value
```

exception .vala

```csharp
Promise<void*> promise = new Promise<void*>();
promise.set_exception( new OptionalError.NOT_PRESENT("Oops!") );
promise.future.then(future => {
    if (future.exception == null) {
        print("then() with a value\n");
    } else {
        print("then() with an exception\n");
    }
});
```

Вывод:

```csharp
then() with an exception
```

#### and\_then\(\) <a id="user-content-and_then"></a>

Функция `and_then()` отложено запускает переданную в качестве аргумента функцию \(принимающую future\) - после вычисления текущего future, результатом чего может является **значение, а** _**не исключение**_ .

value.vala

```csharp
Promise<void*> promise = new Promise<void*>();
promise.set_value(null);
promise.future.and_then(g => {
    print("and_then()\n");
});
```

output:

```csharp
and_then()
```

exception.vala

```csharp
Promise<void*> promise = new Promise<void*>();
promise.set_exception( new OptionalError.NOT_PRESENT("Oops!") );
promise.future.and_then(g => {
    print("and_then()\n");
});
```

output: `(none)`

####  zip\(\) <a id="user-content-zip"></a>

zip\(\) объединяет значения двух фьючерсов в один.

Простой пример: zip \[1,2,3\] \[3,2,1\] - соеденяет списки \[\(1,3\),\(2,2\),\(3,1\)\]

```csharp
Future<string> future = Future.of<string>("100");
Future<string> future2 = Future.of<string>("23");
Future<int> result = future.zip<string,int>((a, b) => {
                         return int.parse(a) + int.parse(b);
                     }, future2);
print("%d\n", result.value);
```

output: `123`

####  transform\(\) <a id="user-content-transform"></a>

transform\(\) создает новый future применяя переданную в качестве аргумента функцию к _этому_ future, после того как _этот_  future будет вычислен.

Все остальные mapping функции — map\(\), flat\_map\(\), then\(\), etc. — кроме light\_map\(\) реализованы с помощью transform\(\).

```csharp
Future<int> future = Future.of<string>("123")
                           .transform<int>(f => {
                               if (f.exception != null) {
                                   // ...
                               }
                               int val = f.value;
                               return Future.of<int>(val);
                           });

print("%d\n", future.value);
```

Вывод: `123`

### Цепочка фьючерсов

Вы можете поместить mapping методы в цепочку.

```csharp
Future.of<int>(100)
      .map<int>(g => g + 23)
      .map<string>(g => @"Hello $g!")
      .and_then(g => print("%s\n", g));
```

вывод: `Hello 123!`

```csharp
Future.of<int>(100)
      .and_then(g => print("Succeeded 0\n"))
      .map<int>(g => {
          throw new OptionalError.NOT_PRESENT("Oops!");
      })
      .and_then(g => print("Succeeded 1\n"))
      .then(f => {
          if (f.exception != null) print("Failed\n");
      });
```

Вывод:

```csharp
Succeeded 0
Failed
```

###  Зачем использовать Gpseq.Future, вместо Gee.Future? <a id="user-content-why-use-gpseq-future-not-gee-future"></a>

Libgee уже предоставляет фьючерсы и промисы. Однако с некоторыми функциями есть проблемы, а некоторых не хватает.

* Gee версия переопределяет исключения с помощью `FutureError.EXCEPTION` в map функциях — `map()`, `flat_map()`, и `zip()`. `FutureError.EXCEPTION` вместо реального исключения не несет никакой информации и нежелателен. [libgee\#23](https://gitlab.gnome.org/GNOME/libgee/issues/23)
* Gee.Future не поддерживает цепочки вычислений фьючерсов. `map()` on an already completed future doesn’t return and blocks the program. [libgee\#31](https://gitlab.gnome.org/GNOME/libgee/issues/31)
* Exceptions can’t be thrown in the map functions, and there is no way to map exceptions. [libgee\#22](https://gitlab.gnome.org/GNOME/libgee/issues/22) [libgee\#24](https://gitlab.gnome.org/GNOME/libgee/issues/24)

Code refactoring may be required to solve the problems, and it will break backward compatibility of libgee. Instead of waiting for the problems to be fixed, I chose to implement Gpseq’s own Future.

Compare [Gpseq.Future](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Future.html?job=build) with [Gee.Future](https://valadoc.org/gee-0.8/Gee.Future.html) to see the difference.

## Seq

[Seq](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Seq.html?job=build) это последовательность элементов, поддерживающая паралельные и последовательные операции. Эквивалент Java’s [streams](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html).

###  Seq pipelines <a id="user-content-seq-pipelines"></a>

Seq поддерживает различные методы обработки данных. В Seq есть 2 типа операций: _промежуточных_ и _терминальных_. Seq последовательность может состоять из нуля и более промежуточных операций и терминальной операции.

![&#x41F;&#x43E;&#x441;&#x43B;&#x435;&#x434;&#x43E;&#x432;&#x430;&#x442;&#x435;&#x43B;&#x44C;&#x43D;&#x43E;&#x441;&#x442;&#x44C; &#x432;&#x44B;&#x437;&#x43E;&#x432;&#x43E;&#x432; \(&#x43E;&#x440;&#x438;&#x433;. A seq pipeline \)](../.gitbook/assets/image%20%285%29.png)

#### Промежуточные Операции <a id="user-content-intermediate-operations"></a>

Промежуточные операции всегда выполняются [лениво](https://ru.wikipedia.org/wiki/%D0%9B%D0%B5%D0%BD%D0%B8%D0%B2%D1%8B%D0%B5_%D0%B2%D1%8B%D1%87%D0%B8%D1%81%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F). Это означает что вычисление seq не начнется пока не будет выполен хотя бы один терминальный оператор. 

Промежуточные операции также делятся на те у которых есть состояния и те у которых их[ нет](https://ru.stackoverflow.com/questions/614175/%D0%A7%D1%82%D0%BE-%D0%B7%D0%BD%D0%B0%D1%87%D0%B8%D1%82-%D0%BE%D1%82%D1%81%D1%83%D1%82%D1%81%D1%82%D0%B2%D0%B8%D0%B5-%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D1%8F)\(_stateless_ и _stateful_ \). Операции без сохранения состояния не сохраняют состояние предыдущего элемента при обработке нового элемента. Поэтому каждый элемент может обрабатываться независимо от других элементов. В операциях с отслеживанием состояния\(_stateful_\) учитываются состояния предыдущих операций над элементами при обработке новых элементов, из-за чего обработка может быть выполнена только последовотельн\(Например вычисление чисел Фибоначчи\).

Следовательно, конвейеры seq, содержащие промежуточные _stateful_ операции, могут потребовать нескольких проходов или буферизации важных промежуточных вычислений. Напротив, конвейеры seq, содержащие только stateless операции, могут обрабатываться за один проход.

Stateless операции:

`filter, flat_map, map, parallel, sequential, etc.`

Stateful операции:

`chop, chop_ordered, distinct, limit, limit_ordered, order_by, skip, skip_ordered, etc.`

####  Терминальные операции <a id="user-content-terminal-operations"></a>

![](../.gitbook/assets/image%20%2814%29.png)

Терминальные операции могут изменять источник для получения результата или побочного эффекта. После выполнения терминальной операции seq последовательность закрывается и больше не может использоваться.

Терминальные операции:

foreach, all\_match, any\_match, collect, collect\_ordered, count, find\_any, find\_first, fold, group\_by, iterator, max, min, none\_match, partition, reduce, spliterator, etc.

####  Укороченные \(Short-circuiting\) операции <a id="user-content-short-circuiting"></a>

Промежуточные укороченные операции могут привести к тому что seq итерирующий по бесконечному источнику\(например генератору четных чисел\) приведет к результату за конечное время .

{% hint style="info" %}
Для того чтобы обработка seq бесконечного источника завершилась нормально за конечное время, необходимо, чтобы трубопровод содержал операцию короткого замыкания.
{% endhint %}

Short-circuiting операции:

all\_match, any\_match, chop, chop\_ordered, find\_any, find\_first, limit, limit\_ordered, none\_match, etc.

####  Parallelism <a id="user-content-parallelism"></a>

При запуске терминальной операции конвейер seq выполняется последовательно или параллельно в зависимости от режима seq, в котором он вызывается. Последовательный режим является режимом по умолчанию. Он может быть изменен с помощью промежуточных операций `sequential()` или `parallel()`.

Все операции соблюдают порядок встречи элементов seq при последовательном выполнении. Однако при параллельном выполнении все промежуточные и терминальные операции с сохранением состояния могут не учитывать реальный порядок порядок элементов\(на то они и паралельные\), за исключением операций, определенных как явно упорядоченные, таких как `find_first()`.

{% hint style="info" %}
Возвращаемый результат любой терминальной операции уже является вычисленным, поэтому вам не нужно ждать future для получения результата
{% endhint %}

### Примеры методов <a id="user-content-method-examples"></a>

Main method, `using Gpseq;`, and `using Gee;` omitted.

####  foreach\(\) <a id="user-content-foreach"></a>

```csharp
string[] array = {"dog", "cat", "pig"};
Seq.of_array<string>(array).foreach(g => print("%s\n", g));
```

output:

```csharp
dog
cat
pig
```

####  filter\(\) <a id="user-content-filter"></a>

Промежуточный оператор filter отбирает только те строки, длина которых не превышает 3.

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
Seq.of_array<string>(array)
    .filter(g => g.length == 3)
    .foreach(g => print("%s\n", g));
```

output:

```text
dog
cat
pig
```

####  map\(\) <a id="user-content-map"></a>

Применяет функцию к каждому элементу и затем возвращает стрим, в котором элементами будут результаты функции. map можно применять для изменения типа элементов.

```csharp
string[] array = {"dog", "cat", "pig"};
Seq.of_array<string>(array)
    .map<string>(g => g.up())
    .foreach(g => print("%s\n", g));
```

output:

```text
DOG
CAT
PIG
```

####  flat\_map\(\) <a id="user-content-flat_map"></a>

Работает как map, но с одним отличием — можно преобразовать один элемент в ноль, один или множество других.

Для того, чтобы один элемент преобразовать в ноль элементов, нужно вернуть `null`, либо пустой стрим. Чтобы преобразовать в один элемент, нужно вернуть стрим из одного элемента, например, через Seq.of\(x\). Для возвращения нескольких элементов, можно любыми способами создать стрим с этими элементами.

```csharp
string[] array = {"dog", "cat", "pig"};
Seq.of_array<string>(array)
    .flat_map<string>(g => {
        var list = new ArrayList<string>();
        list.add(g);
        list.add(g + "2");
        return list.iterator();
    })
    .foreach(g => print("%s\n", g));
```

output:

```text
dog
dog2
cat
cat2
pig
pig2
```

####  skip\(\), limit\(\), and chop\(\) <a id="user-content-skip-limit-and-chop"></a>

Example 1. skip\(\)

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
Seq.of_array<string>(array)
    .skip(3) // It is equivalent to chop(3)
    .foreach(g => print("%s\n", g));
```

output:

```text
boar
bear
```

Example 2. limit\(\)

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
Seq.of_array<string>(array)
    .limit(3) // It is equivalent to chop(0, 3)
    .foreach(g => print("%s\n", g));
```

output:

```text
dog
cat
pig
```

Example 3. chop\(\)

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
Seq.of_array<string>(array)
    .chop(2, 1)
    .foreach(g => print("%s\n", g));
```

output:

```text
pig
```

####  skip\_ordered\(\), limit\_ordered\(\), and chop\_ordered\(\) <a id="user-content-skip_ordered-limit_ordered-and-chop_ordered"></a>

They are the ordered version of skip/limit/chop. They always respect encounter order regardless of sequential/parallel execution. They are, however, quite expensive on parallel execution.

####  order\_by\(\) <a id="user-content-order_by"></a>

```csharp
string[] array = {"dog", "cat", "pig"};
Seq.of_array<string>(array)
    .order_by()
    .foreach(g => print("%s\n", g));
```

output:

```text
cat
dog
pig
```

####  distinct\(\) <a id="user-content-distinct"></a>

```csharp
string[] array = {"dog", "dog", "cat", "cat", "pig"};
Seq.of_array<string>(array)
    .distinct()
    .foreach(g => print("%s\n", g));
```

output:

```text
dog
cat
pig
```

####  all\_match\(\) and any\_match\(\) <a id="user-content-all_match-and-any_match"></a>

You should read the [documentation](02.-Futures-and-promises) of [Future](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Future.html?job=build) first.Example 4. all\_match\(\)

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
bool result = (!) Seq.of_array<string>(array)
                     .all_match(g => g.length == 3).value;
print("%s\n", result.to_string());
```

output:

```text
false
```

Example 5. any\_match\(\)

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
Future<bool?> future = Seq.of_array<string>(array)
                          .any_match(g => g.length == 3);
bool result = (!) future.value;
print("%s\n", result.to_string());
```

output:

```text
true
```

|  |  `(!)` is the explicit non-null cast operator in Vala. \(Currently not necessary, except when using `--enable-experimental-non-null` option\) |
| :--- | :--- |


####  count\(\) <a id="user-content-count"></a>

```csharp
string[] array = {"dog", "dog", "cat", "cat", "pig"};
int64 count = (!) Seq.of_array<string>(array).distinct().count().value;
print("%" + int64.FORMAT + "\n", count);
```

output:

```text
3
```

####  find\_any\(\) and find\_first\(\) <a id="user-content-find_any-and-find_first"></a>

Example 6. find\_any\(\)

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
Optional<string> result = Seq.of_array<string>(array)
                             .find_any(g => g.length == 4).value;
print("%s\n", result.value);
```

output:

```text
boar
```

Example 7. find\_first\(\)

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
Optional<string> result = Seq.of_array<string>(array)
                             .find_first(g => g.length == 4).value;
print("%s\n", result.value);
```

output:

```text
boar
```

Example 8. find\_any\(\) vs find\_first\(\) on parallel execution

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};

Optional<string> any = Seq.of_array<string>(array)
                          .parallel()
                          .find_any(g => g.length == 3).value;
assert(any.value == "dog" || any.value == "cat" || any.value == "pig");

Optional<string> first = Seq.of_array<string>(array)
                            .parallel()
                            .find_any(g => g.length == 3).value;
assert(first.value == "dog");
```

####  max\(\) and min\(\) <a id="user-content-max-and-min"></a>

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
Optional<int> max = Seq.of_array<string>(array)
                          .map<int>(g => g.length)
                          .max();
Optional<int> min = Seq.of_array<string>(array)
                          .map<int>(g => g.length)
                          .min();
print("max: %d\n", max.value);
print("min: %d\n", min.value);
```

output:

```text
max: 4
min: 3
```

####  Seq.iterate\(\) <a id="user-content-seq-iterate"></a>

```csharp
public static Seq<G> iterate<G> (owned G seed,
                                 owned Gpseq.Predicate<G> pred,
                                 owned Gpseq.MapFunc<G,G> next,
                                 TaskEnv? env = null)
```

Each element is generated by the given _next_ function applied to the previous element, and the initial element is the _seed_.Example 9. For loop

```text
Seq.iterate<int>(0, i => i < 5, i => ++i)
    .foreach(g => print("%s ", g));
```

output:

```text
0 1 2 3 4
```

####  reduce\(\) and fold\(\) <a id="user-content-reduce-and-fold"></a>

Example 10. reduce\(\)

```csharp
Optional<int> result = Seq.iterate<int>(0, i => i < 5, i => ++i)
                          .reduce((a, b) => a + b);
print("%d\n", result.value);
```

output:

```text
10
```

Example 11. fold\(\)

```csharp
Optional<int> result = Seq.iterate<int>(0, i => i < 5, i => ++i)
                          .fold<int>((a, b) => a + b, (a, b) => a + b, 0);
print("%d\n", result.value);
```

output:

```text
10
```

####  group\_by\(\) <a id="user-content-group_by"></a>

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
Map<int,Gee.List<string>> result = Seq.of_array<string>(array)
                                      .group_by<int>(g => g.length)
                                      .value;
print("3-length: %d\n", result[3].size);
print("4-length: %d\n", result[4].size);
```

output:

```text
3-length: 3
4-length: 2
```

####  partition\(\) <a id="user-content-partition"></a>

```csharp
string[] array = {"dog", "cat", "pig", "boar", "bear"};
Map<bool,Gee.List<string>> result = Seq.of_array<string>(array)
                                      .partition(g => g.length == 3)
                                      .value;
print("true: %d\n", result[true].size);
print("false: %d\n", result[false].size);
```

output:

```text
true: 3
false: 2
```

####  iterator\(\) <a id="user-content-iterator"></a>

```csharp
string[] array = {"dog", "dog", "cat", "cat", "pig"};
Iterator<string> iter = Seq.of_array<string>(array).distinct().iterator();
iter.foreach(g => {
    print("%s\n", g);
    return true;
});
```

output:

```text
dog
cat
pig
```

####  spliterator\(\) <a id="user-content-spliterator"></a>

[Spliterator](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Spliterator.html?job=build) is an object for traversing and partitioning elements of a data source, used in Seq.

You should read the documentation of Spliterator first.

```csharp
string[] array = {"dog", "dog", "cat", "cat", "pig"};
Spliterator<string> spliter = Seq.of_array<string>(array)
                                 .distinct()
                                 .spliterator();
spliter.each(g => print("%s\n", g));
```

output:

```text
dog
cat
pig
```

####  collect\(\) and collect\_ordered\(\) <a id="user-content-collect-and-collect_ordered"></a>

See the section Collectors.

Both perform a mutable reduction operation on the elements of a seq.collect\(\)

collect\(\) performs a concurrent reduction if the seq is in parallel mode and the collector is CONCURRENT.collect\_ordered\(\)

collect\_ordered\(\) preserves encounter order even though the seq is in parallel mode, if the collector is not CONCURRENT or not UNORDERED. If, however, the seq is in parallel mode and the collector is CONCURRENT and UNORDERED, it performs an unordered concurrent reduction.

###  Error handling <a id="user-content-error-handling"></a>

Most seq operations take delegates which can throw errors. If an error occurs in the delegate, the returned future is completed with the error.Example 12. Error handling

```csharp
try {
    Seq.iterate<int>(0, i => i < 100, i => ++i)
        .parallel()
        .foreach(i => {
            if (i == 42) {
                throw new OptionalError.NOT_PRESENT("%d? Oops!", i);
            }
        }).wait(); // Gpseq.Future.wait() throws Error
} catch (Error err) {
    error(err.message);
}
```

output:

```text
ERROR: 42? Oops!
```

###  Collectors <a id="user-content-collectors"></a>



[Collector](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Collector.html?job=build) is an object for mutable reduction operation that accumulates input elements into a mutable accumulator, and transforms it into a final result. It is used with `collect()` or `collect_ordered()`.

[Gpseq.Collectors](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Collectors.html?job=build) namespace provides various useful collector implementations. Here are some examples:

Main method, `using Gpseq;`, `using Gpseq.Collectors;`, and `using Gee;` omitted.Example 13. Collectors.to\_list\(\)

```text
string[] array = {"dog", "dog", "cat", "cat", "pig"};
Gee.List<string> list = Seq.of_array<string>(array)
                                 .distinct()
                                 .collect( to_list<int>() )
                                 .value;
Seq.of_list<string>(list).foreach(g => print("%s\n", g));
```

output:

```text
dog
cat
pig
```

Example 14. Collectors.to\_generic\_array\(\)

```text
string[] array = {"dog", "dog", "cat", "cat", "pig"};
GenericArray<string> array = Seq.of_array<string>(array)
                                 .distinct()
                                 .collect( to_generic_array<int>() )
                                 .value;
Seq.of_generic_array<string>(array).foreach(g => print("%s\n", g));
```

output:

```text
dog
cat
pig
```

|  |  GenericArray \(Vala\) == GPtrArray \(C-lang\) |
| :--- | :--- |


Example 15. Collectors.sum\_int\(\)

```text
string[] array = {"dog", "cat", "pig"};
int sum = Seq.of_array<string>(array)
             .collect( sum_int<string>(g => g.length) )
             .value;
print("%d\n", sum);
```

output:

```text
9
```

|  |  The arithmetic wraps around on overflow; e.g. _the sum of int.MAX and 1 ⇒ int.MIN_ |
| :--- | :--- |


Example 16. Collectors.count\(\)

```text
string[] array = {"dog", "cat", "pig"};
int64 result = (!) Seq.of_array<string>(array)
                     .collect( count<string>() )
                     .value;
print("%d\n", result);
```

output:

```text
3
```

Example 17. Collectors.join\(\)with\_default\_delimiter.vala

```text
string[] array = {"dog", "cat", "pig"};
string result = Seq.of_array<string>(array)
                   .collect( join() )
                   .value;
print("%s\n", result);
```

output:

```text
dogcatpig
```

delimiter\_specified.vala

```text
string[] array = {"dog", "cat", "pig"};
string result = Seq.of_array<string>(array)
                   .collect( join(",") )
                   .value;
print("%s\n", result);
```

output:

```text
dog,cat,pig
```

Example 18. Collectors.partition\_with\(\)

```text
public Collector<Map<bool,V>,Object,G> partition_with<V,G> (
        owned Predicate<G> pred,
        Collector<V,Object,G> downstream)
```

partition\_with.vala

```text
string[] array = {"dog", "cat", "pig", "boar", "bear"};
// Map<bool,Gee.List<string>>
var map = Seq.of_array<string>(array)
              .collect( partition_with<Gee.List<string>,string>(
                    g => g.length,
                    to_list<string>()) )
              .value;
print("true: %d\n", map[true].size);
print("false: %d\n", map[false].size);
```

output:

```text
true: 3
false: 2
```

|  |  `Seq.partition(pred)` is equivalent to `collect( partition<G>(pred) )`. And `partition<G>(pred)` is equivalent to `partition_with<Gee.List<G>,G>(pred, to_list<G>())`. |
| :--- | :--- |


Example 19. Collectors.wrap\(\)

wrap\(\) takes a collector and returns a collector will produce a [Wrapper](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Wrapper.html?job=build) object containing the result of the given collector.

```text
string[] array = {"dog", "cat", "pig"};
Wrapper<int64?> result = Seq.of_array<string>(array)
                     .collect( wrap<int64?,string>(count<string>()) )
                     .value;
print("%d\n", result.value);
```

output:

```text
3
```

Example 20. Collectors.tee\(\)

```text
public Collector<A,Object,G> tee<A,G> (
        owned Collector<Object,Object,G>[] downstreams,
        owned TeeMergeFunc<A> merger)

public delegate A TeeMergeFunc<A> (Object[] results) throws Error
```

tee.vala

```text
var seq = Seq.iterate<int>(0, i => i < 5, i => ++i);
var collector = tee<Gee.List<int64?>,string>(
    {
        wrap<int64?,G>( sum_int64<string>(g => g) ),
        wrap<int64?,G>( count<string>() )
    },
    (results) => {
        var list = new ArrayList<int64?>();
        list.add( ((Wrapper<int64?>)results[0]).value );
        list.add( ((Wrapper<int64?>)results[1]).value );
        return list;
    }
);
Gee.List<int64?> result = seq.collect(collector).value;
print("sum: %d\n", result[0]);
print("count: %d\n", result[1]);
```

output:

```text
sum: 10
count: 5
```

####  Custom collectors <a id="user-content-custom-collectors"></a>

A collector implements four methods: create\_accumulator, accumulate, combine, and finish.

1. create\_accumulator\(\) - Creates a new accumulator, such as Gee.Collection.
2. accumulate\(\) - Incorporates a new element into a accumulator.
3. combine\(\) - Combines two accumulators into one.
4. finish\(\) - Transforms the accumulator into a final result.

The methods must satisfy an _identity_ and an _associativity_ constraints. The identity constraint means that combining any accumulator with an empty accumulator must produce an equivalent result. i.e. an accumulator a must be equivalent to `collector.accumulate(a, collector.create_accumulator())`.

The associativity constraint means that splitting the computation must produce an equivalent result. i.e.:Associativity constraint

```text
// the two computations below must be equivalent.
// collector: a collector
// g0, g1: elements

A a0 = collector.create_accumulator();
collector.accumulate(g0, a0);
collector.accumulate(g1, a0);
A r0 = collector.finish(a0);

A a1 = collector.create_accumulator();
collector.accumulate(g0, a1);
A a2 = collector.create_accumulator();
collector.accumulate(g1, a2);
A r1 = collector.finish( collector.combine(a1, a2) );
```

Collectors also have a property, [features](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Collector.features.html?job=build). it provides hints that can be used to optimize the operation.

Here is the collector implementation of Collectors.to\_collection&lt;G&gt;\(\):CollectionCollector.vala

```text
using Gee;

private class Gpseq.Collectors.CollectionCollector<G> : Object, Collector<Collection<G>,Collection<G>,G> {
    private Supplier<Collection<G>> _factory;
    private CollectorFeatures _features;

    public CollectionCollector (Supplier<Collection<G>> factory, CollectorFeatures features) {
        _factory = factory;
        _features = features;
    }

    public CollectorFeatures features {
        get {
            return _features;
        }
    }

    public Collection<G> create_accumulator () throws Error {
        return _factory.supply();
    }

    public void accumulate (G g, Collection<G> a) throws Error {
        a.add(g);
    }

    public Collection<G> combine (Collection<G> a, Collection<G> b) throws Error {
        a.add_all(b);
        return a;
    }

    public Collection<G> finish (Collection<G> a) throws Error {
        return a;
    }
}
```

###  Worker pools <a id="user-content-worker-pools"></a>

TODO

###  References <a id="user-content-references"></a>

* More detailed description about seq pipelines can be found on the Seq documentations: [Valadoc](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/valadoc/gpseq-1.0/Gpseq.Seq.html?job=build), [GtkDoc](https://gitlab.com/kosmospredanie/gpseq/-/jobs/artifacts/master/file/gtkdoc/html/gpseq-1.0-GpseqSeq.html?job=build)

