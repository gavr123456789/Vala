# Введение

На системном уровне библиотека Vala это то же самое, что и библиотека на С, поэтому можно использовать те же инструменты. Для упрощения процесса обработки исходников компилятором Vala в них содержится вспомогательная информация.

Таким образом, библиотека Vala имеет системную часть:

Системная библиотека \(напр. libgee.so\)

Элемент pkg-config \(напр. gee-1.0.pc

Они оба устанавливаются в стандартные пути. А также специфичные для Vala файлы:

VAPI файл \(напр. gee-1.0.vapi\)

Опциональный файл зависимостей \(e.g. gee-1.0.deps\)

Эти файлы описаны более подробно в этом разделе. Нужно отметить, что названия файлов специфичных для Vala и pkg-config файлов одинаковы.

