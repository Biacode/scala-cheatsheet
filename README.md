### Tip 1
Best thing that yo can do for you is:
* Use byte code viewer
* Use IntelliJ `Desugar scala code` tool.

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
In general, `Scala` programming language mostly based on syntax sugar.\
For example there is no "real" functions, scala has `Function1` which accepts 1 argument,
`Function2` with 2 arguments and so on... Until `Function22`.\
Same situation with tuples `Tuple1`, `Tuple2`, ..., `Tuple22`.\
Of course this is expected behavior, and there is no magic...
At the end of the day it compiles to `Bytecode` which execution platform is `JVM`.

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
Because of type inference we can write class properties and methods like this
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
We can't have `val` and `var` keywords in arguments. Also we can't infer the method argument types.
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

### Specific 9
You can not use object name as type:
```scala
trait T
class C
object O

val x: T = null
val y: C = null
val z: O = null // Illegal
val w: O.type = null

def f(x: T): T = ???
def g(x: C): C = ???
def h(x: O): O = ??? // Illegal
def q(x: O.type): O.type = ???

def r(arg: Any): String = arg match {
  case _: T => "T"
  case _: C => "C"
  case _: O => "O" // Illegal
  case _: O.type => "O"
}
```
In example above the `val z: O = null`, `def h(x: O): O = ???` and `case _: O => "O"` are illegal.
Since the objects are not types itself. If we want to use them as type, we should use `Object.type` construction.\
We can have constructions like `case O => "O"` which is fullly acceptable.

### Specific 10
Methods - with and without parenthesis:
```scala
class Demo {
  def f0 = 1

  def f1: Int = 1

  def g0() = 1

  def g1(): Int = 1
}

new Demo().f0
new Demo().f0() // Illegal

new Demo().g0
new Demo().g0()
```
In example above, we can say `f0 = f1` and `g0 = g1` but `{f0, f1} != {g0, g1}`.
The `new Demo().f0()` expression is illegal.

Read the [Uniform access principle](https://en.wikipedia.org/wiki/Uniform_access_principle)
and [Scala Glossary](http://docs.scala-lang.org/glossary/?_ga=1.169830245.530428506.1486141446#uniform-access-principle) for more details.

In short. Suppose we have class person, which has property `age` like in example below:
```scala
class Person {
  var age: Int = 45
}

val person = new Person
println(person.age)
```
If some day we decide to make `age` computable and not just stored value, we can simple replace `var` with `def`:
```scala
class Person {
  def age: Int = {
    val x = 44
    val y = x + 1
    y
  }
}

val person = new Person
println(person.age)
```
Notice that the client code will not crush, and everything works as expected.
For example we can return cached value of `age` computation etc...

### Specific 11
Imports.\
What is really make me happy in scala, is imports. There is various ways to import `components` in scala:
```scala
import java.util.ArrayList
import java.util.{HashMap, TreeSet}
import java.util._
```
The last one analog in java is `import java.util.*`.\
And finally the feature we're really missing in Java is aliasing, or let's call it rename.
```scala
import java.lang.Double.{isInfinite => isInf, isNaN}
```
Here the `isInfinite` will be renamed to `isInf`.\
We can apply same aliasing to classes, traits etc...\
Also we can use `import` in any scope. For instance:
```scala
def foo(): Unit = {
  import java.util.ArrayList
  println("boo")
}
```
We can do even this:
```scala
class Person(val name: String, val age: Int)

object Demo {
  def f(person: Person): Unit = {
    import person._
    println("name: " + name + " age: " + age)
  }
}
```
Also this:
```scala
import java.util.{ArrayList => _, _}
```
Which means import everything from `java.util` excluded `ArrayList`.\
We can even rename package names and so on...

### Specific 12
In java we have following overloaded `+` operator:
```java
public class DemoApp {
    public static void main(String[] args) {
        // overloaded versions of '+' operator
        String s = "A" + "B";
        int k = 123 + 456;
        long l = 12L + 34L;
        float f = 1.f + 2.f;
        double d = 1.2 + 3.4;
    }
}
```
Of course the `String + String` is not same as `int + int`.\
We can assume that there `+` operator has lot of overloaded versions for different types.\
For instance. In strings it's often called `concatenation` which concates two string literals etc...\
In many cases, operators can be:
```
|---------------------------------------------------------|
| prefix `++1`      | (when operator is before operand)   |
|-------------------|-------------------------------------|
| infix   `1+2`     | (when operator is between operands) |
|-------------------|-------------------------------------|
| postfix `1++`     | (when operator is after operand)    |
|---------------------------------------------------------|
```
In scala we call them `operations`.\
See [Prefix, Infix, and Postfix Operations](https://www.scala-lang.org/files/archive/spec/2.11/06-expressions.html#prefix-infix-and-postfix-operations)\
If we try to investigate Scala's scala.Int class, we can find all our known "operators" as method definitions.\
For instance:
```scala
def /(x: Int): Int
def %(x: Short): Int
def *(x: Byte): Int
def -(x: Float): Float
// ...
```
So we can say.
1. In scala there is no predefined 38 operators like in Java.
2. We can "overload operations" as we needed.
3. Also we can make our "new" operators like - `<<:><<` (just make sure the guy who will read this can not find you...)

If we try to reflect the class and print the method names.
```java
import java.lang.reflect.Method;
for (final Method method : MainApp.class.getDeclaredMethods()) {
    System.out.println(method);
}
```
The output will be something like this:
```
$plus
$minus
$times
$div
$bslash
$colon$colon
$minus$greater
$less$tilde
```
Because the JVM does not accepts method names with such symbols.

### Specific 13
In scala, method calls that contains only 1 arity can be called as infix form with `pointless style`.
```scala
case class I(k: Int) {
  def add(that: I): I = I(this.k + that.k)

  def +(that: I): I = I(this.k + that.k)
}

val x0 = I(1).add(I(2))
val x1 = I(1) add I(2) // pointless
val x2 = I(1).+(I(2))
val x3 = I(1) + I(2) // pointless
```
In scala standard lib you can see a lot of examples of this semantics. For instance:
```scala
1.to(10)
1 to 10 // pointless infix form

for (k <- 1 to 10) {
  println(s"k = $k")
}
```
or
```scala
var map0 = Map("France" -> "Paris")
map0 += "Japan" -> "Tokyo" // pointless

var map1: Map[String, String] = Map.apply(new ArrowAssoc("France").->("Paris"))
map1.+=(new ArrowAssoc("Japan").->("Tokyo"))
```
NOTE: The `pointless style` is not same as `point free style` which is widely used in functional programming languages.\
In point free style we're not using arguments at all.
```scala
val cos: Double => Double = Math.cos
val sin: Double => Double = Math.sin

val f: Double => Double = x => cos(sin(x))
val g: Double => Double = cos compose sin // point free style
```
or
```scala
// not pointless, not point free
val f0: Int => Int = x => 1.+(x)
// pointless, not point free
val f1: Int => Int = x => 1 + x
// not pointless, point free
val f2: Int => Int = 1.+(_)
// pointless, point free with placeholder
val f3: Int => Int = 1 + _
// pointless, point free without placeholder
val f4: Int => Int = 1 +
```
### Specific 14
Operator precedence in scala.
```scala
case class I(k: Int) {
  def add(that: I): I = I(this.k + that.k)

  def mul(that: I): I = I(this.k * that.k)
}

// 1 add 2 mul 3 = 1 + 2 * 3 = 7
println(I(1) add I(2) mul I(3))
// (1 add 2) mul 3 = (1 + 2) * 3 = 9
println((I(1) add I(2)) mul I(3))
// 1 add (2 mul 3) = 1 + (2 * 3)
println(I(1) add (I(2) mul I(3)))
```
As you can see we have problem in `println(I(1) add I(2) mul I(3))` which returns `9` and not `7` as we expected.

We have several ways to fix this.\
First:
 ```scala
//NOT recommended
println(I(1) add I(2).mul(I(3)))
// recommended
println(I(1).add(I(2) mul I(3)))
```
In scala method calls with `.` dot has more priority relatively to infix calls.
Which means in `println(I(1) add I(2).mul(I(3)))` example `I(2).mul(I(3))` will be called first,
and then the result will be applied to `I(1) add ...`. But using this is not recommended,
because it's hard for your code users to read and understand it. It's not really natural.

So what is really best solution for this?
As you already may guess, scala solves this problems in mathematical manner as well.
```scala
case class I(k: Int) {
  def add(that: I): I = I(this.k + that.k)
  def mul(that: I): I = I(this.k * that.k)

  def +(that: I): I = I(this.k + that.k)
  def *(that: I): I = I(this.k * that.k)
}

// 1 add 2 mul 3 => (1 add 2) mul 3
println(I(1) add I(2) mul I(3)) // res = 9
// 1 + 2 * 3 => 1 + (2 * 3)
println(I(1) + I(2) * I(3)) // res = 7
```
Even though the scala `point less` and `infix form` calls are left associative the priority of predefined symbols are differs.

For more details please read
[scala specification about infix operations](https://www.scala-lang.org/files/archive/spec/2.11/06-expressions.html#infix-operations)
### Specific 15
Operator associativity in scala.

In [infix operations specification](https://www.scala-lang.org/files/archive/spec/2.11/06-expressions.html#infix-operations)
page we can see
> The associativity of an operator is determined by the operator's last character. Operators ending in a colon `:' are right-associative. All other operators are left-associative.

In two words we cab say. If the priority is about first symbol, the associativity is about last symbol of infix operation.
If last character of operation ending with a `:` then we have right association. Anything else is left associative.
```scala
case class I(k: Int) {
  def ++(that: I): I = I(this.k + that.k)

  def +:(that: I): I = I(this.k + that.k)
}

// 1 ++ 2 ++ 3 ++ 4
// ((1 ++ 2) ++ 3) ++ 4
println(I(1) ++ I(2) ++ I(3) ++ I(4))

// 1 +: 2 +: 3 +: 4
// 1 +: (2 +: (3 +: 4))
println(I(1) +: I(2) +: I(3) +: I(4))
```
If you are nerd enough and have reade the this part of specification.
>A left-associative binary operation e1;op;e2e1;op;e2 is interpreted as e1.op(e2)e1.op(e2). If opop is right-associative, the same operation is interpreted as { val xx=e1e1; e2e2.opop(xx) }, where xx is a fresh name.
```scala
// 1 ++ 2 ++ 3 ++ 4
// ((1 ++ 2) ++ 3) ++ 4
println(I(1) ++ I(2) ++ I(3) ++ I(4))
println(I(1).++(I(2)).++(I(3)).++(I(4)))

// 1 +: 2 +: 3 +: 4
// ((4 +: 3) +: 2) +: 1
println(I(1) +: I(2) +: I(3) +: I(4))
println(I(4).+:(I(3).+:(I(2) +: I(1))))
```
Again, do this only if you are sure if you are too far from a guy who will read your code.