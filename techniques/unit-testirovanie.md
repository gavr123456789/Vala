# Unit тестирование

Мне понравилась идея гайдов в 1 скрин, помойму это одновременно показывает насколько это просто и является хорошей шпаргалкой. Так что вот еще один. 

## Meson

![&#x412; &#x43A;&#x43E;&#x43D;&#x446;&#x435; &#x441;&#x442;&#x430;&#x442;&#x44C;&#x438; &#x435;&#x441;&#x442;&#x44C; &#x43A;&#x43E;&#x434; &#x434;&#x43B;&#x44F; &#x43A;&#x43E;&#x43F;&#x438;&#x440;&#x43E;&#x432;&#x430;&#x43D;&#x438;&#x44F;](../.gitbook/assets/image%20%289%29.png)

Как делать Unit тесты в Meson: 

1.  добавить в meson build `test(название, executable(execname, files, deps))` 

Всё, meson неплохо интегрирован в VS Code, так что теперь в его подразделе просто появятся пункты запуска тестов, клик по ним скомпилирует и запустит соответствующий тест.

### Фичи

* **Параметры** для тестов `test('command line test', exe, args : ['first', 'second'])`
* **MALLOC\_PERTURB\_**  По умолчанию для переменной среды [`MALLOC_PERTURB_`](http://man7.org/linux/man-pages/man3/mallopt.3.html) задано случайное значение между 1..255. Это может помочь обнаружить утечки памяти в конфигурациях с использованием glibc, в том числе с компиляторами, не относящимися к GCC.
* **Coverage**  
  Флаг **** `-Db_coverage=true`, вкючает создание отчетов о покрытии кода. Meson автоматически определит, какие инструменты генератора покрытия вы установили, и сгенерирует соответствующие цели. Эти цели являются `coverage-xml`и `coverage-text` , обе предоставлены [Gcovr](http://gcovr.com/) \(версия 3.3 или выше\). Для `coverage-html` требуются [Lcov](https://ltp.sourceforge.io/coverage/lcov.php) и [GenHTML](https://linux.die.net/man/1/genhtml) или [Gcovr](http://gcovr.com/) . Для удобства мониторинга покрытия также создается цель высокого уровня, которая, по возможности, создаст все 3 типа отчетов о покрытии.

  Вывод этих команд записывается в каталог журнала `meson-logs`в вашей директории сборки.

* **Parallelism** Чтобы сократить время тестирования, Meson по умолчанию будет запускать несколько модульных тестов параллельно. Если тест не следует запускать паралельно, укажите флаг `test('unique test', t, is_parallel : false)` По умолчанию Meson использует столько параллельных процессов, сколько ядер имеется на тестовой машине. Для переопределения и этого параметра используйте переменную окружения `MESON_TESTTHREADS=5 ninja test`
* **Priorities**  
  Тестам может быть назначен приоритет, который определяет, когда тест начнется.

  ```text
  test('started second', t, priority : 0)
  test('started third', t, priority : -50)
  test('started first', t, priority : 1000)
  ```

* **Subsets**  
  Для ясности рассмотрим meson.build, содержащий:

  ```text
  test('A', ..., suite: 'foo')
  test('B', ..., suite: 'foo')
  test('C', ..., suite: 'bar')
  test('D', ..., suite: 'baz')
  ```

  Укажите тест \(ы\) по имени, например: `$ meson test A D` или по имени suite `$ meson test --suite (sub)project_name:suite`

* **repeat** `$ meson test --repeat=10`
* **wrap** `$ meson test --wrap=valgrind testname`
* **args** for wrap `$ meson test --wrap='valgrind --tool=helgrind' testname`
* **GDB** `$ meson test --gdb testname`
* Пример команды для теста который даёт сбой в редких случаях `$ meson test --gdb --repeat=10000 testname` Это автоматически запускает тест до 10 000 раз под GDB. В случае сбоя программы GDB остановится, и пользователь сможет отладить приложение.
* **путь к GDB** `$ meson test --gdb --gdb-path /path/to/gdb testname`
* **errorlogs** Meson сообщит результаты, полученные в результате неудачных испытаний, вместе с другой полезной информацией в качестве переменных среды. Это полезно, например, когда вы запускаете тесты на Travis-CI, Jenkins и т.п.

## GLib.Test

В GLib уже встроен юнит тест фреймворк \(а значит и в Vala\), все что нужно это инициализировать его передав аргументы `Test.init (ref args)`, добавить функций `Test.add_func(test_path,func)` и запустить `Test.run()` 

{% hint style="info" %}
test\_path это абстрактный путь к объекту тестирования, например "xml/read" "xml/parse", вложенность может быть любой глубины. Как вы можете заметить для каждой вложенности выводиться сообщение `# Start of xml tests` `# End of xml tests`
{% endhint %}

#### Фичи:

* `timer_start` и `timer_elapsed` для замера времени и таймаутов 
* `Test.minimized_result` и `Test.maximized_result` для отображения результатов бенчей по времени с сортировкой;
* `Test.skip(string msg)` для пропуска теста с сообщением;
* `Test.summary` - Установите сводку для теста, которая описывает, что проверяет тест, и как он проходит проверку;
* `Test.log_set_fatal_handler(log_func)` для кастомного обработчика ошибок который будет решать насколько они фатальны для завершения программы, тут сложно, читать фул [здесь](https://valadoc.org/glib-2.0/GLib.Test.log_set_fatal_handler.html);
* `Test.bug_base(URL)` и `Test.bug(string)` вот это вообще круто, в отчет будут сразу вставленны ссылки на issue например гитхаба\(на скриншоте это есть\);
* `Test.fail()` отметить тест зафейленным, но продолжать его выполнение; -- `Test.failed()` возвращает `true` если тест уже зафейлин;
* `Test.trap_reached_timeout` добавляет в тест timeout;
* `Test.trap_assert_passed` Если вы хотите выстроить зависимости между тестами, чтобы например если тест А не прошел то Б даже не стартовал и наоборот. Данная функция это assert что предыдущий тест был завершен успешно;
* `Test.incomplete(string? Msg = null )` Указывает что тест не пройден из-за неполной функциональности, функция может быть вызвана несколько раз за один и тот же тест;
* `Test.trap_subprocess(test_path,timeout,flags)` возвращает текущую программу\(ага\) как подпроцесс для запуска из другой при вызове с аргументом `null` то есть, Программа А делает`trap_subprocess(путь до exec программы Б)` а также `trap_assert_failed` и `trap_assert_stderr` для перехвата результата и вывода. А в программе Б стоит `trap_subprocess (NULL, 0, 0);` который перезапускает Б как подпроцесс А. [Пример](https://valadoc.org/glib-2.0/GLib.Test.trap_subprocess.html)

Это далеко не все, но я думаю хватит, осталось подключение файлов к конкретным тестам по путям, [TestCase](https://valadoc.org/glib-2.0/GLib.Test.add_data_func.html) - класс-тест, функции для генерации рандомных значений и тд. 



### Для копирования

{% tabs %}
{% tab title="meson.build" %}
```css
#main meson.build
project('Compose', 'c', 'vala',
        version: '1.0.0-dev')

glib_dep = dependency('glib-2.0')
libxml_dep = dependency('libxml-2.0')

subdir('src')
subdir('tests')

```
{% endtab %}

{% tab title="tests/meson.build" %}
```css
test('compose', executable('compose-test', 'compose-test.vala', dependencies: [glib_dep, compose_dep]))
test('html5', executable('html5-test', 'html5-test.vala', dependencies: [glib_dep, compose_dep]))
```
{% endtab %}

{% tab title="html5-test.vala" %}
```csharp
using Compose.HTML5;
public int main (string[] args){
	Test.init (ref args);
	Test.bug_base ("https://gitlab.com/kosmospredanie/gpseq/issues/");
	Test.add_func ("/html5/element", () => {
		assert ("<tr&gt; a=\"\"\"><input/></tr&gt;>" == element ("tr>", {"a=\""}, "<input/>"));
	});
	Test.add_func ("/html5/5*5", () => {
		assert_true (5*5 == 25);
	});
	Test.add_func ("/html3.5/25=5", () => {
		if (25 != 5){
			Test.fail();
			Test.bug("7");
		}
	});
	return Test.run ();
}

```
{% endtab %}

{% tab title="Log.txt" %}
```
Log of Meson test suite run on 2019-12-13T04:12:37.173382

Inherited environment: SHELL='/usr/bin/zsh' COLORTERM='truecolor' SDK_HOME='/usr/lib64/jvm/java' XDG_CONFIG_DIRS='/etc/xdg/pantheon:/etc/xdg' LESS='-M -I -R' XDG_MENU_PREFIX='pantheon-' JDK_HOME='/usr/lib64/jvm/java' GTK_IM_MODULE='ibus' MACHTYPE='x86_64' G_BROKEN_FILENAMES='1' HOSTNAME='localhost.localdomain' HISTSIZE='50000' MINICOM='-c on' QT4_IM_MODULE='xim' JAVA_ROOT='/usr/lib64/jvm/java' JAVA_HOME='/usr/lib64/jvm/java' AUDIODRIVER='pulseaudio' JRE_HOME='/usr/lib64/jvm/java' SSH_AUTH_SOCK='/run/user/1001/keyring/ssh' INPUT_METHOD='ibus' CPU='x86_64' JAVA_BINDIR='/usr/lib64/jvm/java/bin' XMODIFIERS='@im=ibus' DESKTOP_SESSION='pantheon' SSH_AGENT_PID='2090' GTK_MODULES='canberra-gtk-module' XDG_SEAT='seat0' PWD='/home/gavr/dev/Builds/compose-master/build' QEMU_AUDIO_DRV='pa' LOGNAME='gavr' XDG_SESSION_DESKTOP='pantheon' QT_QPA_PLATFORMTHEME='gtk3' XDG_SESSION_TYPE='x11' MANPATH='/usr/local/man:/usr/share/man' XAUTHORITY='/run/user/1001/gdm/Xauthority' XKEYSYMDB='/usr/X11R6/lib/X11/XKeysymDB' QT_STYLE_OVERRIDE='adwaita' WINDOWPATH='2' XNLSPATH='/usr/share/X11/nls' HOME='/home/gavr' USERNAME='gavr' SSH_ASKPASS='/usr/lib/ssh/ssh-askpass' LANG='ru_RU.UTF-8' XDG_CURRENT_DESKTOP='Unity' PYTHONSTARTUP='/etc/pythonstart' OSTYPE='linux-gnu' QT_IM_SWITCHER='imsw-multi' LESS_ADVANCED_PREPROCESSOR='no' GTK_CSD='1' XSESSION_IS_UP='yes' LESSCLOSE='lessclose.sh %s %s' XDG_SESSION_CLASS='user' TERM='xterm-256color' G_FILENAME_ENCODING='@locale,UTF-8,KOI8-R,CP1251' HOST='localhost.localdomain' XAUTHLOCALHOSTNAME='localhost' LESSOPEN='lessopen.sh %s' USER='gavr' SDL_AUDIODRIVER='pulse' MORE='-sl' CSHEDIT='emacs' DISPLAY=':0' SHLVL='1' WINDOWMANAGER='/usr/bin/pantheon' PAGER='less' QT_IM_MODULE='ibus' CVS_RSH='ssh' XDG_VTNR='2' XDG_SESSION_ID='1' XDG_RUNTIME_DIR='/run/user/1001' XCURSOR_THEME='DMZ' GTK3_MODULES='pantheon-filechooser-module' XDG_DATA_DIRS='/home/gavr/.local/share/flatpak/exports/share:/var/lib/flatpak/exports/share:/usr/local/share:/usr/share:/var/lib/snapd/desktop' CONFIG_SITE='/usr/share/site/x86_64-unknown-linux-gnu' PATH='/home/gavr/.nimble/bin:/home/gavr/bin:/usr/local/bin:/usr/bin:/bin:/snap/bin:/home/gavr/bin/SDK//flutter/bin:/home/gavr/bin/SDK/Dart/bin:/home/gavr/bin/dotnet/:/home/gavr/.dotnet/tools:/usr/local/lib64:/usr/local/lib64/girepository-1.0' GDMSESSION='pantheon' DBUS_SESSION_BUS_ADDRESS='unix:path=/run/user/1001/bus' PROFILEREAD='true' MAIL='/var/spool/mail/gavr' HOSTTYPE='x86_64' LESSKEY='/etc/lesskey.bin' OLDPWD='/home/gavr/dev/Builds/compose-master/build' SESSION_MANAGER='local/localhost.localdomain:@/tmp/.ICE-unix/2019,unix/localhost.localdomain:/tmp/.ICE-unix/2019' GIO_LAUNCHED_DESKTOP_FILE='/usr/share/applications/code.desktop' GIO_LAUNCHED_DESKTOP_FILE_PID='20774' GSETTINGS_SCHEMA_DIR='/home/gavr/data' NO_AT_BRIDGE='1' CHROME_DESKTOP='code-url-handler.desktop' ANDROID_HOME='/home/gavr/bin/SDK/Android/Sdk/' DOTNET_ROOT='/home/gavr/bin/dotnet/' GI_TYPELIB_PATH='/usr/local/lib64/girepository-1.0' LD_LIBRARY_PATH='/usr/local/lib64' LSCOLORS='Gxfxcxdxbxegedabagacad' LS_COLORS='no=00:fi=00:di=01;34:ln=00;36:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=41;33;01:ex=00;32:*.cmd=00;32:*.exe=01;32:*.com=01;32:*.bat=01;32:*.btm=01;32:*.dll=01;32:*.tar=00;31:*.tbz=00;31:*.tgz=00;31:*.rpm=00;31:*.deb=00;31:*.arj=00;31:*.taz=00;31:*.lzh=00;31:*.lzma=00;31:*.zip=00;31:*.zoo=00;31:*.z=00;31:*.Z=00;31:*.gz=00;31:*.bz2=00;31:*.tb2=00;31:*.tz2=00;31:*.tbz2=00;31:*.xz=00;31:*.avi=01;35:*.bmp=01;35:*.dl=01;35:*.fli=01;35:*.gif=01;35:*.gl=01;35:*.jpg=01;35:*.jpeg=01;35:*.mkv=01;35:*.mng=01;35:*.mov=01;35:*.mp4=01;35:*.mpg=01;35:*.pcx=01;35:*.pbm=01;35:*.pgm=01;35:*.png=01;35:*.ppm=01;35:*.svg=01;35:*.tga=01;35:*.tif=01;35:*.webm=01;35:*.webp=01;35:*.wmv=01;35:*.xbm=01;35:*.xcf=01;35:*.xpm=01;35:*.aiff=00;32:*.ape=00;32:*.au=00;32:*.flac=00;32:*.m4a=00;32:*.mid=00;32:*.mp3=00;32:*.mpc=00;32:*.ogg=00;32:*.voc=00;32:*.wav=00;32:*.wma=00;32:*.wv=00;32:' LS_OPTIONS='-N --color=tty -T 0' P9K_SSH='0' ZSH='/home/gavr/.oh-my-zsh' _='/usr/bin/meson' TERM_PROGRAM='vscode' TERM_PROGRAM_VERSION='1.40.2' 

1/1 html5                                   FAIL     0.01 s (killed by signal 6 SIGABRT)

--- command ---
/home/gavr/dev/Builds/compose-master/build/tests/html5-test
--- stdout ---
# random seed: R02S4003e282caf0dc56924c02fc042d365e
1..3
# Start of html5 tests
ok 1 /html5/element
ok 2 /html5/5*5
# End of html5 tests
# Start of html3.5 tests
# Bug Reference: https://gitlab.com/kosmospredanie/gpseq/issues/7
not ok 3 /html3.5/25=5
Bail out!
-------
// Подсчет идет не для тесткейсов в одном тесте, а количества запущенных тестов

Ok:                    0
Expected Fail:         0
Fail:                  1
Unexpected Pass:       0
Skipped:               0
Timeout:               0

```
{% endtab %}
{% endtabs %}

## ValaDate

Также, если одних Unit тестов вам мало, существует фреймворк для тестирования [ValaDate](https://github.com/astavale/valadate) рассказывать тут я про него не буду, у автора и так хорошая вики. Только приведу список фич и пример. И вот еще статейка [Writing tests for Vala](https://esite.ch/2012/06/writing-tests-for-vala/) из 2012.

Arrange &gt;&gt; Act &gt;&gt; Assert

#### Описание

Для разработчиков Vala, которым необходимо тестировать свой код, Valadate представляет собой мощную среду тестирования, которая предоставляет функции поведенческого, функционального и модульного тестирования, помогающие им писать отличное ПО с открытым исходным кодом. В отличие от других сред тестирования, Valadate разработан специально для Vala, в то же время плавно интегрируясь в существующие наборы инструментов.

### Фичи:

* Автоматическое обнаружение тестов, также как в JUnit или .NET Framework.
* Функции утилиты для ожидания в `mainloop`, пока не произойдет указанное событие или тайм-аут.
* Поддержка асинхронных тестов.
* Служебные функции, предоставляющие временный каталог для тестов.
* Пропущенные тесты и ожидаемые сбои.

### Пример

```csharp
[Test (name="Annotated TestCase with name")]
public class MyTest : Valadate.Framework.TestCase {

	[Test (name="Annotated Method With Name")]
	public void annotated_test_with_name () {
		assert_true(true);
	}


	[Test (name="Asynchronous Test", timeout=1000)]
	public async void test_async () {
		assert_true(true);
	}

	[Test (skip="yes")]
	public void skip_test () {
		assert_true(false);
	}

}



1..3
# Start of Annotated TestCase with name tests
ok 1 /Annotated TestCase with name/Annotated Method With Name
ok 2 /Annotated TestCase with name/Asynchronous Test
ok 3 /Annotated TestCase with name/skip_test # SKIP Skipping Test skip_test
# End of Annotated TestCase with name tests
```

####   

