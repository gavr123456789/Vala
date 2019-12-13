# Классы

## Классы

```text
 className класса : SuperClassName , InterfaceName {
}
```

Определяет класс \(ссылочный тип\). В отличии от структур, экземпляры классов выделяются в куче. Более полно классы обсуждаются в секции "Объектно-ориентированное программирование".

## Интерфейсы

```text
interface  InterfaceName : SuperInterfaceName {
}
```

Определяет интерфейс - неинстанциируемый тип. Чтобы создать экземпляр интерфейса, необходимо сначала реализовать его абстрактные методы в неабстрактном классе. Интерфейсы в Vala имеют больше возможностей, чем интерфейсы в Java или C\#. Фактически они могут быть использованы как [**примеси**](https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%B8%D0%BC%D0%B5%D1%81%D1%8C_%28%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%29). Детали будут описаны в секции "Объектно-ориентированное программирование".

------------

wiki: Преимуществом примесей является то, что повышая [повторную используемость текстов программ](https://ru.wikipedia.org/wiki/%D0%9F%D0%BE%D0%B2%D1%82%D0%BE%D1%80%D0%BD%D0%BE%D0%B5_%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5_%D0%BA%D0%BE%D0%B4%D0%B0), этот метод избегает многих проблем [множественного наследования](https://ru.wikipedia.org/wiki/%D0%9C%D0%BD%D0%BE%D0%B6%D0%B5%D1%81%D1%82%D0%B2%D0%B5%D0%BD%D0%BD%D0%BE%D0%B5_%D0%BD%D0%B0%D1%81%D0%BB%D0%B5%D0%B4%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5). Однако при этом метод накладывает свои ограничения.

Вообщем примеси это так сказать ООП как оно задумывалось изначально, а не как его реализовали создатели С++.

примеси поддерживают следующие языки:

* [Swift](https://ru.wikipedia.org/wiki/Swift_%28%D1%8F%D0%B7%D1%8B%D0%BA_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F%29) \(в виде extension\)
* [Kotlin](https://ru.wikipedia.org/wiki/Kotlin) \(в виде extension\)
* [Ruby](https://ru.wikipedia.org/wiki/Ruby);
* [D](https://ru.wikipedia.org/wiki/D_%28%D1%8F%D0%B7%D1%8B%D0%BA_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F%29);
* [Dart](https://ru.wikipedia.org/wiki/Dart);
* [Python](https://ru.wikipedia.org/wiki/Python);
* [Scala](https://ru.wikipedia.org/wiki/Scala_%28%D1%8F%D0%B7%D1%8B%D0%BA_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F%29) \(в виде [trait](https://ru.wikipedia.org/wiki/Trait)'ов\);
* [XOTcl](https://ru.wikipedia.org/wiki/XOTcl);
* [PHP](https://ru.wikipedia.org/wiki/PHP);
* [Haxe](https://ru.wikipedia.org/wiki/Haxe);
* [Vala](https://ru.wikipedia.org/wiki/Vala).

[Статья](https://blogs.gnome.org/jnelson/2011/11/01/a-few-of-my-favorite-vala-things-interface/) в которой подробно рассматриваются преимущества интерфейсов Vala.

## Атрибуты

Атрибуты указывают компилятору Vala детали о том, как именно должен работать код на целевой платформе. Их синтаксис прост: `[AttributeName]` или `[AttributeName(param1 = value1, param2 = value2, ...)]`.

Они в основном используются для привязок в vapi файлах, `[CCode(...)]` наиболее используемый атрибут в них. Другой пример - атрибут `[DBus(...)]`, используемый для экспортирования удалённых интерфейсов через D-Bus.

## Примечание

![](../.gitbook/assets/image%20%288%29.png)

Существует возможность создать компактный класс - он не наследуется от Object и следовательно имеет ограниченные возможности, инициализация членов таких классов будет выполнятся быстрее.

"Только классы GObject имеют свойства gobject \([интроспектируемые](https://ru.wikipedia.org/wiki/%D0%98%D0%BD%D1%82%D1%80%D0%BE%D1%81%D0%BF%D0%B5%D0%BA%D1%86%D0%B8%D1%8F_%28%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%29) во время выполнения\). Эта особенность не выделена на изображении выше."

  


