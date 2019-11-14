---
description: Contains the public fields of a GArray.
---

# Array

### **Пример**

Вставка нового значения:

```csharp
public int main (string[] args) {
	Array<string> array = new Array<string> ();
	array.append_val ("1. entry");
	array.append_val ("3. entry");

	array.insert_val (1, "2. entry");

	// Output:
	//  ``1. entry``
	//  ``2. entry``
	//  ``3. entry``
	for (int i = 0; i < array.length ; i++) {
		print ("%s\n", array.index (i));
	}

	return 0;
}
```

#### Компиляция

```text
valac --pkg glib-2.0 GLib.Array.insert_val.vala
```

### Содержание:

#### Конструкторы:

* public [**Array**](https://valadoc.org/glib-2.0/GLib.Array.Array.html) \([bool](https://valadoc.org/glib-2.0/bool.html) zero\_terminated = true, [bool](https://valadoc.org/glib-2.0/bool.html) clear = true, [ulong](https://valadoc.org/glib-2.0/ulong.html) element\_size = sizeof \( null ?\)\)
* public [**Array.sized**](https://valadoc.org/glib-2.0/GLib.Array.Array.sized.html) \([bool](https://valadoc.org/glib-2.0/bool.html) zero\_terminated = true, [bool](https://valadoc.org/glib-2.0/bool.html) clear = true, [ulong](https://valadoc.org/glib-2.0/ulong.html) element\_size = sizeof \( null ?\), [uint](https://valadoc.org/glib-2.0/uint.html) reserved\_size = 0\)

#### Methods:

* public void [**\_remove\_index**](https://valadoc.org/glib-2.0/GLib.Array._remove_index.html) \([uint](https://valadoc.org/glib-2.0/uint.html) index\) - Удалить по индексу на месте
* public void [**\_remove\_index\_fast**](https://valadoc.org/glib-2.0/GLib.Array._remove_index_fast.html) \([uint](https://valadoc.org/glib-2.0/uint.html) index\) - Удалить по индексу на месте быстро
* public void [**\_remove\_range**](https://valadoc.org/glib-2.0/GLib.Array._remove_range.html) \([uint](https://valadoc.org/glib-2.0/uint.html) index, [uint](https://valadoc.org/glib-2.0/uint.html) length\) - Удалить промежуток длинной length
* public void [**append\_val**](https://valadoc.org/glib-2.0/GLib.Array.append_val.html) \(owned [G](https://valadoc.org/glib-2.0/GLib.Array.G.html) value\) - Добавить в конец
* public void [**append\_vals**](https://valadoc.org/glib-2.0/GLib.Array.append_vals.html) \(void\* data, [uint](https://valadoc.org/glib-2.0/uint.html) len\) - Добавить несколько в конец
* public [bool](https://valadoc.org/glib-2.0/bool.html) [**binary\_search**](https://valadoc.org/glib-2.0/GLib.Array.binary_search.html) \([G](https://valadoc.org/glib-2.0/GLib.Array.G.html) target, [CompareFunc](https://valadoc.org/glib-2.0/GLib.CompareFunc.html)&lt;[G](https://valadoc.org/glib-2.0/GLib.Array.G.html)&gt; compare\_func, out [uint](https://valadoc.org/glib-2.0/uint.html) match\_index\) - Поиск
* public Array&lt;unowned [G](https://valadoc.org/glib-2.0/GLib.Array.G.html)&gt; [**copy**](https://valadoc.org/glib-2.0/GLib.Array.copy.html) \(\) - Создать копию
* public unowned [G](https://valadoc.org/glib-2.0/GLib.Array.G.html) [**index**](https://valadoc.org/glib-2.0/GLib.Array.index.html) \([uint](https://valadoc.org/glib-2.0/uint.html) index\) 
* public void [**insert\_val**](https://valadoc.org/glib-2.0/GLib.Array.insert_val.html) \([uint](https://valadoc.org/glib-2.0/uint.html) index, owned [G](https://valadoc.org/glib-2.0/GLib.Array.G.html) value\)
* public void [**insert\_vals**](https://valadoc.org/glib-2.0/GLib.Array.insert_vals.html) \([uint](https://valadoc.org/glib-2.0/uint.html) index, void\* data, [uint](https://valadoc.org/glib-2.0/uint.html) len\)
* public void [**prepend\_val**](https://valadoc.org/glib-2.0/GLib.Array.prepend_val.html) \(owned [G](https://valadoc.org/glib-2.0/GLib.Array.G.html) value\)
* public void [**prepend\_vals**](https://valadoc.org/glib-2.0/GLib.Array.prepend_vals.html) \(void\* data, [uint](https://valadoc.org/glib-2.0/uint.html) len\)
* public [G](https://valadoc.org/glib-2.0/GLib.Array.G.html) [**remove\_index**](https://valadoc.org/glib-2.0/GLib.Array.remove_index.html) \([uint](https://valadoc.org/glib-2.0/uint.html) index\)
* public [G](https://valadoc.org/glib-2.0/GLib.Array.G.html) [**remove\_index\_fast**](https://valadoc.org/glib-2.0/GLib.Array.remove_index_fast.html) \([uint](https://valadoc.org/glib-2.0/uint.html) index\)
* public [G](https://valadoc.org/glib-2.0/GLib.Array.G.html)\[\] [**remove\_range**](https://valadoc.org/glib-2.0/GLib.Array.remove_range.html) \([uint](https://valadoc.org/glib-2.0/uint.html) index, [uint](https://valadoc.org/glib-2.0/uint.html) length\)
* public void [**set\_clear\_func**](https://valadoc.org/glib-2.0/GLib.Array.set_clear_func.html) \([DestroyNotify](https://valadoc.org/glib-2.0/GLib.DestroyNotify.html) clear\_func\)
* public void [**set\_size**](https://valadoc.org/glib-2.0/GLib.Array.set_size.html) \([uint](https://valadoc.org/glib-2.0/uint.html) length\)
* public void [**sort**](https://valadoc.org/glib-2.0/GLib.Array.sort.html) \([CompareFunc](https://valadoc.org/glib-2.0/GLib.CompareFunc.html)&lt;[G](https://valadoc.org/glib-2.0/GLib.Array.G.html)&gt; compare\_func\)
* public void [**sort\_with\_data**](https://valadoc.org/glib-2.0/GLib.Array.sort_with_data.html) \([CompareDataFunc](https://valadoc.org/glib-2.0/GLib.CompareDataFunc.html)&lt;[G](https://valadoc.org/glib-2.0/GLib.Array.G.html)&gt; compare\_func\)
* public [G](https://valadoc.org/glib-2.0/GLib.Array.G.html)\[\] [**steal**](https://valadoc.org/glib-2.0/GLib.Array.steal.html) \(\)

#### Fields:

* public [G](https://valadoc.org/glib-2.0/GLib.Array.G.html)\[\] [**data**](https://valadoc.org/glib-2.0/GLib.Array.data.html)
* public [uint](https://valadoc.org/glib-2.0/uint.html) [**length**](https://valadoc.org/glib-2.0/GLib.Array.length.html)



