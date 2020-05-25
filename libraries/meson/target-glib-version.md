# Target GLib Version

Вы можете указывать нужную вам версию зависимостей через version параметр.

### **retcode.c** <a id="blob-path"></a>

```c
int
get_ret_code (void)
{
  return 42;
}
```

### **meson.build** <a id="blob-path"></a>

```csharp
project('valatest', 'vala', 'c')

if not meson.is_unity()
  add_global_arguments('-Werror', language : 'c')
endif

valadeps = [dependency('glib-2.0', version : '>=2.32'), dependency('gobject-2.0')]

e = executable('valaprog', 'GLib.Thread.vala', 'retcode.c', dependencies : valadeps)
test('valatest', e)
```

### **GLib.Thread.vala** <a id="blob-path"></a>

```csharp
extern int get_ret_code ();

public class MyThread : Object {
    public int x_times { get; private set; }

    public MyThread (int times) {
        this.x_times = times;
    }

    public int run () {
        for (int i = 0; i < this.x_times; i++) {
            stdout.printf ("ping! %d/%d\n", i + 1, this.x_times);
            Thread.usleep (10000);
        }

        // return & exit have the same effect
        Thread.exit (get_ret_code ());
        return 43;
    }
}

public static int main (string[] args) {
    // Check whether threads are supported:
    if (Thread.supported () == false) {
        stderr.printf ("Threads are not supported!\n");
        return -1;
    }

    try {
        // Start a thread:
        MyThread my_thread = new MyThread (10);
        Thread<int> thread = new Thread<int>.try ("My fst. thread", my_thread.run);

        // Wait until thread finishes:
        int result = thread.join ();
        // Output: `Thread stopped! Return value: 42`
        stdout.printf ("Thread stopped! Return value: %d\n", result);
    } catch (Error e) {
        stdout.printf ("Error: %s\n", e.message);
    }

    return 0;
}
```

\*\*\*\*

