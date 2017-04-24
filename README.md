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

|---------------------------------------------------------------------------------------|
| Java                                    | Scala                                       |
|---------------------------------------------------------------------------------------|
| class (only non statics)                | class                                       |
|---------------------------------------------------------------------------------------|
| class (only static)                     | object                                      |
|---------------------------------------------------------------------------------------|
| field, method, constructor              | field (var, val), method (def), constructor |
|---------------------------------------------------------------------------------------|
| inner class                             | inner class (type projection)               |
|---------------------------------------------------------------------------------------|
| -                                       | inner class (type dependent)                |
|---------------------------------------------------------------------------------------|
| -                                       | inner class (type)                          |
|---------------------------------------------------------------------------------------|
| object = class instance                 | class instance                              |
|---------------------------------------------------------------------------------------|
| abs. class, interface (methods, Java 8) | abs. class, trait, (def, var, constructor)  |
|---------------------------------------------------------------------------------------|
| -                                       | case class = class + auto-generated code    |
|---------------------------------------------------------------------------------------|
| -                                       | case object = object + auto-generated code  |
|---------------------------------------------------------------------------------------|
| -                                       | package object                              |
|---------------------------------------------------------------------------------------|
| -                                       | var (mutable)                               |
|---------------------------------------------------------------------------------------|
| final                                   | val (immutable)                             |
|---------------------------------------------------------------------------------------|
```

Scala field sugar
```scala
class Person {
  var age: Int = 1
}
```
Public by default. Generates `age()` accessor and `age_$eq` mutator (Avoid using symbol `$` for naming)\
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
Notice that in `Scala` terminology, the `Java` getter is called accessor and the setter is called mutator

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
Note. The `age_=` is a method name. But the `JVM` do not allow to do this.\
After compilation it will take name like `age_$eq`. And here where _parasites_ comes from :)\
Also the `age_=` is recommended way to do that.\
It's easy to notice that we can not declare both of them `age_$eq` and `age_=` in same class.

### Specific 4
Assume were writing Scala class which will be used from Java, and we want to keep Java conventions.

_Method 1_
```scala
class Person {
  private [this] var age: Int = _

  def getAge(): Int = this.age

  def setAge(age: Int): Unit = this.age = age
}
```
_Method 2_
```scala
import scala.beans.BeanProperty
class Person {
  @BeanProperty var age: Int = _
}
```
_Method 3_
```scala
import scala.beans.BeanProperty
class Person(@BeanProperty var age: Int)
```
And welcome back to ugly world of Java

### Specific 5
Primary constructor (no fields)
```scala
class Person(age: Int)
```
If we try to access `age` with accessor or mutator we will get compile time error.
```scala
val person = new Person(45)
// accessor
person.age
// mutator
person.age = 15
```
Useful if
```scala
class Person(_age: Int) {
  var age: Int = _age
}
```
It's so common that we have following
```scala
// var or val, both are acceptable
class Person(var age: Int)
```
It also worth to mention that we can have access modifiers as well
```scala
// here we have
// protected class Person
// which has protected constructor
// which has private field age, and public field name
protected class Person protected (private var age: Int, val name: String)
```

### Specific 6
Auxiliary constructors
```scala
// primary constructor
class Person(var name: String, var age: Int) {
  // auxiliary 1
  def this(name: String) {
    this(name, Person.DEFAULT_AGE)
  }
  // auxiliary 2
  def this(age: Int) {
    this(Person.DEFAULT_NAME, age)
  }
  // auxiliary 3
  def this() {
    this(Person.DEFAULT_NAME, Person.DEFAULT_AGE)
    // also we can call this(Person.DEFAULT_AGE) which will call primary constructor
  }
}
// companion object of class Person
object Person {
  val DEFAULT_NAME = "Arthur"
  val DEFAULT_AGE = 45
}

def main(args: Array[String]): Unit = {
  val person = new Person()
  person.name
  person.age
}
```
Here the `object Person` is called companion object which is constant holder in out case.\
We can not avoid primary constructor call.

For this common construction we can:
```scala
// default parameters
class Person(var name: String = "Arthur", var age: Int = 45)

new Person("Vasya")
new Person(age = 15)
new Person("Vasya", 15)
new Person(age = 15, name = "Vasya")
new Person()
```
Notice the `new Person(age = 15, name = "Vasya")` so we can call constructors and methods in any order, just by mentioning the parameter name.\
This is called `named parameters`.
```scala
def foo(name: String, age: Int): String = {
  s"name = $name, age = $age"
}

println(foo(age = 45, name = "Arthur"))
```
### Specific 7
```

|---------------------------------------------------------------------------------------|
| Scala objects                       | Java static                                     |
|---------------------------------------------------------------------------------------|
| companion object = companion module | class static members (Person.MAX_AGE)           |
|---------------------------------------------------------------------------------------|
| single object (not companion)       | utility functions/constants (Math.sin, Math.PI) |
|---------------------------------------------------------------------------------------|
| singleton                           | -                                               |
|---------------------------------------------------------------------------------------|
| package object                      | -                                               |
|---------------------------------------------------------------------------------------|

```
Note: The companion object should be at the same file with companion class or trait.\
Utility functions/constants
```scala
object Demo {
  import IntLib.max
  def main(args: Array[String]): Unit = {
    println(max(7, 3))
  }
}
object IntLib {
  val MAX_INT = java.lang.Integer.MAX_VALUE

  def max(x: Int, y: Int): Int = if (x > y) x else y
}
```
Notice `max()` function. In Java you would need to `import static IntLib.max`. In scala it's just `import IntLib.max`

If in companion object we have method called `apply`, and the `apply` method returns companion class instance, then we have following syntax sugar.
```scala
class Person(val name: String, val age: Int)

object Person {
  def apply(name: String, age: Int): Person = {
    new Person(name, age)
  }
}

var personWithoutSugar1 = new Person("Mike", 45)
var personWithoutSugar2 = Person.apply("Mike", 45)
val personWithSugar = Person("Mike", 45)
```
The Java alternative could be:
```java
package com.biacode.scala_cheatsheet;
import static com.biacode.scala_cheatsheet.JPerson.JPerson;

public class JPerson {
    public final String name;
    public final int age;

    public JPerson(final String name, final int age) {
        this.name = name;
        this.age = age;
    }

    public static JPerson JPerson(final String name, final int age) {
        return new JPerson(name, age);
    }
}

final JPerson person0 = new JPerson("Mike", 45);
final JPerson person1 = JPerson.JPerson("Mike", 45);
final JPerson person2 = JPerson("Mike", 45);
```
The visibility:
```scala
// companion class - can see private `objectPrivate` of companion object
class PrivateDemo {
  private val classPrivate = 0
  val tmp: Int = PrivateDemo.objectPrivate
}

// companion object - can see private `classPrivate` of companion class
object PrivateDemo {
  private val objectPrivate = 0
  val tmp: Int = new PrivateDemo().classPrivate
}
```
### Specific 8
What is interesting in scala that we can do following:
```scala
// companion class
class Person {
  val name: String = _
  val age: Int = _

  def getName: String = this.name
}

// companion object
object Person {
  val name: String = _
  val age: Int = _

  def getName: String = this.name
}
```
Which alternative in scala is:
```java
public class JPerson {
    public final String name;
    public final int age;

    public static final String name;
    public static final int age;

    public JPerson(final String name, final int age) {
        this.name = name;
        this.age = age;
    }
}
```
Which will throw compile time exception like - variable name is already defined in scope blah blah...\
I guess it's mainly because we can access the class static properties either:
```java
new MyClass().THE_STATIC_PROPERTY
```
and/or
```java
MyClass().THE_STATIC_PROPERTY
```
And the Java compiler can not assume which property we calling (in case of `new MyClass().THE_STATIC_PROPERTY`).\
But this is just language dessign and nothing else. The static properties and instance properties has their own memory location, which is separate.
