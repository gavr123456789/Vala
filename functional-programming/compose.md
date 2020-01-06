---
description: Functional templating for Vala
---

# Compose

[https://github.com/arteymix/compose](https://github.com/arteymix/compose)

Functional templating for Vala.

```csharp
using Compose;
using Compose.HTML5;

var app = new Valum.Router ();

app.get ("/users", () => {
  var users = User.all ();
  return res.expand_utf8 (
    html ({},
      head ({},
        title ("My really amazing page"),
        link ("/static/style.css")
      ),
      body ({"lang=en"},
        section ({"class=users"},
          h2 ({}, "Users"),
          take<User> (()     => users.next (),
                      (user) => p (user.username)))
      )
    )
  );
});
```

Using expressions for evaluating templates presents many advantages:

* it's fast
* it's quite elegant
* lazy evaluation using callbacks with native environments
* it's reusable since nodes can be isolated in functions
* it provides minimal compile-time correctness
* HTML5 elements support

Attributes are passed as an null-terminated array of `key=value` entries, just like environment variables. There is no need to quote or escape what follows the `=` sign.

```text
script ("/static/script.js", {"type=application/javascript", "defer"});
```

Also, Compose will automatically escape data where it has to be.

### HTML5

Every HTML5 elements can be found within the `Compose.HTML5` namespace like shown in the first code example.

To escape values, which is not done by default if HTML is expected, use the `e` helper.

```text
a (e (post.url), e (post.title));
```

Note that all attributes are already escaped.

Support for other format will be included if the project happen to become successful.

### Utilities

Compose work like a sink, so it will consume data from various sources through a simple callback API. Lazy evaluation at it's finest!

#### When

Test some condition and render either the first or second callback result.

```text
when (some_condition,
      () => { return p ({}, "Paragraphe"); },
      () => { return div ({}, "Erreur"); });
```

#### Take

Consume a callback until it return `null`, using another callback to perform evaluation.

To help keep track of the current index an array is used, a counter is passed as first argument.

```text
string[] @values = {"Jim", "John"};

take<string> ((i)    => { return i >= @values.length ? null : @values[i]; },
              (name) => { return h2 (name) });
```

