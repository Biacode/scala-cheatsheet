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
Public by default. Generates `age()` accessor and `age_$eq` mutator

##### Specific 1
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