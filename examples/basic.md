# Basic

## Std

Std means GLib

### Hello world

```csharp
print ("hello, world\n"); 
//If you don't need to allocate memory in the heap then main is not needed
```

### User Input <a id="Reading_User_Input"></a>

```csharp
stdout.printf (@"Your name is : $(stdin.read_line ())");
```

### Mathematics <a id="Mathematics"></a>

```csharp
stdout.printf ("Please enter the radius of a circle: ");
double radius = double.parse (stdin.read_line ());
stdout.printf ("Circumference: %g\n", 2 * Math.PI * radius);
stdout.printf ("sin(pi/2) = %g\n", Math.sin (Math.PI / 2));
```

### Random Numbers

```csharp
var rand = Random.int_range (1, 49);
```

### Reading and Writing Text File Content <a id="Reading_and_Writing_Text_File_Content"></a>

```csharp
// Writing
string content = "hello, world";
FileUtils.set_contents ("data.txt", content);
// Reading
string read;
FileUtils.get_contents ("data.txt", out read);
```

### Spawning Processes <a id="Spawning_Processes"></a>

```csharp
string output;
Process.spawn_command_line_sync ("ls", ref output); // now output is result of ls
```

### Timer

```csharp
Timer timer = new Timer ();
stdout.printf (@"$(timer.elapsed()) sec elapsed");// 0 sec elapsed
```

### Signals with data \(Qt like\)

```csharp
public class SignalTest: Object {
    public signal void sig_1(int a);
    string to_string(){ return "hello from SignalTest\n";}
}
void main() {
    var t1 = new SignalTest();
    t1.sig_1.connect((t, a) => {
        //The first argument is always the object that caused the signal.
        print(@"$a $t\n");
    });
    t1.sig_1(5);
}
```

### Native Regular Expression Literals

```csharp
string email = "tux@kernel.org";
if (/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}$/i.match(email))
    stdout.printf("Valid email address\n");

var r = /(foo|bar|cow)/;
var o = r.replace ("this foo is great", -1, 0, "thing");
print (o);//this thing is great

```

### Class

Классы выделяются на куче.

Memory for classes is allocated on the Heap.

```csharp
class Klass{
    public int a {private set; owned get; default = 3;}
    public int? b = 3; //? means it may be null|=3 same as default = 3
    public string c;
    public inline void hello() {print("Hello");}
    signal void sig_1(int a);// Event with data
    public Klass(){print("usual constructor\n");}
    public Klass.with_a(int _a){a = _a;}//named constructor
}

void main () {
    var klass1 = new Klass(){a = 5,b = 6,c = "param named constructor"};
    var klass2 = new Klass.with_a(5);
}
```

GObject Constructor Class

```csharp
public class Person : Object {
    /* Construction properties */
    public string name { get; construct; }
    public int age { get; construct set; }
    public Person(string name) {
        Object(name: name);
    }
    public Person.with_age(string name, int years) {
        Object(name: name, age: years);
    }
    construct {
        // do anything else
        stdout.printf(@"Welcome $name\n");
    }
}
void main () {
    var klass1 = new Person("Mark");//output: Welcome Mark
    var klass2 = new Person.with_age("Mark",20);//output: Welcome Mark
}
```

### Structs

Структуры выделяются на стеке  
Memory for structs is allocated on the Stack.

```csharp
struct Klass{
    int a;
    int b;
    public Klass(){print("Constructor\n");}
}
struct Sas: Klass{
    public Sas.with_args(int _a = 2, int _b = 3){a = _a; b = _b;}
    public int sum(){return base.a + base.b;}//base here mean Klass
}
void main () {
    Sas sas = {5,6};
    var sus = Sas() {a = 5,b = 6};
    var ses = Sas.with_args(5,6);
    prin(sas.sum()," ",sus.sum()," ",ses.sum());
}
```

## Gpseq

### Parallel sorting 

```csharp
Gee.List<int> list = Seq.iterate<int>(9999, i => i >= 0, i => --i)
                        .parallel()
                        .order_by()
                        .collect( to_list<int>() )
                        .value;

for (int i = 0; i < 5; i++) {
    print("%d ", list[i]);
}
```

### Simple Parallel Data Processing

```csharp
using Gpseq;

void main () {
    string[] array = {"dog", "cat", "pig", "boar", "bear"};
    Seq.of_array<string>(array)
        .parallel()
        .filter(g => g.length == 3)
        .map<string>(g => g.up())
        .foreach(g => print("%s\n", g))
        .wait();
}
```

### GoLang like channels

```csharp
using Gpseq;

void main () {
    Channel<string> chan = Channel.bounded<string>(0);
    run(() => { chan.send("ping").ok(); });
    print("%s\n", chan.recv().value);
}
```

### GoLang like Managed Blocked Parallelism



```text
const ulong SEC = 1000000;

TaskEnv.apply(new CustomTaskEnv(), () => {
    Seq.iterate<int>(0, i => i < 100, i => ++i)
       .parallel()
       .foreach(g => {
           blocking(() => {
               Thread.usleep(1 * SEC);
           });
       })
       .wait();
});
```

If you comment out the `blocking` block and put only `Thread.usleep` then the program will run for a time equal to the number of running threads divided by the number of threads of your processor.

With `blocking`, it is equal to the execution time of a single thread, that is, one second. This is called `work stealing`. When a thread starts waiting for something, such as a packet from the network, another thread steals its CPU time and starts working until the first thread waits for the event it needs.

Если вы закомментируете `blocking`  блок и оставив только `Thread.usleep`, то программа будет работать в течение времени, равного числу запущенных потоков, деленному на число потоков вашего процессора.

C `blocking` оно равно времени выполнения одного потока, то есть одной секунде. Это называется `work stealing`. Когда поток начинает чего то ожидать, например пакета из сети, другой поток ворует у него процессорное время и начинает работать, пока первый не дождется нужного ему события.

## GObject

### 3 Types of classes

Vala supports three different types of class:

* GObject subclasses are any classes derived directly or indirectly from GLib.Object. This is the most powerful type of class, supporting all features described in this page. This means signals, managed properties, interfaces and complex construction methods, plus all features of the simpler class types.
* Fundamental GType classes are those either without any superclass or that don't inherit at any level from GLib.Object. These classes support inheritence, interfaces, virtual methods, reference counting, unmanaged properties, and private fields. They are instantiated faster than GObject subclasses but are less powerful - it isn't recommended in general to use this form of class unless there is a specific reason to.
* Compact classes, so called because they use less memory per instance, are the least featured of all class types. They are not registered with the GType system and do not support reference counting, virtual methods, or private fields. They do support unmanaged properties. Such classes are very fast to instantiate but not massively useful except when dealing with existing libraries. They are declared using the Compact attribute on the class, See

Memory for Compact classes is allocated by [this ](https://developer.gnome.org/glib/stable/glib-Memory-Slices.html)monster.

```csharp
//The slowest class. 
//Supports introspection of property and functions as well
// as thread-safe signals to property.
public class GObjectKlassExample : Object {
}
public class GTypeKlassExample{
}
[Compact]
public class FastestKlassExample {
}
```

### Run-Time Type Information

```csharp
bool b = object is SomeTypeName;// run time type check
Type type = object.get_type();// run time get object type name
print(type.name());
```

### Dynamic Type Casting

```csharp
class Alpha {
    public int a;
    public int a2;
}

class Beta : Alpha {
    public new int a;
}

void main() {
    var b = new Beta();
    
    b.a = 3;   // accesses field Beta.a
    b.a2 = 4;  // accesses field Alpha.a2
    (b as Alpha).a = 5; // accesses field Alpha.a Also run time cast
}
```



