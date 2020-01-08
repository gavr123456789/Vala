# Change Log

## Релиз Meson 0.53 !

### Новый модуль fs

Например для проверки файлов на существование: 

```javascript
fs = import('fs')
assert(fs.exists('important_file'),
       'The important file is missing.')
```

### Новый флаг --include-subprojects для команды meson dist

Включает все подпроекты\(wrap\) в конечный tarball для команды [dist](https://mesonbuild.com/Creating-releases.html)\(создает .tar.xz релиз исключая git метаданные и проводя полный цикл компиляции + тест + установка  + sha-256 контрольные суммы перед запаковкой\)   
Это может быть полезно для self contained пакетов с модом --wrap-mode=nodownload

### Добавлены шаблоны проектов для D, Rust, Objective-C! 

Команда `meson init --language` сгенерирует новый проект с Meson

### Новая функция summary\(\) для подведения итогов в конце сборки

```javascript
project('My Project', version : '1.0')
summary({'bindir': get_option('bindir'),
         'libdir': get_option('libdir'),
         'datadir': get_option('datadir'),
        }, section: 'Directories')
summary({'Some boolean': false,
         'Another boolean': true,
         'Some string': 'Hello World',
         'A list': ['string', 1, true],
        }, section: 'Configuration')
```

Output:

```javascript
My Project 1.0

  Directories
             prefix: /opt/gnome
             bindir: bin
             libdir: lib/x86_64-linux-gnu
            datadir: share

  Configuration
       Some boolean: False
    Another boolean: True
        Some string: Hello World
             A list: string
                     1
                     True
```

### Универсальный Overrider для выбора динамического компоновщика

До meson 0.52.0 вам нужно было устанавливаливать динамический компоновщик, используя специфичные для компилятора флаги, передаваемые через флаги компилятору языки, и надеяться, что все получится.

В meson 0.52.0 meson научился обнаруживать компоновщик и принимать разумные решения по его использованию. К сожалению, это не помогло в выбре компоновщика по умолчанию. Теперь для этого есть общий механизм: вы можете использовать переменную среды LD \(с обычными правилами переменных среды meson\) или добавить следующее в кросс-файл:

```javascript
[binaries]
ld = 'gold'
```

И Meson выберет компоновщик автоматически, если это возможно.

### Поддержка стандартов фортран \(вплоть до f2018\) в виде переменных fortran\_std

### Scalapack 

```javascript
scalapack = dependency('scalapack')
```

Исторически и до сегодняшнего дня сложилось что Scalapack сетап сломан и не находится с помощью `pkg-config` или `FindScalapack.cmake`   
Теперь Meson может найти Scalapack на следующих сетапах

* Linux: Intel MKL or OpenMPI + Netlib
* MacOS: Intel MKL or OpenMPI + Netlib
* Windows: Intel MKL \(OpenMPI not available on Windows\)

### Поиск по каталогам для `find_program()` <a id="search-directories-for-find_program"></a>

Теперь можно дать список абсолютных путей, по которым `find_program()`следует также искать, используя `dirs`ключевое слово аргумент.

Например, в Linux `/sbin`и `/usr/sbin`не всегда в `$PATH`:

```javascript
prog = find_program('mytool', dirs : ['/usr/sbin', '/sbin'])
```

### Tags для целей сборки <a id="source-tags-targets"></a>

Если соответствующие инструменты доступны, Meson будет генерировать цели 'ctags', 'TAGS' и 'cscope', если вы не определили свои собственные.

### Словари с использованием  строковой переменной в качестве ключа <a id="dictionary-entry-using-string-variable-as-key"></a>

Ключами теперь могут быть любые выражения, имеющие строковое значение, больше не ограничиваясь строковыми литералами.

```javascript
d = {'a' + 'b' : 42}
k = 'cd'
d += {k : 43}
```

### Улучшена поддержка подпроектов CMake <a id="improved-cmake-subprojects-support"></a>

В этом выпуске еще больше проектов CMake поддерживаются через [CMake subprojects](https://mesonbuild.com/CMake-module.html#cmake-subprojects) благодаря следующим внутренним улучшениям:

* Использование API файла CMake для CMake &gt; = 3.14
* Обрабатывание явных зависимостей через `add_dependency`
* Базовая поддержка для `add_custom_target`
* Улучшенна поддержка`add_custom_command`
* Поддержка библиотеки объектов в Windows \(?\)

### compiler.get\_linker\_id \(\) <a id="compilerget_linker_id"></a>

начиная с 0.53.0, `compiler.get_linker_id()`позволяет получить имя компоновщика в нижнем регистре. Поскольку в каждом семействе компиляторов обычно используются различные компоновщики, в зависимости от операционной системы, это помогает пользователям определять логику для угловых случаев, которые иначе сложно обработать.

### Зависимость CUDA <a id="cuda-dependency"></a>

Встроенная поддержка компиляции и связывания с CUDA Toolkit с использованием `dependency`функции:

```javascript
project('CUDA test', 'cpp', meson_version: '>= 0.53.0')
exe = executable('prog', 'prog.cc', dependencies: dependency('cuda'))
```

См. [Зависимость CUDA](https://mesonbuild.com/Dependencies.html#cuda) для получения дополнительной информации.

### Добавлена ​​глобальная опция для отключения C ++ RTTI <a id="added-global-option-to-disable-c-rtti"></a>

Новая bool опция называется `cpp_rtti`.

### Изменения API интроспекции <a id="introspection-api-changes"></a>

dependencies \(--dependencies, intro-dependencies.json\):

* добавлена опция `version`key

сканирование зависимостей \(--scan-dependencies\):

* добавлена опция `version`key, содержащий требуемую версию зависимости

Тесты и бенчмарки \(--tests, --benchmarks, intro-tests.json, intro-benchmarks.json\):

* добавлена опция `protocol`key

