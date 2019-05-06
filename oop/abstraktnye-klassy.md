# Абстрактные классы

Другой модификатор для методов называется abstract. Он позволяет объявить метод, который не реализуется в том же классе, а в классе, который наследует от того, в котором такой метод объявлен. Это дает гарантию, что все подклассы будут иметь подобный метод, хотя реализация может быть у всех разная.

Класс, содержащий абстрактные методы тоже должен быть абстрактным, чтобы не было попыток создания его экземпляра.

```csharp
public abstract class Animal : Object {

    public void eat() {
        stdout.printf("*ням ням*\n");
    }

    public abstract void say_hello();
}

public class Tiger : Animal {

    public override void say_hello() {
        stdout.printf("*Рёв*\n");
    }
}

public class Duck : Animal {

    public override void say_hello() {
        stdout.printf("*Кряканье*\n");
    }
}
```

Разработка абстрактных методов должна быть помечена модификатором override. Свойства тоже могут быть абстрактными.

**Virtual Methods**

A virtual method allows to define default implementations to abstract classes and allows to derived classes to override its behavior, this is different than hiding methods.

```csharp
public abstract class Caller : GLib.Object {
   public abstract string name { get; protected set; }
   public abstract void update (string new_name);
   public virtual bool reset ()
   {
      name = "No Name";
      return true;
   }
}

public class ContactCV : Caller
{
   public override string name { get; protected set; }
   public override void update (string new_name)
   {
     name = "ContactCV - " + new_name;
   }
   public override bool reset ()
   {
      name = "ContactCV-Name";
      stdout.printf ("CotactCV.reset () implementation!\n");
      return true;
   }
}

public class Contact : Caller {
   public override string name { get; protected set; }
   public override void update (string new_name)
   {
     name = "Contact - " + new_name;
   }
   
   public static void main ()
   {
      var c = new Contact ();
      c.update ("John Strauss");
      stdout.printf(@"Name: $(c.name)\n");
      c.reset ();
      stdout.printf(@"Reset Name: $(c.name)\n");
      
      var cv = new ContactCV ();
      cv.update ("Xochitl Calva");
      stdout.printf(@"Name: $(cv.name)\n");
      cv.reset ();
      stdout.printf(@"Reset Name: $(cv.name)\n");
      stdout.printf("END\n");
   }
}
```

As you can see in the above example, Caller is an abstract class defining both an abstract property and a method, but adds a virtual method which can be overridden by derived classes. Contact class implements abstract methods and properties of Caller, while using the default implementation for reset\(\) by avoiding to define a new one. ContactCV class implements all abstract definitions on Caller, but overrides reset\(\) so as to define its own implementation.

