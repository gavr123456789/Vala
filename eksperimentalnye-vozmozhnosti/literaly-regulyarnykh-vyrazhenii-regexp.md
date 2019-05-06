# Литералы регулярных выражений\(regexp\)

Регулярные выражения - это мощное средство поиска по шаблону в строках. Vala имеет экспериментальную поддержку литералов для регулярных выражений. Например:

```csharp
string email = "tux@kernel.org";
if (/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}$/i.match(email)) {
    stdout.printf("Valid email address\n");
}
```

i в конце делает выражение не чувствительным к регистру. Вы можете хранить регулярное выражение в переменной типа Regex:

```csharp
Regex regex = /foo/;
```

A example of regular expression replacement:

```csharp
var r = /(foo|bar|cow)/;
var o = r.replace ("this foo is great", -1, 0, "thing");
print ("%s\n", o);
```

The following trailing characters can be used:

* _i_, letters in the pattern match both upper- and lowercase letters
* _m_, the "start of line" and "end of line" constructs match immediately following or immediately before any newline in the string, respectively, as well as at the very start and end.
* _s_, a dot metacharater _._ in the pattern matches all characters, including newlines. Without it, newlines are excluded.
* _x_, whitespace data characters in the pattern are totally ignored except when escaped or inside a character class.

