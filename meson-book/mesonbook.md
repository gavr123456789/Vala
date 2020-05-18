# MesonBook

Советую посмотреть вот эту презентацию чтобы лучше понимать зачем стоит использовать Meson. [https://youtu.be/SCZLnopmYBM](https://youtu.be/SCZLnopmYBM)

Вот список каких то фич:

### Features <a id="features"></a>

* multiplatform support for Linux, macOS, Windows, GCC, Clang, Visual Studio and others
* supported languages include C, C++, D, Fortran, Java, Rust, Vala, C\#
* build definitions in a very readable and user friendly non-Turing complete DSL
* cross compilation for many operating systems as well as bare metal
* optimized for extremely fast full and incremental builds without sacrificing correctness
* built-in multiplatform dependency provider that works together with distro packages
* fun!

Вообщем лучше всего Meson описывает третий пункт, это DSL для сборок. 

## Hello world

Начнем с самого простого, у вас 1 файл, который является программой которую можно запустить, а не библиотекой. 

#### source

{% tabs %}
{% tab title=".c" %}
```c
#include<stdio.h>

int main(int argc, char **argv) {
  printf("Hello there.\n");
  return 0;
}
```
{% endtab %}

{% tab title=".vala" %}
```cpp
print("Hello world!");
```
{% endtab %}

{% tab title=".swift" %}
```swift
print("Hello world!")
```
{% endtab %}

{% tab title=".d" %}
```d
import std.stdio;

void main(){
    writeln("Hello World!");
}
```
{% endtab %}

{% tab title=".cs" %}
```csharp
using System;

public class Prog {
    static public void Main () {
        Console.WriteLine("Hello World!");
    }
}
```
{% endtab %}

{% tab title=".java" %}
```java
package com.mesonbuild;

class Simple {
    public static void main(String [] args) {
        System.out.println("Java is working.\n");
    }
}
```
{% endtab %}

{% tab title=".f90" %}
```ocaml
print *, "Fortran compilation is working."
end program
```
{% endtab %}

{% tab title=".m" %}
```objectivec
#import<stdio.h>

int main(void) {
    return 0;
}
```
{% endtab %}

{% tab title=".rs" %}
```rust
fn main() {
    println!("rust compiler is working");
}
```
{% endtab %}

{% tab title=".cpp" %}
```cpp
#include<iostream>

int main(void) {
  std::cout << "Hello World" << std::endl;
  return 0;
}
```
{% endtab %}

{% tab title=".cu" %}
```cpp
#include <iostream>

int main(void) {
    int cuda_devices = 0;
    std::cout << "CUDA version: " << CUDART_VERSION << "\n";
    cudaGetDeviceCount(&cuda_devices);
    if(cuda_devices == 0) {
        std::cout << "No Cuda hardware found. Exiting.\n";
        return 0;
    }
    std::cout << "This computer has " << cuda_devices << " Cuda device(s).\n";
    cudaDeviceProp props;
    cudaGetDeviceProperties(&props, 0);
    std::cout << "Properties of device 0.\n\n";

    std::cout << "  Name:            " << props.name << "\n";
    std::cout << "  Global memory:   " << props.totalGlobalMem << "\n";
    std::cout << "  Shared memory:   " << props.sharedMemPerBlock << "\n";
    std::cout << "  Constant memory: " << props.totalConstMem << "\n";
    std::cout << "  Block registers: " << props.regsPerBlock << "\n";

    std::cout << "  Warp size:         " << props.warpSize << "\n";
    std::cout << "  Threads per block: " << props.maxThreadsPerBlock << "\n";
    std::cout << "  Max block dimensions: [ " << props.maxThreadsDim[0] << ", " << props.maxThreadsDim[1]  << ", " << props.maxThreadsDim[2] << " ]" << "\n";
    std::cout << "  Max grid dimensions:  [ " << props.maxGridSize[0] << ", " << props.maxGridSize[1]  << ", " << props.maxGridSize[2] << " ]" << "\n";
    std::cout << "\n";

    return 0;
}
```
{% endtab %}

{% tab title=".v" %}
```groovy
module top(input clk, output LED1, output LED2, output LED3, output LED4, output LED5);
   
   reg ready = 0;
   reg [23:0] divider;
   reg [3:0] spin;
   
   always @(posedge clk) begin
      if (ready) 
        begin
           if (divider == 6000000) 
             begin
                divider <= 0;
                spin <= {spin[2], spin[3], spin[0], spin[1]};
             end
           else 
             divider <= divider + 1;
        end
      else 
        begin
           ready <= 1;
           spin <= 4'b1010;
           divider <= 0;
        end
   end
   
   assign LED1 = spin[0];
   assign LED2 = spin[1];
   assign LED3 = spin[2];
   assign LED4 = spin[3];
   assign LED5 = 1;
endmodule

```
{% endtab %}

{% tab title=".pcf" %}
```
set_io LED1 99
set_io LED2 98
set_io LED3 97
set_io LED4 96
set_io LED5 95
set_io clk 21
```
{% endtab %}
{% endtabs %}

#### meson.build

{% tabs %}
{% tab title="C" %}
```csharp
project('simple', 'c')
executable('myexe', 'source.c')
```
{% endtab %}

{% tab title="Vala" %}
```csharp
project('vala app', 'vala', 'c')
executable('app', 'app.vala', dependencies: dependency('glib-2.0'))
```
{% endtab %}

{% tab title=" Swift" %}
```csharp
project('swift exe', 'swift')
executable('swifttest', 'prog.swift')
```
{% endtab %}

{% tab title="D" %}
```csharp
project('myapp', 'd')
executable('myapp', 'app.d')
```
{% endtab %}

{% tab title="CS" %}
```csharp
project('simple c#', 'cs')
executable('prog', 'app.cs')
```
{% endtab %}

{% tab title="Java" %}
```csharp
project('simplejava', 'java')
jar('myprog',  'com/mesonbuild/Simple.java',
  main_class : 'com.mesonbuild.Simple')

#Проверить версию компилятора
jc = meson.get_compiler('java')
message(jc.get_id())
message(jc.get_linker_id())
```
{% endtab %}

{% tab title="F90" %}
```csharp
project('simple fortran', 'fortran')

fc = meson.get_compiler('fortran')
if fc.get_id() == 'gcc'
  add_global_arguments('-fbounds-check', language : 'fortran')
endif

args = fc.first_supported_argument(['-ffree-form', '-free', '/free'])
assert(args != [], 'No arguments found?')

executable('simple', 'app.f90',
  fortran_args : args)
```
{% endtab %}

{% tab title="Objc" %}
```csharp
project('objective c', 'objc')
exe = executable('prog', 'app.m')
```
{% endtab %}

{% tab title="Rust" %}
```csharp
project('rustprog', 'rust')
executable('program', 'app.rs')
```
{% endtab %}

{% tab title="cpp" %}
```csharp
project('emcctest', 'cpp')
executable('hello', 'hello.cpp')
```
{% endtab %}

{% tab title="cppwasm.txt" %}
```elixir
[binaries]
cpp = '/home/jpakkane/emsdk/fastcomp/emscripten/em++'

[properties]
cpp_args = ['-s', 'WASM=1', '-s', 'EXPORT_ALL=1']
cpp_link_args = ['-s', 'EXPORT_ALL=1']

[host_machine]
system = 'emscripten'
cpu_family = 'wasm32'
cpu = 'wasm32'
endian = 'little'
```
{% endtab %}

{% tab title="Cuda" %}
```cpp
project('simple', 'cuda', version : '1.0.0')
executable('prog', 'prog.cu')
```
{% endtab %}

{% tab title="fpga" %}
```csharp
project('lattice', 'c')

is = import('unstable_icestorm')

is.project('spin',
  'spin.v',
  constraint_file : 'spin.pcf',
)
```
{% endtab %}
{% endtabs %}

Даже не уверен стоит ли тут что-то пояснять, в этом вся суть Meson. 

#### project

C этой строчки начинается любой проект, первый аргумент название проекта а затем перечисление языков которые используются в проекте \(Например так как Vala компилируется в C для нее перечислены оба языка\)

#### executable

executable это цель этого билд скрипта, в одном билдскрипте их может быть сколько угодно. Первый аргумент название, второй список файлов. 

#### Build

Это всё, теперь мы готовы забилдить наше приложение. Сначала нам нужно инициализировать сборку, перейдя в исходный каталог и выполнив:  
`meson build` 

Мы создаем отдельный каталог сборки, чтобы содержать весь вывод компилятора. Meson отличается от некоторых других систем сборки тем, что он не допускает in-source builds. Вы всегда должны создать отдельный каталог сборки. Общепринятой нормой считается размещение build каталога в корневой директории вашего проекта.

```cpp
❯ meson build
The Meson build system
Version: 0.54.1
Source dir: /home/gavr/Dev/Meson
Build dir: /home/gavr/Dev/Meson/build
Build type: native build
Project name: simple
Project version: undefined
C compiler for the host machine: cc (gcc 9.3.0 "cc (Arch Linux 9.3.0-1) 9.3.0")
C linker for the host machine: cc ld.bfd 2.34
Vala compiler for the host machine: valac (valac 0.48.5)
Host machine cpu family: x86_64
Host machine cpu: x86_64
Found pkg-config: /usr/bin/pkg-config (1.6.3)
Run-time dependency glib-2.0 found: YES 2.64.2
Build targets in project: 1

Found ninja-1.10.0 at /usr/bin/ninja
```

ninja -C build

Meson использует ninja в качестве бэкенда. 

> **Ninja** - это небольшая система сборки с акцентом на скорость. Она отличается от других систем сборки в двух основных аспектах: она предназначен для того, чтобы её входные файлы создавались системой сборки более высокого уровня, и она предназначен для запуска сборок с максимальной скоростью.
>
> По сути, Ninja предназначена для замены Make, которая медлителен при выполнении инкрементных \(или no-op\) сборок. Это может значительно замедлить работу разработчиков над большими проектами, такими как Google Chrome, который компилирует 40 000 входных файлов в один исполняемый файл. На самом деле Google Chrome - это основной пользователь и мотивация для создания ninja. Она также используется для сборки Android и большинством разработчиков, работающих над LLVM.

```cpp
❯ ninja -C build
ninja: Entering directory `build'
[3/3] Linking target myexe
```

{% hint style="info" %}
Флаг -C указывает название директории, аналогично cd build && ninja
{% endhint %}

### Несколько файлов

{% tabs %}
{% tab title="C" %}
```cpp
project('manyfiles', 'c')
src = ['source1.c', 'source2.c', 'source3.c']
executable('myexe', src)
```
{% endtab %}

{% tab title="Vala" %}
```csharp
project('manyfiles', 'vala', 'c')
src = ['source1.vala', 'source2.vala', 'source3.vala']
executable('myexe', src,  dependencies: dependency('glib-2.0'))
```
{% endtab %}
{% endtabs %}

src является переменной, которая содержит массив файлов, затем мы передаем его в executable.

Аргументы могут быть именованными, чаще всего люди используют их именно так

```cpp
exe = executable('myexe', sources : src, dependencies: deps)
```

### Тесты

```cpp
test('simple test', exe)
```

Первый аргумент название теста, второй executable target.

Команда `ninja -C build test` запустит все тесты проекта

{% hint style="info" %}
Meson не заставляет использовать какой-либо конкретный тест-фреймворк. Вы можете свободно использовать GTest\(GLib\), Boost Test, Check или даже пользовательские исполняемые файлы.
{% endhint %}

### Зависимости

Просто выводить текст в консоль слишком скучно, переключаемся на GTK. 

#### source

{% tabs %}
{% tab title="С" %}
```cpp
#include<gtk/gtk.h>

int main(int argc, char **argv) {
  GtkWidget *win;
  gtk_init(&argc, &argv);
  win = gtk_window_new(GTK_WINDOW_TOPLEVEL);
  gtk_window_set_tiну tle(GTK_WINDOW(win), "Hello from C!");
  g_signal_connect(win, "destroy", G_CALLBACK(gtk_main_quit), NULL);
  gtk_widget_show(win);
  gtk_main();                            //run gtk main loop
}
```
{% endtab %}

{% tab title="Vala" %}
```cpp
using Gtk;
 
void main(string[] args){
    Gtk.init(ref args);
    var window = new Window(){ title = "Hello from Vala!"}; 
    window.destroy.connect(Gtk.main_quit); 
    window.show_all();
    Gtk.main();                            
}
```
{% endtab %}

{% tab title="D" %}
```d
import std.stdio;

import gtk.MainWindow;
import gtk.Main;
import gtk.Widget;

void main(string[] args){
	Main.init(args);
	auto window = new MainWindow("Hello from D!");
	window.addOnDestroy(delegate void(Widget w) { Main.quit();});
	window.showAll();
	Main.run();
}
```
{% endtab %}
{% endtabs %}

#### meson.build

{% tabs %}
{% tab title="C" %}
```cpp
project('gtkapp', 'c')
src = ['app.c']
gtk = dependency('gtk+-3.0')
exe = executable('demo', 'app.c', dependencies : gtk)
test('test', exe)
```
{% endtab %}

{% tab title="Vala" %}
```cpp
project('gtkapp', 'vala', 'c')
src = ['app.vala']
gtk = dependency('gtk+-3.0')
exe = executable('demo', src, dependencies: gtk)
test('test', exe)
```
{% endtab %}

{% tab title="D" %}
```csharp
project('gtkapp', 'd')
src = ['app.d']
gtkd = dependency ('gtkd-3')
exe = executable('myexe', src,  dependencies: gtkd)
test('test', exe)
```
{% endtab %}
{% endtabs %}

Объяснять буквально нечего, dependencies работают также как sources.

{% hint style="info" %}
После того как вы настроили свой каталог сборки в первый раз, вам больше никогда не нужно будет запускать команду meson. Meson автоматически определит, когда вы внесли изменения в определения сборки, и позаботится обо всем, чтобы пользователям не пришлось беспокоиться.
{% endhint %}

![](../.gitbook/assets/image%20%2818%29.png)



