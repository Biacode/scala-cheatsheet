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
Public by default. Generates `age()` accessor and `age_$eq` mutator\
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