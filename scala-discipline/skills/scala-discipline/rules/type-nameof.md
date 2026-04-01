# type-nameof

> Any time there is a need to have the name of a type or a property as a string constant, use the scala-nameof library.

## Usage and examples

Add the library (if it's not in the project already) as "provided", because it's only needed during compilation and not at runtime:
```sbt
libraryDependencies += "com.github.dwickern" %% "scala-nameof" % "5.0.0" % "provided"
```

And import the package:
```scala
import com.github.dwickern.macros.NameOf._
```

Now you can use `nameOf` to get the name of a variable or class member:
```scala
case class Person(name: String, age: Int)

def toMap(person: Person) = Map(
  nameOf(person.name) -> person.name,
  nameOf(person.age) -> person.age
)
```
```scala
// compiles to:

def toMap(person: Person) = Map(
  "name" -> person.name,
  "age" -> person.age
)
```

To get the name of a function:
```scala
def startCalculation(value: Int): Unit = {
  println("Entered " + nameOf(startCalculation _))
}
```
```scala
// compiles to:

def startCalculation(value: Int): Unit = {
  println("Entered startCalculation")
}
```

Without having an instance of the type:
```scala
case class Person(name: String, age: Int) {
  def sayHello(other: Person) = s"Hello ${other.name}!"
}

println(nameOf[Person](_.age))
println(nameOf[Person](_.sayHello(???)))
```
```scala
// compiles to:

println("age")
println("sayHello")
```

Without having an instance of the type for nested case classes:
```scala
case class Pet(age: Int)
case class Person(name: String, pet: Pet)

println(qualifiedNameOf[Person](_.pet.age))
```
```scala
// compiles to:

println("pet.age")
```

You can also use `nameOfType` to get the unqualified name of a type:
```scala
println(nameOfType[java.lang.String])
```
```scala
// compiles to:

println("String")
```

And `qualifiedNameOfType` to get the qualified name:
```scala
println(qualifiedNameOfType[java.lang.String])
```
```scala
// compiles to:

println("java.lang.String")
```