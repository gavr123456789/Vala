# Config file

Иногда может понадобится чтобы некоторые константы подставлялись в код перед компиляцией. 

Это пример такого поведения:

### prog.vala

```csharp
class MainProg : GLib.Object {

    public static int main(string[] args) {
        stdout.printf("DATA_DIRECTORY is: %s.\n", DATA_DIRECTORY);
        stdout.printf("SOMETHING_ELSE is: %s.\n", SOMETHING_ELSE);
        return 0;
    }
}
```

### meson build

```csharp
project('valatest', 'vala', 'c')

valac = meson.get_compiler('vala')
# Try to find our library
valadeps = [valac.find_library('meson-something-else', dirs : meson.current_source_dir())]
valadeps += [dependency('glib-2.0'), dependency('gobject-2.0')]

e = executable(
'valaprog',
sources : ['config.vapi', 'prog.vala'],
dependencies : valadeps,
c_args : ['-DDATA_DIRECTORY="@0@"'.format(meson.current_source_dir()),
          '-DSOMETHING_ELSE="Out of this world!"']
)
test('valatest', e)
```

Как можно заметить значения аргументов выставляются через флаги для С компилятора 

```csharp
-DDATA_DIRECTORY="@0@".format(meson.current_source_dir())
-DSOMETHING_ELSE="Out of this world!"
```

Значение между собаками подставляется через функцию строки format.

### **meson-something-else.vapi** <a id="blob-path"></a>

```csharp
public const string SOMETHING_ELSE;
```

### **config.vapi** <a id="blob-path"></a>

```csharp
public const string DATA_DIRECTORY;
```

