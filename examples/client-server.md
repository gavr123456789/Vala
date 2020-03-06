# Client Server

{% hint style="info" %}
Оригинал статьи\(wiki репы\) и репозиторий с кодом находятся [здесь](https://gitlab.gnome.org/MrSyabro/soup_websocket_Example).
{% endhint %}

Для начала слегка теории о том, зачем все это, что такое Soup, Websocket и немного остального.

[**libsoup**](https://gitlab.gnome.org/GNOME/libsoup) - это HTTP клиент/сервер библиотека для GNOME. Использует объектную систему GObject и GLib Main Loop\(спизжено с источника по ссылке\)

[**WebSocket**](https://ru.wikipedia.org/wiki/WebSocket) — протокол ~~заднеприводной~~ полнодуплексной связи поверх TCP-соединения, предназначенный для обмена сообщениями между браузером и веб-сервером в режиме реального времени.\(источник опять по ссылке\)

А теперь о том, что же я хотел и как меня это все напрягло. Возникла задача написать бота для Discord. Знания в программирование, на данный момент, у меня только в 2-х языках: Vala и Lua. Не долго думая побежал рыться в интернете на тему API Discord'a на языке Vala и наткнулся на заброшеный проект [Valacord](https://github.com/G3bE/valacord). В нем была реализована часть обращения к серверам посредством простых `GET`/`POST` запросов, но конская работа по наладке и поддержке Websocket соединения предстояла в будущем. Я полез рыться по Soup докам и..

> кошка лягла на мышь.

В следующий раз напишу 2 страницы на тему клиента и сервера, и подводных камней, которые не очевидны и я с ними столкнулся.

### Client

**С клиентом не так все просто...**

С клиентом, казалось бы, все просто: создали, подключились и "общаемся". Но не тут то было..

В первую очередь выделим место под экземпляр соединения за пределами основной функции чтобы иметь к ней доступ из остальной программы:

```text
static Soup.WebsocketConnection websocket;
```

Для соединения нам понадобится `Soup.Session`, который соединит нас с сервером и `Soup.Message` с данными о сервере. Собственно:

```text
var session = new Soup.Session();
var message = new Soup.Message("GET", "http://<server_host>[:server_port]/[path]");
```

*  `<server_host>` - _собственно адрес сервера_
*  `[:server_port]` - _порт сервера, если требуется_
*  `[path]` - _путь к соединению на сервере_

> Учтем, что Websocket работает над HTTP и GET. Указываем это при создании `Soup.Message`.

Затем создаем новый основной поток, в котором и будет ждать сообщений наш `Soup.WebsocketConnection`:

```text
var loop = new MainLoop();
```

> Вопросы? Поидее должны отсутствовать..![:thinking:](/assets/emoji/thinking-4f0b84e5ab8a650cafb166e93688f0e9b31b9ade22a91035261ac90490edb9d3.png)️

Получаем экземпляр `Soup.WebsocketConnection`:

```csharp
session.websocket_connect_async.begin(message, null, null, null, (obj, res) => {
    websocket = session.websocket_connect_async.end(res);
    websocket.message.connect(ws_message);
    websocket.closed.connect(ws_closed);
    websocket.error.connect(ws_error);
});
```

Тут то вот что происходит по строкам:

1. Функция `websocket_connect_async()` имеет такой синтаксис:  
   `public async WebsocketConnection websocket_connect_async (Message msg, string? origin, string[]? protocols, Cancellable? cancellable) throws Error`

   соответственно мы указваем наш `Soup.Message message`, а остальные аргументы опускаем т.к. они нам не важны + асинхронная обвязка. _\(за подробностями по аргументам обратитесь к_ [_valadoc_](https://valadoc.org/libsoup-2.4/Soup.Session.websocket_connect_async.html)_\)_

2. Тут, по завершению работы функции из первой строки, мы получаем заветное `Soup.WebsocketConnection websocket`
3. Соединяем сигнал получения сообщения от сервера с функцией, которую рассмотрим позже
4. Также сигнал закрытия соединения
5. И сигнал ошибки соединения, соответственно

 **ws\_message\(int type, Bytes message\)**

*  `type` - тип полученного сообщения \(`BINARY|TEXT`\)[`public enum WebsocketDataType`](https://valadoc.org/libsoup-2.4/Soup.WebsocketDataType.html)
*  `message` - [`GLib.Bytes`](https://valadoc.org/glib-2.0/GLib.Bytes.html) экземпляр, который хранит полученное сообщение

Код функции:

```csharp
void ws_message(int type, Bytes message){
    stdout.printf((string)message.get_data());
}
```

> На второй строке получаем наше сообщение в виде `uint8[]` из `message.get_data()` и представляем в виде строки.

 **ws\_closet\(\)**

Тут все просто. Мы просто обрабатываем ситуацию, когда соединение закроется.

```csharp
void ws_closed(){
    stdout.printf("WebSock closed!"); // я просто вывел сообщение
}
```

 **ws\_error\(Error error\)**

Функция получает экземпляр класса [`GLib.Error`](https://valadoc.org/glib-2.0/GLib.Error.html)

```csharp
void ws_error(Error error){
    int code = error.code;
    string message = error.message;
    stdout.printf("WS Error %d: %s\n".printf(code,message));
}
```

Тут вроде бы все тоже ясно и понятно.  
На этом эта статься заканчивается.️ Не судите строго, я в этом не силен.

