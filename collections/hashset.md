# HashSet&lt;G&gt;

Реализует: Iterable&lt;G&gt;, Collection&lt;G&gt;, Set&lt;G&gt;

Набор из элементов типа G. Дубликаты выявляются через вычисление хэш каждого ключа, что можно настроить через передачу указателей на функции хэширования и проверки равенства ключей нужным вам способом.

Вы можете получить вид коллекции только для чтения через свойство read\_only\_view, напр. `my_map.read_only_view`. Оно вернет вам оболочку, которая будет иметь тот же интерфейс, что и коллекция внутри, но без возможности модифицировать данную коллекцию.

### Когда использовать

Эта реализация лучше подходит для сильно разнородных значений. В случае высокого значения избыточности хэшей или большого количества данных предпочтение [отдается](https://valadoc.org/gee-0.8/Gee.TreeSet.html) реализации дерева, такой как [TreeSet](https://valadoc.org/gee-0.8/Gee.TreeSet.html) .

### Set Example <a id="Set_Example"></a>

Sets are unordered and do not contain duplicate elements.

```csharp
using Gee;

void main () {
    var my_set = new HashSet<string> ();
    my_set.add ("one");
    my_set.add ("two");
    my_set.add ("three");
    my_set.add ("two");         // will not be added because it's a duplicate
    foreach (string s in my_set) {
        stdout.printf ("%s\n", s);
    }
}
```

#### Compile and Run <a id="Compile_and_Run-1"></a>

```text
$ valac --pkg gee-0.8 gee-set.vala
$ ./gee-set
```

### Content:

#### Properties:

* public [EqualDataFunc](https://valadoc.org/gee-0.8/Gee.EqualDataFunc.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.HashSet.G.html)&gt; [**equal\_func**](https://valadoc.org/gee-0.8/Gee.HashSet.equal_func.html) { get; }The elements' equality testing function.
* public [HashDataFunc](https://valadoc.org/gee-0.8/Gee.HashDataFunc.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.HashSet.G.html)&gt; [**hash\_func**](https://valadoc.org/gee-0.8/Gee.HashSet.hash_func.html) { get; }The elements' hash function.
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**read\_only**](https://valadoc.org/gee-0.8/Gee.HashSet.read_only.html) { get; }
* public override [int](https://valadoc.org/glib-2.0/int.html) [**size**](https://valadoc.org/gee-0.8/Gee.HashSet.size.html) { get; }

#### Creation methods:

* public [**HashSet**](https://valadoc.org/gee-0.8/Gee.HashSet.HashSet.html) \(owned [HashDataFunc](https://valadoc.org/gee-0.8/Gee.HashDataFunc.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.HashSet.G.html)&gt;? hash\_func = null, owned [EqualDataFunc](https://valadoc.org/gee-0.8/Gee.EqualDataFunc.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.HashSet.G.html)&gt;? equal\_func = null\) Constructs a new, empty hash set.

#### Methods:

* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**@foreach**](https://valadoc.org/gee-0.8/Gee.HashSet.@foreach.html) \([ForallFunc](https://valadoc.org/gee-0.8/Gee.ForallFunc.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.HashSet.G.html)&gt; f\)
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**add**](https://valadoc.org/gee-0.8/Gee.HashSet.add.html) \([G](https://valadoc.org/gee-0.8/Gee.HashSet.G.html) key\)
* public override void [**clear**](https://valadoc.org/gee-0.8/Gee.HashSet.clear.html) \(\)
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**contains**](https://valadoc.org/gee-0.8/Gee.HashSet.contains.html) \([G](https://valadoc.org/gee-0.8/Gee.HashSet.G.html) key\)
* public override [Iterator](https://valadoc.org/gee-0.8/Gee.Iterator.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.HashSet.G.html)&gt; [**iterator**](https://valadoc.org/gee-0.8/Gee.HashSet.iterator.html) \(\)
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**remove**](https://valadoc.org/gee-0.8/Gee.HashSet.remove.html) \([G](https://valadoc.org/gee-0.8/Gee.HashSet.G.html) key\)

###  <a id="Set_Example"></a>

Полный список коллекций см [здесь](https://valadoc.org/gee-0.8/index.htm). 

