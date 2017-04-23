### Tip 1
Best things that yo can do for you
* Use byte code viewer
* Use IntelliJ `Desugar scala code` tool HotKey - `CTRL+ALT+D` (you will be surprised)

Scala base OOP components are: `class`, `trait`, `object`
```scala
class DemoClassA
trait DemoTraitA
object DemoObjectA
```

Some basic differences between Java
```
Java                                    | Scala
-------------------------------------------------------------------------------------
class (only non statics)                | class
-------------------------------------------------------------------------------------
class (only static)                     | object
-------------------------------------------------------------------------------------
field, method, constructor              | field (var, val), method (def), constructor
-------------------------------------------------------------------------------------
inner class                             | inner class (type projection)
-------------------------------------------------------------------------------------
-                                       | inner class (type dependent)
-------------------------------------------------------------------------------------
-                                       | inner class (type)
-------------------------------------------------------------------------------------
object = class instance                 | class instance
-------------------------------------------------------------------------------------
abs. class, interface (methods, Java 8) | abs. class, trait, (def, var, constructor)
-------------------------------------------------------------------------------------
-                                       | case class = class + auto-generated code
-------------------------------------------------------------------------------------
-                                       | case object = object + auto-generated code
-------------------------------------------------------------------------------------
-                                       | package object
-------------------------------------------------------------------------------------
-                                       | var (mutable)
-------------------------------------------------------------------------------------
final                                   | val (immutable)
-------------------------------------------------------------------------------------
```

Scala field sugar
```scala
class Person {
  var age: Int = 1
}
```
Public by default. Generates `age()` accessor and `age_$eq` mutator (Avoid using symbol `$` for naming.)\
The `Java` code below:
```java
public class Person {
    // property
    private int age = 1;
    // constructor
    public Person() {}
    // getter
    public int getAge() {
        return age;
    }
    // setter
    public void setAge(final int age) {
        this.age = age;
    }
}
```
Notice that in `Scala` terminology, the `Java` getter is called accessor\
and the setter is called mutator

### Specific 1
```scala
// This will produce compile time error, the class should be abstract
// In scala fields are not default initialized as in Java
class Person {
  var age: Int
}
```
If you want default initialization you sould use `_`\
So the right one is either
```scala
class Person {
  var age: Int = _
}
```
or make the class abstract
```scala
abstract class Person {
  var age: Int
}
```
### Specific 2
Because of type inference you can write class properties and methods like this
```scala
class Person {
  // not recommended
  // because scala's default access modifier is public
  // that mean the property is part of public API
  // and the user of your class should guess (at least read/open class) to work with it
  val age = 1
  // not recommended - read reason above
  def printAge() = {
      print(age)
  }
}
```
The recommended way
```scala
class Person {
  val age: Int = 1
  
  def printAge(): Unit = {
      print(age)
  }
}
```
You can't have `val` and `var` keywords in arguments
```scala
class Person  {
  var age: Int = 10
  
  // try to add val or var to `by` argument 
  def increaseAge(by: Int): Int = {
    age += by
    age
  }
}
```
Scala is `no ceremony` language.\
Notice in the code above there is no `;` at the end of `age += by` statement\
and `return` keyword at the end of a `def`\
In some cases we can avoid even `{}` braces for `def` like `def foo(x) = x + 1`\
We can rewrite code above like this:
```scala
class Person {
  var age: Int = 10
  def increaseAge(by: Int): Int = {
      age += by;
      return age
  }
}
```
### Specific 3
Make field private\
_Try 1 (fail try)_
```scala
class Person {
  // will be private mutator and private accessor
  private var age: Int = _
}
```
_Try 2_
```scala
class Person {
  // no accessor no mutator
  private[this] var age: Int = _
}
```
_Try 3_
```scala
class Person {
  private[this] var privateAge: Int = _

  // accessor
  def age: Int = {
    println("hello from accessor")
    privateAge
  }

  // mutator: fieldName_$eq(...)
  def age_$eq(value: Int): Unit = {
    println("hello from mutator")
    privateAge = value
  }
}
```
Lets write client code for `Try 3`
```scala
object Demo extends App {
  val person = new Person
  // accessor
  person.age
  // mutator
  person.age = 4
  // `parasite` mutators
  person.age_$eq(4)
  // another `parasite` mutator
  person.age_=(4)
}
```
The interesting output...
```
hello from accessor
res1: Int = 0

hello from mutator
hello from accessor
person.age: Int = 4

hello from mutator
res2: Unit = ()

hello from mutator
res3: Unit = ()
```
Note. The `age_=` is a method name. But the `JVM` do not allow to do this,\
so after compilation it will take name like `age_$eq`. And here where comes our parasites :)\
Also the `age_=` is recommended way.\
It's easy to say that we can not declare both of them `(age_$eq and age_=)` in same class.