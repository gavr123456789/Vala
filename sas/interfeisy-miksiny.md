---
description: Пожалуйста поищите еще языки в которых есть миксины
---

# Интерфейсы / Миксины

Классы в Vala могут реализовывать любое количество интерфейсов. Каждый интерфейс - это тип, очень похожий на класс, но у которого не может быть экземпляров. Реализуя определенный интерфейс, экземпляры данного класса становятся экземплярами и интерфейса, и их можно использовать везде, где требуется объект определенного интерфейса.

Процедура реализации интерфейса такая же, как и при наследовании от абстрактного класса - если нужно иметь возможность создавать экземпляры этого класса, то придется реализовать все объявленные в интерфейсе методы.

Вот пример простого определения интерфейса:

```csharp
public interface ITest : GLib.Object {
    public abstract int data_1 { get; set; }
    public abstract void method_1();
}
```

В этом коде описан интерфейс `ITest`, который наследует от `GLib.Object` и содержит два поля. `data_1` - свойство, такие вы уже видели, но он абстрактный. Vala не будет реализовывать это свойство, но у классов реализующий данный интерфейс будет требовать его присутствия, с get и set внутри, т.к. интерфейсы не могут иметь поля данных. Второе поле - это метод `method_1`. Данный метод должен реализовывать классами, реализующими данный интерфейс.

Простейший способ реализации этого интерфейса:

```csharp
public class Test1 : GLib.Object, ITest {
    public int data_1 { get; set; }
    public void method_1() {
    }
}
```

И может быть использовано следующим образом:

```csharp
var t = new Test1();
t.method_1();

ITest i = t;
i.method_1();
```

Интерфейсы Vala не могут наследовать от других интерфейсов, но могут требовать их присутствия, что при работе с ними означает одно и то же. Например можно сказать, что любой класс, реализующий интерфейс List должен реализовывать так же интерфейс Сollection. Синтаксис в подобном случае не отличается от синтаксиса, описывающего реализацию интерфейса классом:

```csharp
public interface List : Collection, Traversable {
}
```

Определение List принуждает при его реализации реализовывать и класс Collection, таким образом Vala вынуждает при реализации интерфейса расписывать все интерфейсы, от которых он зависит:

```csharp
public class ListClass : GLib.Object, Collection, List {
}
```

Vala интерфейсы могут требовать и классы. Если в требованиях указано название класса, то интерфейс может быть реализован только в тех классах, которые наследуют от этого указанного класса. Обычно это применяется, чтобы быть уверенным, что экземпляр интерфейса так же наследует от `GLib.Object`, таким образом интерфейс может использоваться в качестве типа для свойства.

Фактически интерфейсы не могут наследоваться от других интерфейсов, но это формально - на самом деле в Vala это работает, как и в других языках, но с дополнительной особенностью предпосылки классов.

Есть еще одно важное отличие Vala интерфейсов от таковых в Java/С\#: Vala интерфейсы могут иметь не абстрактные методы! Vala позволяет в интерфейсе реализовывать методы, поэтому интерфейсы Vala можно использовать как миксины. И это ограниченная форма множественного наследования.

**Mixins and Multiple Inheritance**

As described above, Vala while it is backed by C and GObject, can provide a limited multiple inheritance mechanism, by adding virtual methods to Interfaces. Is possible to add some ways to define default method implementations in interface implementor class and allow derived classes to override that methods.

If you define a virtual method in an interface and implement it in a class, you can't override interface's method without leaving derived classes unable to access to interface default one. Consider following code:

```csharp
public interface Callable : GLib.Object {
   public abstract bool answering { get; protected set; }
   public abstract void answer ();
   public abstract bool hang ();
   public static bool default_hang (Callable call)
   {
      stdout.printf ("At Callable.hang()\n");
      call.answering = false;
      return true;
   }
}

public abstract class Caller : GLib.Object, Callable
{
   public bool answering { get; protected set; }
   public void answer ()
   {
     stdout.printf ("At Caller.answer()\n");
     answering = true;
     hang ();
   }
   public virtual bool hang () { return Callable.default_hang (this); }
}

public class TechPhone : Caller {
        public string number { get; set; }
}

public class Phone : Caller {
   public override bool hang () {
        stdout.printf ("At Phone.hang()\n");
        return false;
   }
   
   public static void main ()
   {
      var f = (Callable) new Phone ();
      f.answer ();
      if (f.hang ())
         stdout.printf("Hand done.\n");
      else
         stdout.printf("Hand Error!\n");
      
      var t = (Callable) new TechPhone ();
      t.answer ();
      if (t.hang ())
         stdout.printf("Tech Hand done.\n");
      else
         stdout.printf("Tech Hand Error!\n");
      stdout.printf("END\n");
   }
}
```

In this case, we have defined a Callable interface with a default implementation for abstract bool hang \(\) called default\_hang, it could be a static or virtual method. Then Caller is a base class implementing Callable for the TechPhone and Phone classes, while Caller'shang \(\) method simple call Callable default implementation. TechPhone doesn't do anything and just takes Caller as base class, using the default method implementations; but Phone overrides Caller.hang \(\) and this makes to use its own implementation, allowing to always call it even if it is cast to Callable object.

**Explicit method implementation**

The explicit interface method implementation allows to implement two interfaces that have methods \(not properties\) with the same name. Example:

```csharp
interface Foo {
 public abstract int m();
}

interface Bar {
 public abstract string m();
}

class Cls: Foo, Bar {
 public int Foo.m() {
  return 10;
 }

 public string Bar.m() {
  return "bar";
 }
}

void main () {
 var cls = new Cls ();
 message ("%d %s", ((Foo) cls).m(), ((Bar) cls).m());
}
```

Will output 10 bar.

