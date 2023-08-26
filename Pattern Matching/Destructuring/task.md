# Destructuring

Destructuring in Scala refers to the practice of breaking up an instance of a given type into its constituent parts. You can think of it as the inversion of construction. In a constructor, or an apply method, we use a collection of parameters to create a new instance of a given type.  When destructuring, we start with an instance of a given type and break it up into values that, at least in theory, could be used again to create an exact copy of the original instance. But also, ust as an apply method can be a smart constructor that performs certain complex operations before creating an instance, we can implement a custom method, called unapply, to break the original instance in a smart way. It's a very powerful and expressive feature of  Scala, and it's heavily used in idiomatic Scala code.
The unapply method should be defined in the companion object. It usually takes the instance of the associated class as its only argument, and returns an option of what’s inside the instance. In the simplest case, this will be just fields of  the class: one, if there is only one field in the class, and otherwise a pair, a triple, a quadruple, and so on. Scala automatically generates simple unapply methods for case classes. In such case, unapply will just break the given instance into a collection of its fields, as in the following example:
```scala 3
case class Person(name: String, age: Int)
val john = Person("John", 25)
// ...
val Person(johnsName, johnsAge) = john
println(s"$johnsName is $johnsAge years old.")
```

As you can notice, similar to how we don't need to explicitly write apply to create an instance of the Person case class, we also don't need to explicitly write unapply to break an instance of the Person case class back into its fields: johnsName and johnsAge.
But you will not see this way of using destructuring very often in Scala. After all, if you already know exactly what case class you have, and you only need to read its public fields, you can do it directly - here, in this example, with john.name and john.age. Instead, unapply becomes much more valuable when used together with pattern matching.
Let's start by defining, once again, the Color enum and the Cat case class.

```scala 3
enum Color:
case White, Ginger, Black

import Color.*
case class Cat(name: String, color: Color, age: Int)
```

Now, let's create five instances of Cat:

```scala 3
val mittens  = Cat("Mittens", Black, 2)
val fluffy   = Cat("Fluffy", White, 1)
val ginger   = Cat("Ginger", Ginger, 3)
val snowy    = Cat("Snowy", White, 1)
val midnight = Cat("Midnight", Black, 4)
```

We have two cats (Fluffy and Snowy) that are one year old, and three cats  (Mittens, Ginger, and Midnight) that are older than one year.
Next, let's put these cats in a Seq:

```scala 3
val cats = Seq(mittens, fluffy, ginger, snowy, midnight)
```

Finally, we can use pattern matching and destructuring to check the age of each cat and print out the appropriate message:

```scala 3
cats.foreach {
case Cat(name, color, age) if age > 1 =>
println(s"This is a $color adult cat called $name")
case Cat(name, color, _) =>
println(s"This is a $color kitty called $name")
}
```

In this code, we're using pattern matching to destructure each Cat object. We're also using a guard (if age >  1) to check the age of the cat. If the age is greater than one, we  print out the message for adult cats. If the age is not greater than one (i.e., it's one or less), we print out the message for kittens. Note that in the second case expression, we're using the wildcard operator _ to ignore the age value, because we don't need to check it – if a cat instance is destructured in the second case, it means that the age of the cat was already checked in the first case, and failed that test.
Also, if we wanted to handle a case where one of the fields has a single constant value (not like in the first case above, where any age larger than 1 suits as well), we can simply substitute it for the field:

```scala 3
cats.foreach {
case Cat(name, _, 2) =>
println(s"$name is exactly two years old")
case Cat(name, color, age) if age > 1 =>
println(s"This is a $color adult cat called $name")
case Cat(name, color, _) =>
println(s"This is a $color kitty called $name")
}
```