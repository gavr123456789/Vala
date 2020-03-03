# Basic

{% hint style="info" %}
All examples except gpseq are run and compiled with the vala filename command.vala
{% endhint %}

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

### OS Specific

```csharp
Environment.get_current_dir();
Environment.get_real_name();
Environment.get_system_config_dirs();
Environment.get_user_config_dir();
Environment.get_application_name();
Environment.get_home_dir();
...
//Windows specific
Win32.is_nt_based();
Win32.OSType.SERVER;//enum
Win32.get_windows_version();
Win32.getlocale();
Win32.get_package_installation_directory_of_module(hmodule);//hmodule is pointer
...
//Linux specific
Linux.Network.ifMap.port;
Linux. anything from linux.h;
```

### Network

Get Twitter Status

#### Soup Library

```csharp
using Soup;
void main () {
    // add your twitter username
    string username = "gnome";
    // format the URL to use the username as the filename
    string url = "http://twitter.com/users/%s.xml".printf (username);
    // create an HTTP session to twitter
    var session = new Soup.Session ();
    var message = new Soup.Message ("GET", url);
    // send the HTTP request and wait for response
    session.send_message (message);
    // output the XML result to stdout 
    stdout.write (message.response_body.data);
}// to run: vala --pkg libsoup-2.4 filename
```

[more soup examples](https://wiki.gnome.org/Projects/Vala/LibSoupSample)

#### GIO \(Vala standart input output library\)

Async Server Example

```csharp
bool on_incoming_connection (SocketConnection conn) {
    stdout.printf ("Got incoming connection\n");
    // Process the request asynchronously
    process_request.begin (conn);// u can call .begin on async funcs
    return true;
}
// Note the async keyword
async void process_request (SocketConnection conn) {
    try {
        var dis = new DataInputStream (conn.input_stream);
        var dos = new DataOutputStream (conn.output_stream);
        string req = yield dis.read_line_async (Priority.HIGH_IDLE);
        dos.put_string ("Got: %s\n".printf (req));
    } catch (Error e) {
        stderr.printf ("%s\n", e.message);
    }
}

void main () {
    try {
        var srv = new SocketService ();
        srv.add_inet_port (3333, null);
        srv.incoming.connect (on_incoming_connection);
        srv.start ();
        new MainLoop ().run ();
    } catch (Error e) {
        stderr.printf ("%s\n", e.message);
    }
}
//
```

{% hint style="info" %}
Connect to localhost via netcat or telnet on port 3333 and issue a command ending with a newline. echo "blub" \| nc localhost 3333
{% endhint %}

[more network GIO examples](https://wiki.gnome.org/Projects/Vala/GIONetworkingSample)

### Signals with data \(Qt like\)

```csharp
public class SignalTest: Object {
    public signal void sig_1(int a);
    string to_string(){ return "hello from SignalTest\n";}
}
//The first argument is always the object that caused the signal.
void main() {
    var t1 = new SignalTest();
    t1.sig_1.connect((t, a) => { print(@"$a $t\n"); });
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

{% hint style="info" %}
Классы выделяются на куче.  
Memory for classes is allocated on the Heap.
{% endhint %}

Things u can do with classes:

```csharp
class Klass{
    public int a {private set; owned get; default = 3;}// property
    public int? b = 3; //? means it may be null|=3 same as default = 3
    public string c;
    public inline void hello() {print("Hello");}//inline
    signal void sig_1(int a);// Event with data
    public Klass(){print("usual constructor\n");}
    public Klass.with_a(int _a){a = _a;}//named constructor
}

void main () {
    var klass1 = new Klass(){a = 5, b = 6, c = "param named constructor"};
    var klass2 = new Klass.with_a(5);
}
```

{% hint style="info" %}
owned get means that the object will own the link after the transfer, that is, it is like move semantics in C++
{% endhint %}

{% hint style="info" %}
In Vala, as in Go, there is no function overload, but you can use named constructors and default arguments

В Vala как и в Go нету перегрузки функций, но можно использовать именованные конструкторы и аргументы по умолчанию

On the 11th line, you can see the creation of a class with some arguments specified without a constructor. This is similar to named function arguments.

На 11той строке вы можете увидеть создание класса с заданием некоторых аргументов без конструктора. Это похоже на именованные аргументы функций.
{% endhint %}

### GObject Constructor Class 

```csharp
using Gtk;
class SpecialBtn : Button {
    construct{
        label = "Thats btn!";
        clicked.connect(on_btn_clicked);
    }
    void on_btn_clicked(){
        print("Button clicked test!");
    }
}
class HelloWorld : Window {
    construct{
        title = "Hello World Window";
        window_position = WindowPosition.CENTER;
        default_width = 200;
        default_height = 300;
        resizable = false;
        add(new SpecialBtn());
        show_all();
        destroy.connect(Gtk.main_quit);//close btn -> close app
    }
}

void main(string[] args) {
    init(ref args);
    var app = new HelloWorld();
    Gtk.main();
}// run: $ vala filename --pkg gtk+-3.0
```

### Structs

{% hint style="info" %}
Структуры выделяются на стеке  
Memory for structs is allocated on the Stack.
{% endhint %}

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

## DBus

### Server

```csharp
[DBus (name = "org.example.Demo")]
public class DemoServer : Object {
    private int counter;
    public int ping (string msg) {
        stdout.printf ("%s\n", msg);
        return counter++;
    }
    public int ping_with_signal (string msg) {
        stdout.printf ("%s\n", msg);
        pong(counter, msg);
        return counter++;
    }
    public int ping_with_sender (string msg, GLib.BusName sender) {
        stdout.printf ("%s, from: %s\n", msg, sender);
        return counter++;
    }
    public void ping_error () throws Error {
        throw new DemoError.SOME_ERROR ("There was an error!");
    }
    public signal void pong (int count, string msg);
}
[DBus (name = "org.example.DemoError")]
public errordomain DemoError{
    SOME_ERROR
}

void on_bus_aquired (DBusConnection conn) {
    try {
        conn.register_object ("/org/example/demo", new DemoServer ());
    } catch (IOError e) {
        stderr.printf ("Could not register service\n");
    }
}
void main () {
    Bus.own_name (BusType.SESSION, "org.example.Demo", BusNameOwnerFlags.NONE,
                  on_bus_aquired,
                  () => {},
                  () => stderr.printf ("Could not aquire name\n"));
    new MainLoop ().run ();
}
```

### Client

```csharp
[DBus (name = "org.example.Demo")]
interface Demo : Object {
    public abstract int ping (string msg) throws IOError;
    public abstract int ping_with_sender (string msg) throws IOError;
    public abstract int ping_with_signal (string msg) throws IOError;
    public signal void pong (int count, string msg);
}

void main () {
    /* Needed only if your client is listening to signals; you can omit it otherwise */
    var loop = new MainLoop();
    /* Important: keep demo variable out of try/catch scope not lose signals! */
    Demo demo = null;
    try {
        demo = Bus.get_proxy_sync (BusType.SESSION, "org.example.Demo",
                                                    "/org/example/demo");
        /* Connecting to signal pong! */
        demo.pong.connect((c, m) => {
            stdout.printf ("Got pong %d for msg '%s'\n", c, m);
            loop.quit ();
        });

        int reply = demo.ping ("Hello from Vala");
        stdout.printf ("%d\n", reply);

        reply = demo.ping_with_sender ("Hello from Vala with sender");
        stdout.printf ("%d\n", reply);

        reply = demo.ping_with_signal ("Hello from Vala with signal");
        stdout.printf ("%d\n", reply);

    } catch (IOError e) {
        stderr.printf ("%s\n", e.message);
    }
    loop.run();
}
```

## HTML5 Generate

[Compose](https://github.com/arteymix/compose) - Functional templating for Vala.

```csharp
using Compose;
using Compose.HTML5;

var app = new Valum.Router ();

app.get ("/users", () => {
  var users = User.all ();
  return res.expand_utf8 (
    html ({},
      head ({},
        title ("My really amazing page"),
        link ("/static/style.css")
      ),
      body ({"lang=en"},
        section ({"class=users"},
          h2 ({}, "Users"),
          take<User> (()     => users.next (),
                      (user) => p (user.username)))
      )
    )
  );
});
```

## Gpseq \(C\# Linq/Java Stream API, Go lang Chanels and Parallelism\)

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

#### simple example

```csharp
using Gpseq;

void main () {
    Channel<string> chan = Channel.bounded<string>(0);
    run(() => { chan.send("ping").ok(); });
    print("%s\n", chan.recv().value);
}
```

#### Complex example

Sum using divide and conquer:

```csharp
using Gpseq;
void main () {
    int[] array = {1, 2, 3, 4, 5, 6};
    Channel<int> chan = Channel.bounded<int>(0);
    run(() => sum(array[0:array.length/2], chan));
    run(() => sum(array[array.length/2:array.length], chan));
    print_sum(chan);
}
void sum (int[] arr, Sender<int> ch) {
    int sum = 0;
    for (int i = 0; i < arr.length; ++i) sum += arr[i];
    ch.send(sum).ok();
}
void print_sum (Receiver<int> ch) {
    int left = ch.recv().value;
    int right = ch.recv().value;
    print(@"$left $right $(left + right)\n");
}//result = left: 6(1+2+3) right: 15 left+right: 21
```

### GoLang like Managed Blocked Parallelism

```csharp
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

![Benchmark for creating and deleting 100K classes of three types](../.gitbook/assets/image%20%289%29.png)

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



