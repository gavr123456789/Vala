# ArrayList&lt;G&gt;

![](../.gitbook/assets/image%20%284%29.png)

Реализация массива изменяемого размера  c интерфейсом [List](https://valadoc.org/gee-0.8/Gee.List.html) .

Массив автоматически увеличивается при необходимости.

### Когда использовать

Эта реализация хороша для редко изменяемых данных. Поскольку данные хранятся в массиве, эта структура не подходит для сильно изменяемых данных. альтернативная реализация см. [LinkedList](https://valadoc.org/gee-0.8/Gee.LinkedList.html) .

## **List Example**

```csharp
using Gee;

void main () {
    var list = new ArrayList<int> ();
    list.add (1);
    list.add (2);
    list.add (5);
    list.add (4);
    list.insert (2, 3);
    list.remove_at (3);
    foreach (int i in list) {
        stdout.printf ("%d\n", i);
    }
    list[2] = 10;                       // same as list.set (2, 10)
    stdout.printf ("%d\n", list[2]);    // same as list.get (2)
}
```

**Compile and Run**

```text
$ valac --pkg gee-0.8 gee-list.vala
$ ./gee-list
```

You can use any type fitting into the size of a pointer \(e.g. int, bool, reference types\) directly as generic type argument: &lt;bool&gt;, &lt;int&gt;, &lt;string&gt;, &lt;MyObject&gt;. Other types must be "boxed" by appending a question mark: &lt;float?&gt;, &lt;double?&gt;, &lt;MyStruct?&gt;. The compiler will tell you this if necessary.

### Content:

#### Properties:

* public [EqualDataFunc](https://valadoc.org/gee-0.8/Gee.EqualDataFunc.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html)&gt; [**equal\_func**](https://valadoc.org/gee-0.8/Gee.ArrayList.equal_func.html) { get; }  The elements' equality testing function.
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**read\_only**](https://valadoc.org/gee-0.8/Gee.ArrayList.read_only.html) { get; }  Specifies whether this collection can change - i.e. wheather [add](https://valadoc.org/gee-0.8/Gee.Collection.add.html), [remove](https://valadoc.org/gee-0.8/Gee.Collection.remove.html) etc. are legal operations.
* public override [int](https://valadoc.org/glib-2.0/int.html) [**size**](https://valadoc.org/gee-0.8/Gee.ArrayList.size.html) { get; }  The number of items in this collection.

#### Creation methods:

* public [**ArrayList**](https://valadoc.org/gee-0.8/Gee.ArrayList.ArrayList.html) \(owned [EqualDataFunc](https://valadoc.org/gee-0.8/Gee.EqualDataFunc.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html)&gt;? equal\_func = null\) Constructs a new, empty array list.
* public [**ArrayList.wrap**](https://valadoc.org/gee-0.8/Gee.ArrayList.ArrayList.wrap.html) \(owned [G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html)\[\] items, owned [EqualDataFunc](https://valadoc.org/gee-0.8/Gee.EqualDataFunc.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html)&gt;? equal\_func = null\) Constructs a new array list based on provided array.

#### Methods:

* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**@foreach**](https://valadoc.org/gee-0.8/Gee.ArrayList.@foreach.html) \([ForallFunc](https://valadoc.org/gee-0.8/Gee.ForallFunc.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html)&gt; f\)
* public override [G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html) [**@get**](https://valadoc.org/gee-0.8/Gee.ArrayList.@get.html) \([int](https://valadoc.org/glib-2.0/int.html) index\) Returns the item at the specified index in this list.
* public override void [**@set**](https://valadoc.org/gee-0.8/Gee.ArrayList.@set.html) \([int](https://valadoc.org/glib-2.0/int.html) index, [G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html) item\) Sets the item at the specified index in this list.
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**add**](https://valadoc.org/gee-0.8/Gee.ArrayList.add.html) \([G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html) item\) Adds an item to this collection. Must not be called on read-only collections.
* public [bool](https://valadoc.org/glib-2.0/bool.html) [**add\_all**](https://valadoc.org/gee-0.8/Gee.ArrayList.add_all.html) \([Collection](https://valadoc.org/gee-0.8/Gee.Collection.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html)&gt; collection\)
* public override [BidirListIterator](https://valadoc.org/gee-0.8/Gee.BidirListIterator.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html)&gt; [**bidir\_list\_iterator**](https://valadoc.org/gee-0.8/Gee.ArrayList.bidir_list_iterator.html) \(\)  Returns a BidirListIterator that can be used for iteration over this list.
* public override void [**clear**](https://valadoc.org/gee-0.8/Gee.ArrayList.clear.html) \(\)  Removes all items from this collection. Must not be called on read-only collections.
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**contains**](https://valadoc.org/gee-0.8/Gee.ArrayList.contains.html) \([G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html) item\) Determines whether this collection contains the specified item.
* public override [int](https://valadoc.org/glib-2.0/int.html) [**index\_of**](https://valadoc.org/gee-0.8/Gee.ArrayList.index_of.html) \([G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html) item\) Returns the index of the first occurence of the specified item in this list.
* public override void [**insert**](https://valadoc.org/gee-0.8/Gee.ArrayList.insert.html) \([int](https://valadoc.org/glib-2.0/int.html) index, [G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html) item\) Inserts an item into this list at the specified position.
* public override [Iterator](https://valadoc.org/gee-0.8/Gee.Iterator.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html)&gt; [**iterator**](https://valadoc.org/gee-0.8/Gee.ArrayList.iterator.html) \(\) Returns a [Iterator](https://valadoc.org/gee-0.8/Gee.Iterator.html) that can be used for simple iteration over a collection.
* public override [ListIterator](https://valadoc.org/gee-0.8/Gee.ListIterator.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html)&gt; [**list\_iterator**](https://valadoc.org/gee-0.8/Gee.ArrayList.list_iterator.html) \(\) Returns a ListIterator that can be used for iteration over this list.
* public override [bool](https://valadoc.org/glib-2.0/bool.html) [**remove**](https://valadoc.org/gee-0.8/Gee.ArrayList.remove.html) \([G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html) item\) Removes the first occurence of an item from this collection. Must not be called on read-only collections.
* public override [G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html) [**remove\_at**](https://valadoc.org/gee-0.8/Gee.ArrayList.remove_at.html) \([int](https://valadoc.org/glib-2.0/int.html) index\) Removes the item at the specified index of this list.
* public override [List](https://valadoc.org/gee-0.8/Gee.List.html)&lt;[G](https://valadoc.org/gee-0.8/Gee.ArrayList.G.html)&gt;? [**slice**](https://valadoc.org/gee-0.8/Gee.ArrayList.slice.html) \([int](https://valadoc.org/glib-2.0/int.html) start, [int](https://valadoc.org/glib-2.0/int.html) stop\) Returns a slice of this list.

Полный список коллекций см [здесь](https://valadoc.org/gee-0.8/index.htm). 

