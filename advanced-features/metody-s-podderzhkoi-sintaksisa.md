# Методы с поддержкой синтаксиса

Vala распознает методы с определенными именами и сигнатурами и обеспечивает синтаксическую поддержку для них. Например если у типа есть метод contains\(\), то их можно использовать с оператором in. Т и Тn здесь обозначают только места, куда должны быть вставлены настоящие типы.

Индексеры

| T2 get\(T1 index\) | index access: obj\[index\] |
| :--- | :--- |
| void set\(T1 index, T2 item\) | index assignment: obj\[index\] = item |
| **Индексеры с множеством индексов** |  |
| T3 get\(T1 index1, T2 index2\) | index access: obj\[index1, index2\] |
| void set\(T1 index1, T2 index2, T3 item\) | index assignment: obj\[index1, index2\] = item |
| **\(... и так далее для большего количества индексов\)** |  |
| **Другие** |  |
| T slice\(long start, long end\) | slicing: obj\[start:end\] |
| bool contains\(T needle\) | `in` operator: bool b = needle in obj |
| string to\_string\(\) | поддержка внутри строковых шаблонов: @"$obj" |
| Iterator iterator\(\) | можно итерировать посредством foreach |
| T2 get\(T1 index\) T1 size { get; } | можно итерировать посредством foreach |

Другие

T slice\(long start, long end\)

slicing: obj\[start:end\]

bool contains\(T needle\)

in operator: bool b = needle in obj

string to\_string\(\)

поддержка внутри строковых шаблонов: @"$obj"

Iterator iterator\(\)

можно итерировать посредством foreach

**Тип Iterator может иметь любое имя и должен реализовать один из следующих протоколов:**

| bool next\(\) T get\(\) | стандартный протокол итератора: итерировать пока .next\(\) не вернеть false. Текущий элемент получается методом .get. |
| :--- | :--- |
| T? next\_value\(\) | альтернативный протокол итератора: Если итератор имеет функцию .next\_value\(\), которая возвращает значение, которое может быть null, то мы итерируем вызывая указанный метод, пока он не вернеть null. |

Данный пример реализует некоторые из этих методов:

```csharp
public class EvenNumbers {
    public int get(int index) {
        return index * 2;
    }

    public bool contains(int i) {
        return i % 2 == 0;
    }

    public string to_string() {
        return "[Данный объект перечисляет четные числа]";
    }

    public Iterator iterator() {
        return new Iterator(this);
    }

    public class Iterator {
        private int index;
        private EvenNumbers even;

        public Iterator(EvenNumbers even) {
            this.even = even;
        }

        public bool next() {
            return true;
        }

        public int get() {
            this.index++;
            return this.even[this.index - 1];
        }
    }
}

void main() {
    var even = new EvenNumbers();
    stdout.printf("%d\n", even[5]);   // get()
    if (4 in even) {                  // contains()
        stdout.printf(@"$even\n");    // to_string()
    }
    foreach (int i in even) {         // iterator()
        stdout.printf("%d\n", i);
        if (i == 20) break;
    }
}

```

