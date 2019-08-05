# HashMap&lt;K,V&gt;

![](../.gitbook/assets/image%20%282%29.png)

Реализация: Iterable&lt;Entry&lt;K,V&gt;&gt;, Map&lt;K,V&gt;

Реализация хеш-таблицы интерфейса [Map](https://valadoc.org/gee-0.8/Gee.Map.html) .

Реализует связи 1 к 1 по ключу типа К к элементам типа V. Отображение делается по хэшу для каждого ключа, что может быть настроено через передачу указателей на функции хэширования и проверку равенства ключей.

Вы можете передать собственную функцию хэширования и проверки равентсва конструктору, например:

```csharp
var map = new Gee.HashMap<Foo, Object>(foo_hash, foo_equal);
```

Для строк и целых чисел хэш и функции проверки равенства подставляются автоматически, объекты различаются по их ссылкам по умолчанию. Вы должны передавать пользовательские функции хэширования и проверки равенства если хотите изменить стандартное поведение.

## Когда использовать

Эта реализация лучше подходит для сильно разнородных ключевых значений. В случае избыточности хеш-ключей или большого объема данных предпочтение отдается использованию дерева, например [TreeMap](https://valadoc.org/gee-0.8/Gee.TreeMap.html) .

## Map Example  <a id="Map_Example"></a>

Maps work like a dictionary. They store key - value pairs.

```csharp
using Gee;

void main () {

    var map = new HashMap<string, int> ();

    // Setting values
    map.set ("one", 1);
    map.set ("two", 2);
    map.set ("three", 3);
    map["four"] = 4;            // same as map.set ("four", 4)
    map["five"] = 5;

    // Getting values
    int a = map.get ("four");
    int b = map["four"];        // same as map.get ("four")
    assert (a == b);

    // Iteration

    stdout.printf ("Итерация по значениям\n");
    foreach (var entry in map.entries) {
        stdout.printf ("%s => %d\n", entry.key, entry.value);
    }

    stdout.printf ("Итерация только по ключам\n");
    foreach (string key in map.keys) {
        stdout.printf ("%s\n", key);
    }

    stdout.printf ("Итерация только по значениям\n");
    foreach (int value in map.values) {
        stdout.printf ("%d\n", value);
    }

    stdout.printf ("Итерация с помощью for\n");
    var it = map.map_iterator ();
    for (var has_next = it.next (); has_next; has_next = it.next ()) {
        stdout.printf ("%d\n", it.get_value ());
    }
}
```

### Compile and Run  <a id="Compile_and_Run-2"></a>

```text
$ valac --pkg gee-0.8 gee-map.vala
$ ./gee-map
```

## Содержание:

### Свойства:

* public override [Set](https://valadoc.org/gee-0.8/Gee.Set.html)&lt;[Entry](https://valadoc.org/gee-0.8/Gee.Map.Entry.html)&lt;[K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html),[V](https://valadoc.org/gee-0.8/Gee.HashMap.V.html)&gt;&gt; [**entries**](https://valadoc.org/gee-0.8/Gee.HashMap.entries.html) { owned get; }  The read-only view of the entries of this map.
* public [EqualDataFunc](https://valadoc.org/gee-0.8/Gee.EqualDataFunc.html)&lt;[K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html)&gt; [**key\_equal\_func**](https://valadoc.org/gee-0.8/Gee.HashMap.key_equal_func.html) { get; }  The keys' equality testing function.
* public [HashDataFunc](https://valadoc.org/gee-0.8/Gee.HashDataFunc.html)&lt;[K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html)&gt; [**key\_hash\_func**](https://valadoc.org/gee-0.8/Gee.HashMap.key_hash_func.html) { get; }  The keys' hash function.
* public override [Set](https://valadoc.org/gee-0.8/Gee.Set.html)&lt;[K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html)&gt; [**keys**](https://valadoc.org/gee-0.8/Gee.HashMap.keys.html) { owned get; } The read-only view of the keys of this map.
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**read\_only**](https://valadoc.org/gee-0.8/Gee.HashMap.read_only.html) { get; }  Specifies whether this collection can change - i.e. wheather [set](https://valadoc.org/gee-0.8/Gee.Map.@set.html), [remove](https://valadoc.org/gee-0.8/Gee.Map.remove.html) etc. are legal operations.
* public override [int](https://valadoc.org/glib-2.0/int.html) [**size**](https://valadoc.org/gee-0.8/Gee.HashMap.size.html) { get; }  The number of items in this map.
* public [EqualDataFunc](https://valadoc.org/gee-0.8/Gee.EqualDataFunc.html)&lt;[V](https://valadoc.org/gee-0.8/Gee.HashMap.V.html)&gt; [**value\_equal\_func**](https://valadoc.org/gee-0.8/Gee.HashMap.value_equal_func.html) { get; }  The values' equality testing function.
* public override [Collection](https://valadoc.org/gee-0.8/Gee.Collection.html)&lt;[V](https://valadoc.org/gee-0.8/Gee.HashMap.V.html)&gt; [**values**](https://valadoc.org/gee-0.8/Gee.HashMap.values.html) { owned get; }  The read-only view of the values of this map.

### Методы создания:

* public [**HashMap**](https://valadoc.org/gee-0.8/Gee.HashMap.HashMap.html) \(owned [HashDataFunc](https://valadoc.org/gee-0.8/Gee.HashDataFunc.html)&lt;[K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html)&gt;? key\_hash\_func = null, owned [EqualDataFunc](https://valadoc.org/gee-0.8/Gee.EqualDataFunc.html)&lt;[K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html)&gt;? key\_equal\_func = null, owned [EqualDataFunc](https://valadoc.org/gee-0.8/Gee.EqualDataFunc.html)&lt;[V](https://valadoc.org/gee-0.8/Gee.HashMap.V.html)&gt;? value\_equal\_func = null\) Constructs a new, empty hash map.

### Методы:

* public override [V](https://valadoc.org/gee-0.8/Gee.HashMap.V.html) [**@get**](https://valadoc.org/gee-0.8/Gee.HashMap.@get.html) \([K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html) key\) Returns the value of the specified key in this map.
* public override void [**@set**](https://valadoc.org/gee-0.8/Gee.HashMap.@set.html) \([K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html) key, [V](https://valadoc.org/gee-0.8/Gee.HashMap.V.html) value\) Inserts a new key and value into this map.
* public override void [**clear**](https://valadoc.org/gee-0.8/Gee.HashMap.clear.html) \(\) Removes all items from this collection. Must not be called on read-only collections.
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**has**](https://valadoc.org/gee-0.8/Gee.HashMap.has.html) \([K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html) key, [V](https://valadoc.org/gee-0.8/Gee.HashMap.V.html) value\) Determines whether this map has the specified key/value entry.
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**has\_key**](https://valadoc.org/gee-0.8/Gee.HashMap.has_key.html) \([K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html) key\) Determines whether this map has the specified key.
* public override [MapIterator](https://valadoc.org/gee-0.8/Gee.MapIterator.html)&lt;[K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html),[V](https://valadoc.org/gee-0.8/Gee.HashMap.V.html)&gt; [**map\_iterator**](https://valadoc.org/gee-0.8/Gee.HashMap.map_iterator.html) \(\) Returns an iterator for this map.
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**unset**](https://valadoc.org/gee-0.8/Gee.HashMap.unset.html) \([K](https://valadoc.org/gee-0.8/Gee.HashMap.K.html) key, out [V](https://valadoc.org/gee-0.8/Gee.HashMap.V.html) value = null\) Removes the specified key from this map.

Полный список коллекций см [здесь](https://valadoc.org/gee-0.8/index.htm).

