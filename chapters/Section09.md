# Chapter 9
## Functions

So far we've been defining behaviour as methods on objects, using the `def` keyword. They may or may not take method parameters, and they may or may not return things. And all we can do with them is call them. Here are a few examples of what we've learnt about methods:

```
class SomeMethods {

  // Return type is Unit, meaning it doesn't return anything to the caller
  def sayHello(): Unit = println("Hello")

  // Takes one parameter of type Int, and returns an Int to the caller
  def timesTwo(x: Int): Int = x * 2

  // Takes two parameters of type Int, and returns an Int to the caller
  def add(x: Int, y: Int): Int = x + y

  // Calling a method on another object
  // Note the syntax [object][period][method]
  def pourMilkFrom(carton: MilkCarton, amount: Int): Unit = carton.pourMilk(amount)

  // Calling another method within the same class
  // Note you don't specify the class name or put a period, just call the 'add' method directly
  def addThenTimesTwo(x: Int, y: Int): Int = {
    val added = add(x, y)
    timesTwo(added)
  }
}
```

That's all well and lovely, but it would be really useful to be able to treat methods like variables. This would allow us, for instance, to pass methods into other methods as parameters, or have methods return other methods. I'm going to show you some really cool examples of this later in the chapter, but as a simple and slightly pointless example let's say that we want to have a `calculate` method that will take a number and another method describing the calculation as parameters. What I'm trying to do is something like this:

```
object WontWork {

  def timesTwo(x: Int): Int = x * 2

  def square(x: Int): Int = x * x

  def calculate(number: Int, calculation: ???): Int = calculation(number)
}
```

```
class WontWorkSpec extends FlatSpec with Matchers {

  "Calculate" should "run the calculation method passed into it" in {
    WontWork.calculate(3, WontWork.timesTwo) shouldBe 6
    WontWork.calculate(3, WontWork.square) shouldBe 9
  }
}
```

Hopefully it's fairly clear from the test case what we're trying to do. If we pass a number and the `timesTwo` method into the `calculate` method, we want to assign the `timesTwo` method to the `calculation` method parameter. In the body of the `calculate` method we want to get the value of the `calculation` parameter, which is the `timesTwo` method, and then call that method passing in the number. We can pass in the `square` method instead, and it will square the number. In fact, we can see that the `calculation` parameter is a method that itself takes a single parameter (an `Int`). And we can also see that the `calculation` method has to return an `Int`, as this is what the `calculate` method returns. So we're not restricted to passing in `timesTwo` and `square` methods. We should be able to pass in *any* method that takes an `Int` and returns and `Int`.

You may need to reread that and let it sink in. If you're still not quite sure about it, don't worry. It should make more sense as we work through the chapter.

We have a bit of a problem with our code. The name of the object may have given you a hint that it wouldn't work! We need to specify the types of method parameters (e.g. `numer: Int`). So what's the type of the `calculation` parameter? It's actually a **function**. Hmm, that's nice, but what's a function? Well, functions are virtually identical to methods, except you can treat them like variables, pass them into methods (or other functions), and have methods (or other functions) return them. The syntax is a little bit different to a method. I'll rewrite our `timesTwo` method as a function, then go through what's going on:

```
val timesTwo: Int => Int = x => x * 2
```

Whoa, that looks complicated! Stay calm, and we'll break it down into easy chunks. Firstly you can see that this is a `val` rather than a `def`, and we know the syntax for `val`s:

```
val x: Int = 8
// [val] [name] [:] [type] [=] [value]
```

It's exactly the same for the function. So we have the word `val`, the name `timesTwo`, a `:`, the type `Int => Int`, an `=`, and the value `x => x * 2`. The only really new thing here is the `=>` sign. In my mind, when I see `=>` I think, "goes to". So the type is a function of `Int => Int`, or "Int goes to Int". In other words, it's a function that takes an Int parameter, and returns an Int. This is called the *type signature* of the function. The value is what gets executed when the function is called. In this example it's `x => x * 2`, or "x goes to x * 2". If you squint, it looks like a method, where the `x` on the left of the `=>` is the method parameter, and everything on the right of the `=>` is the method body.

Now we've got a function, it's very easy to call it. It looks just like calling a method:

```
object WillWork {

  val timesTwo: Int => Int = x => x * 2
  val square: Int => Int = x => x * x

  val result1 = timesTwo(3) // result1 is 6
  val result2 - square(3) // result2 is 9
}
```

Cool. Now we're got our functions that we want to pass into our `calculate` method we can fill in the type of the `calculation` parameter. The type is a function of `Int => Int`, and we write it like this:

```
def calculate(number: Int, calculation: Int => Int): Int = calculation(number)
```

So the whole thing now looks like this:

```
object WilWork {

  val timesTwo: Int => Int = x => x * 2
  val square: Int => Int = x => x * x

  def calculate(number: Int, calculation: Int => Int): Int = calculation(number)
}
```

```
class WillWorkSpec extends FlatSpec with Matchers {

  "Calculate" should "run the calculation method passed into it" in {
    WillWork.calculate(3, WillWork.timesTwo) shouldBe 6
    WillWork.calculate(3, WillWork.square) shouldBe 9
  }
}
```

Here's another little trick. Look at how we're calling the `calculate` method in the test case. We're passing in the two parameters in different ways. The first parameter is just the value (`3`), whereas the second parameter is the name of variable that the function is assigned to (`WillWork.timeTwo`). It would be annoying if we couldn't just pass values directly into methods, as it would mean we would have to create variables for every parameter, like this:

```
"Calculate" should "run the calculation method passed into it" in {
  val number = 3
  WillWork.calculate(number, WillWork.timesTwo) shouldBe 6
}
```

We can do the same thing with functions, passing the value of the function directly into the method. Remember, the value of the function is the part on the right side of the `=`. Let's add some more method calls to the test:

```
class WillWorkSpec extends FlatSpec with Matchers {

  "Calculate" should "run the calculation method passed into it" in {
    WillWork.calculate(3, WillWork.timesTwo) shouldBe 6

    WillWork.calculate(3, x => x * 2) shouldBe 6 // passing in the value of the timesTwo function

    WillWork.calculate(3, x => x * x) shouldBe 9
	
    // The function parameter doesn't have to be called x
    WillWork.calculate(3, input => input + 5) shouldBe 8

    // This won't work though
    WillWork.calculate(3, input => "Hello World") shouldBe "Hello World"
  }
}
```

Why doesn't the final statement work? Well we've defined the `WillWork.calculate` method to take a function which takes an `Int` and returns an `Int`, but we're calling the method with a function that takes an `Int` and returns a `String`. Have a go at updating the `WillWork` object to make this test pass. Remember, you can *overload* methods with the same name, so long as they have different parameter types. And a function of "Int goes to Int" is a different type to a function of "Int goes to String".

This is my solution:

```
object WillWork {
  // Don't need the twoTimes and square variables anymore

  def calculate(number: Int, calculation: Int => Int): Int = calculation(number)

  def calculate(number: Int, calculation: Int => String): String = calculation(number)
}
```

The functions we've seen so far have type signatures that take a single parameter, and return a single type. But there are other combinations too. Here are some examples of different type signatures:

```
// If your function doesn't take exactly 1 parameter, you have to wrap the parameters in parentheses.
// Here we're wrapping 0 parameters in parentheses.
val gimmeFive: () => Int = () => 2 + 3

// Here our function takes 2 parameters, so we've wrapped them in parenthesis, separated with a comma.
val add: (Int, Int) => Int = (x, y) => x + y

// This function takes no parameters, and returns nothing (Unit) to the caller.
val nada: () => Unit = () => println("Hello there!")

// This one takes a single String parameter, but returns nothing to the caller.
val printMe: String => Unit = me => println(me)

// Just to show you that you can use types other than String and Int. Here we return a Boolean.
val lengthOfStringEqualsNumber: (String, Int) => Boolean = (str, number) => str.length == number

// How about a Monster creator?
val monster: (String, Int) => Monster = (name, amount) => new Monster(name = name, health = amount)

// Or something a bit more complex?
val fight: (Player, Player) => Player = (player1, player2) => {
  // Use curly braces if the function body takes more than one line
  player1.attack(player2)
  player2.attack(player1)

  if (player1.health > player2.health) player1
  else player2
}
```

### How about you show me something useful?

I hope that was somewhat interesting, but I can understand if you're finding it hard to see the point of all this. Sure, as you get more experienced you'll start writing your own methods and functions that take or return functions. These are called *higher order functions* by the way. But let's start with some useful higher order functions that other people have written. In fact there are a tonne of them on classes that are built into Scala. I'm going to show you a higher order function called `map` that's defined on a class we already know about: the `List`.

Say we have a List of integers, and we'd like to do some kind of transformation on it to get a new list, where all the elements have been doubled. Here's a test case describing what we want:

```
"We" should "be able to double the elements in a list" in {
  val originalList = List[Int](1, 2, 3)
  val doubledList = ??? // Some code to create a new list with the elements doubled

  doubledList shouldBe List[Int](2, 4, 6)
}
```

As I mentioned, the `List` class has a method called `map` which takes a single parameter: a function of type `A => B`, where `A` is the type of the elements in the List (in this case `Int`), and `B` is the type of the elements we want in our new List (also `Int` in this example). It goes through the List, applying this function to each element and creating a new List from the results. We've already seen a function that takes an `Int`, returns an `Int`, and has the effect of doubling the input: our `timesTwo` function. Let's try passing that into the `map` method and see what happens:

```
"We" should "be able to double the elements in a list" in {
  val timesTwo: Int => Int = x => x * 2
  val originalList = List[Int](1, 2, 3)
  val doubledList = originalList.map(timesTwo)

  doubledList shouldBe List[Int](2, 4, 6)
}
```

And there we go! Using a function to easily transform every element in a list. As you know, we can also pass a function value directly into the `map` method. Here's another test with everything condensed.

```
"We" should "be able to square the elements in a list" in {
  List[Int](1, 2, 3).map(x => x * x) shouldBe List[Int](1, 4, 9)
}
```

The type of the elements in the transformed list don't have to be the same as the type in the original list:

```
"We" should "be able to transform a list to have elements of a different type" in {
  List[String]("Hello", "dear", "reader").map(x => x.length) shouldBe List[Int](5, 4, 6)
}
```

And `map` isn't the only higher order function defined for Lists. Oh no, there are plenty more. And they're also found on other data structures as well. Say we've got a List of strings, and we only want those over a certain length? There's a higher order function for that! It's called `filter`, and it takes a function with a type signature of `A => Boolean`, where `A` is the type of the elements in the List. It applies the function to each element in the list, and if the result of the function is `true` then the element gets added to the new list. Otherwise it's denied entry.

```
"We" should "be able to filter words longer than a certain value" in {
  val input = List[String]("Hello", "dear", "reader")
  
  input.filter(x => x.length > 4) shouldBe List[String]("Hello", "reader")
}
```