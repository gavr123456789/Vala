# Meson 0.54

## Новые фичи

### Emscripten \(emcc\) now supports threads <a id="emscripten-emcc-now-supports-threads"></a>

В дополнение к правильной настройке аргументов компиляции и компоновщика был добавлен новый встроенный в meson для управления параметром PTHREAD\_POOL\_SIZE, `-D_thread_count`, который может быть установлен в любое целое значение, большее 0. Если он установлен в 0, то параметр PTHREAD\_POOL\_SIZE не будет передан.

### Introduce dataonly for the pkgconfig module <a id="introduce-dataonly-for-the-pkgconfig-module"></a>

Это позволяет пользователям отключить запись встроенных переменных в файл pkg-config, поскольку они могут фактически не потребоваться.

Одна из причин этого - наличие независимых от архитектуры файлов pkg-config в проектах, которые 

имеют зависимые от архитектуры выходные данные.

```text
pkgg.generate(
  name : 'libhello_nolib',
  description : 'A minimalistic pkgconfig file.',
  version : libver,
  dataonly: true
)
```

### Consistently report file locations relative to cwd <a id="consistently-report-file-locations-relative-to-cwd"></a>

Пути для имен файлов в которых расположены ошибоки и предупреждения теперь сообщаются относительно текущего рабочего каталога \(когда это возможно\) или как абсолютные пути \(когда относительный путь не существует, например путь Windows, начинающийся с другой буквы диска к текущему рабочему каталогу\).

\(Предыдущее поведение состояло в том, чтобы сообщить путь относительно исходного корня для всех предупреждений и большинства ошибок, а также относительно cwd для некоторых ошибок синтаксического анализатора\)

### `dependency()` consistency <a id="dependency-consistency"></a>

Когда dependency найдена первый раз используя `dependency('foo', ...)`, возвращаемое значение кешируется. Любой последующий вызов будет возвращать то же самое значение до тех пор, пока запрошенная версия совпадает, в противном случае возвращается not-found dependency.  Это означает, что если системная зависимость будет найдена первой, то она больше не будет возвращаться к subproject в последующем вызове и скорее вернет not-found, если системная версия не совпадает. 

Аналогично, если первый вызов возвращает резервную зависимость подпроекта, он также вернет зависимость подпроекта в последующем вызове, даже если резервная зависимость не предусмотрена.

For example, if the system has `foo` version 1.0:  


```python
# d2 is set to foo_dep and not the system dependency, even without fallback argument.
d1 = dependency('foo', version : '>=2.0', required : false,
                fallback : ['foo', 'foo_dep'])
d2 = dependency('foo', version : '>=1.0', required : false)
```

```python
# d2 is not-found because the first call returned the system dependency, but its version is too old for 2nd call.
d1 = dependency('foo', version : '>=1.0', required : false)
d2 = dependency('foo', version : '>=2.0', required : false,
                fallback : ['foo', 'foo_dep'])
```

### Override `dependency()`  <a id="override-dependency"></a>

Теперь можно переопределить результат dependence \(\), чтобы указать на любой объект зависимости, который вы хотите. Переопределение является глобальным и применяется к каждому подпроекту c момента переопределения.

Например, этот подпроект предоставляет 2 библиотеки с версией 2.0:

```python
project(..., version : '2.0')

libfoo = library('foo', ...)
foo_dep = declare_dependency(link_with : libfoo)
meson.override_dependency('foo', foo_dep)

libbar = library('bar', ...)
bar_dep = declare_dependency(link_with : libbar)
meson.override_dependency('bar', bar_dep)
```

Предположим, что в системе установлены foo и bar 1.0, и мастер-проект делает это:

```python
foo_dep = dependency('foo', version : '>=2.0', fallback : ['foo', 'foo_dep'])
bar_dep = dependency('bar')
```

Это использовалось для смешивания зависимостей system 1.0 version и subproject 2.0, но благодаря переопределению bar\_dep теперь устанавливлена на версию подпроекта.

Еще один кейс, который может быть полезен, - это заставить подпроект использовать определенную зависимость. Если подпроект делает зависимость\('foo'\), но основной проект хочет обеспечить свою собственную реализацию foo, он может, например, вызвать  `meson.override_dependency('foo', declare_dependency(...))` перед конфигурацией подпроекта.

### Simplified `dependency()` fallback <a id="simplified-dependency-fallback"></a>

В этом кейсе подпроект  `foo` вызывает `meson.override_dependency('foo-2.0', foo_dep)`, родительский проект может опустить имя переменной зависимости в fallback keyword: `dependency('foo-2.0', fallback : 'foo')`. \(Тк кк оно уже указано в `override_dependency`\)



### Backend agnostic compile command <a id="backend-agnostic-compile-command"></a>

Новая `meson compile`команда была добавлена ​​для поддержки независимости от бэкенда компиляции. Она принимает два аргумента `-j`и `-l`, которые используются по возможности \( `-l`ничего не делает с msbuild\). Значение `-j`или `-l`&lt; 1 позволяет бэкэнду решить, сколько потоков использовать. Для msbuild это означает `-m`, что для ninja является отсутствием аргументов.

```python
meson builddir --backend vs
meson compile -C builddir -j0  # this is the same as `msbuild builddir/my.sln -m`
```

```python
meson builddir
meson compile -C builddir -j3  # this is the same as `ninja -C builddir -j3`
```

Еще `meson compile` предоставляет `--clean` switch для очистки проекта.  
Полный список аргументов всегда документируется с помощью `meson compile --help`

### Native \(build machine\) compilers not always required <a id="native-build-machine-compilers-not-always-required"></a>

`add_languages()` gained a `native:` keyword, indicating if a native or cross compiler is to be used.

For the benefit of existing simple build definitions which don't contain any `native: true` targets, without breaking backwards compatibility for build definitions which assume that the native compiler is available after `add_languages()`, if the `native:` keyword is absent the languages may be used for either the build or host machine, but are never required for the build machine.

This changes the behaviour of the following meson fragment \(when cross-compiling but a native compiler is not available\) from reporting an error at `add_language` to reporting an error at `executable`.

```python
add_language('c')
executable('main', 'main.c', native: true)
```

### Summary improvements <a id="summary-improvements"></a>

Новый `list_sep`ключевое слово аргумент было добавлено в `summary()`. Если значение определено и является списком, элементы будут разделены предоставленной строкой, а не каждый на новой строке.

 The automatic `subprojects` section now also print the number of warnings encountered during that subproject configuration, or the error message if the configuration failed.

### Add a system type dependency for zlib <a id="add-a-system-type-dependency-for-zlib"></a>

Это позволяет обнаруживать zlib в macOS и FreeBSD без использования pkg-config или cmake, которые не являются частью базовой установки в этих ОС \(но при этом zlib присутствует\).

Побочным эффектом этого изменения является то, что  `dependency('zlib')`также работает с cmake вместо того, чтобы требовать `dependency('ZLIB')`.

### Added 'name' method <a id="added-name-method"></a>

Build target объекты \(возвращаемые executable\(\), library\(\), ...\) теперь есть имеют метод name \(\).

### New option `--quiet` to `meson install` <a id="new-option-quiet-to-meson-install"></a>

Теперь вы можете запустить, `meson install --quiet`и Meson не будет подробно печатать каждый файл, когда он устанавливается. Как и прежде, полный журнал всегда доступен внутри builddir в `meson-logs/install-log.txt`.

Когда эта опция пропущена, в установочных скриптах будет установлена ​​переменная окружения `MESON_INSTALL_QUIET`.

Многочисленные ускорения были также сделаны для этапа установки, особенно в Windows, где он теперь на 300% - 1200% быстрее, чем раньше, в зависимости от вашей рабочей нагрузки.

### Property support emscripten's wasm-ld <a id="property-support-emscriptens-wasmld"></a>

До 0.54.0 мы рассматривали emscripten как компилятор и компоновщик, что не совсем так. У него есть компоновщик, называемый wasm-ld \(имя мезона - ld.wasm\). Это специальная версия clang's lld. Теперь это будет обнаружено правильно.

### Skip sanity tests when cross compiling <a id="skip-sanity-tests-when-cross-compiling"></a>

Для определенных сред кросс-компиляции невозможно скомпилировать приложение проверки работоспособности. Теперь это можно отключить, добавив следующую запись в `properties`раздел кросс-файла :

```text
skip_sanity_check = true
```

### Support for overiding the linker with ldc and gdc <a id="support-for-overiding-the-linker-with-ldc-and-gdc"></a>

LDC \(компилятор D llvm\) и GDC \(компилятор D Gnu\) теперь поддерживают переменную компоновщика D\_LD \(или d\_ld в кросс-файле\) и могут выбирать разные компоновщики.

GDC поддерживает все те же значения, что и GCC, LDC поддерживает ld.bfd, ld.gold, ld.lld, ld64, link и lld-link.

### Native file properties <a id="native-file-properties"></a>

Начиная с версии Meson 0.54.0, `--native-file nativefile.ini`может содержать:

* binaries
* paths
* properties

которые определены и используются так же, как в кросс-файлах. `properties`являются новыми для Meson 0.54.0 и читаются как:

```python
x = meson.get_external_property('foobar', 'foo')
```

где `foobar`- имя свойства, а необязательное `foo`- запасное строковое значение.

Для кросс-скомпилированных проектов `get_external_property()`читает кросс-файл, пока не определено `native: true`.

### Changed the signal used to terminate a test process \(group\) <a id="changed-the-signal-used-to-terminate-a-test-process-group"></a>

Тестовый процесс \(группа\) теперь завершается через SIGTERM вместо SIGKILL, что позволяет обрабатывать сигнал. Однако теперь пользовательский обработчик сигналов \(если таковой имеется\) несет ответственность за правильное завершение любого процесса, порожденного процессами тестирования верхнего уровня.

### Dynamic Linker environment variables actually match docs <a id="dynamic-linker-environment-variables-actually-match-docs"></a>

В документах всегда указывалось, что переменная окружения Dynamic Linker должна быть `${COMPILER_VAR}_LD`, но это касается только половины переменных. Другая половина отличается. В 0.54.0 переменные совпадают. Старые переменные все еще поддерживаются, но устарели и выдают предупреждение об устаревании.

### Added 'pkg\_config\_libdir' property <a id="added-pkg_config_libdir-property"></a>

Позволяет определить список папок, используемых pkg-config для кросс-сборки, и избежать использования системных каталогов.

### More new sample Meson templates for \(`Java`, `Cuda`, and more\) <a id="more-new-sample-meson-templates-for-java-cuda-and-more"></a>

Мезон теперь поставляется с предопределенными шаблонами проектов для `Java`, `Cuda`, `Objective-C++`и `C#`мы обеспечены соответствующих значений для соответствующих языков, как для Имеющихся библиотек и исполняемый файл.

### Ninja version requirement bumped to 1.7 <a id="ninja-version-requirement-bumped-to-17"></a>

Meson теперь использует функцию [неявных выходов](https://ninja-build.org/manual.html#ref_outputs) Ninja для некоторых типов целей, которые имеют несколько выходов, которые могут не отображаться в командной строке. Для этой функции требуется ниндзя 1.7+.

Обратите внимание, что последняя версия [Ninja, доступная в Ubuntu 16.04](https://packages.ubuntu.com/search?keywords=ninja-build&searchon=names&suite=xenial-backports&section=all) \(самая старая Ubuntu LTS на момент написания статьи\) - 1.7.1. Если ваш дистрибутив не поставляется с достаточно новым Ninja, вы можете загрузить последнюю версию со страницы GitHub Ninja: https://github.com/ninja-build/ninja/releases

### Added `-C` argument to `meson init` command <a id="added-c-argument-to-meson-init-command"></a>

Мезон инициализации предполагает, что он запускается внутри корневого каталога проекта. Если это не так, теперь вы можете использовать `-C`для указания фактического исходного каталога проекта.

### More than one argument to `message()` and `warning()` <a id="more-than-one-argument-to-message-and-warning"></a>

 Аргументы переданы `message()`и `warning()`будут напечатаны через пробел.

### Added `has_tools` method to qt module <a id="added-has_tools-method-to-qt-module"></a>

Он должен использоваться для компиляции необязательного кода Qt:

```python
qt5 = import('qt5')
if qt5.has_tools(required: get_option('qt_feature'))
  moc_files = qt5.preprocess(...)
  ...
endif
```

### The MSI installer is only available in 64 bit version <a id="the-msi-installer-is-only-available-in-64-bit-version"></a>

Microsoft прекратила поддержку Windows 7, поэтому официально поддерживаются только 64-битные ОС Windows. Таким образом, в будущем будет предоставлен только 64-битный установщик MSI. Люди, которым нужна 32-битная версия, могут создавать свои собственные с помощью `msi/createmsi.py`скрипта в исходном хранилище Meson.

### Uninstalled pkg-config files <a id="uninstalled-pkgconfig-files"></a>

**Примечание** : функциональность этого модуля регулируется [правилами Meson по смешиванию систем сборки](https://mesonbuild.com/Mixing-build-systems.html) .

`pkgconfig`Модуль теперь генерирует неустановленные компьютерные файлы. Для любого сгенерированного `foo.pc`файла в `foo-uninstalled.pc` помещается дополнительный файл `<builddir>/meson-uninstalled`. Их можно использовать для создания приложений на основе библиотек, созданных meson, без их установки, указывая `PKG_CONFIG_PATH` на этот каталог. Это экспериментальная функция, предоставляемая с максимальным усилием \("on a best-effort basis"\), она может работать не во всех случаях использования.

### CMake find\_package COMPONENTS support <a id="cmake-find_package-components-support"></a>

Теперь можно передавать компоненты в бэкэнд зависимостей CMake через новый `components`kwarg в `dependency`функции.

### Added Microchip XC16 C compiler support <a id="added-microchip-xc16-c-compiler-support"></a>

Убедитесь, что исполняемые файлы компилятора настроены правильно по вашему пути. Компилятор доступен на веб-сайте Microchip бесплатно.

### Added Texas Instruments C2000 C/C++ compiler support <a id="added-texas-instruments-c2000-cc-compiler-support"></a>

Убедитесь, что исполняемые файлы компилятора настроены правильно по вашему пути. Компилятор доступен на сайте Texas Instruments бесплатно.

### Unity file block size is configurable <a id="unity-file-block-size-is-configurable"></a>

Традиционно файлы Unity, которые автоматически генерирует Meson, содержат все исходные файлы, принадлежащие одной цели. Это наиболее эффективный параметр для полных сборок, но он замедляет добавочные сборки. В этом выпуске добавлена ​​новая опция, `unity_size`которая указывает, сколько исходных файлов следует поместить в каждый файл Unity.

Значение по умолчанию для размера блока равно 4. Это означает, что если у вас есть цель с восемью исходными файлами, Meson создаст два файла единиц, каждый из которых включает четыре исходных файла. Старое поведение можно воспроизвести, установив `unity_size`большое значение, например 10000.  


