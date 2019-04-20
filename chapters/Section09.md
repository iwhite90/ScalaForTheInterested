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

We're only scratching the surface here. If you want a whole load more information on Lists and the ways you can use them, a quick Google search for "Scala Lists" will get you started down the rabbit hole.

A great feature of these methods are that they don't alter the existing List, but return a new List. This means that we can call another method on the returned List in order to transform the original List in several ways. Let's say we have a List of words, and we'd like to know the total number of letters in the words that start with a letter that comes after "m" in the alphabet. We can start by filtering the List into a new List that only contains the words starting with a letter after "m".

```
"We" should "be able to find out the number of letters in words starting after m" in {
  val originalList = List[String]("the", "cow", "jumped", "over", "the", "moon")
  val filteredList = originalList.filter(word => word > "m")

  filteredList shouldBe List[String]("the", "over", "the", "moon")
}
```

Then we can map the words to their lengths:

```
"We" should "be able to find out the number of letters in words starting after m" in {
  val originalList = List[String]("the", "cow", "jumped", "over", "the", "moon")
  val filteredList = originalList.filter(word => word > "m")
  val mappedList = filteredList.map(word => word.length)

  mappedList shouldBe List[Int](3, 4, 3, 4)
}
```

And to finish we just need to sum up the values in the `mappedList`. Remember our `ListOps` class from the last chapter? We had a handy method that does just that.

```
"We" should "be able to find out the number of letters in words starting after m" in {
  val originalList = List[String]("the", "cow", "jumped", "over", "the", "moon")
  val filteredList = originalList.filter(word => word > "m")
  val mappedList = filteredList.map(word => word.length)
  val totalWords = ListOps.sumList(mappedList)

  totalWords shouldBe 14
}
```

We could have written one big function that takes in the List of words, iterates over the elements looking for words starting after "m", getting the size of the word, and adding it to counter of some sort. But that would be a very specific piece of code that is only good for this one use case. Using smaller functions and composing them together allows them to be used in many different cases. You can think of the big function as a cast iron model of a car, and the smaller functions as Lego bricks that you could use to create a car, but could also create lots of other things as well. This concept is so important to the art of coding that it has its own name - the **Single Responsibilty Principal**. There are nuances to this, but basically you can think of it as keeping your classes, objects, methods and functions small, and focussed on doing just one thing well.

Our test code works, but I feel it's a bit messy. I'd like to **refactor** the test. Refactoring just means changing (hopefully improving) the code, without affecting what the code actually does. In this instance, there are two things we can change to make the code more readable. Firstly, there's actually a method defined on the List class that will sum up the elements in a List, so we can use that instead of our own `sumList` method. And secondly, we don't need all these intermediate `val`s to assign the stages of the transformation to. We can just chain the methods together, like so:

```
"We" should "be able to find out the number of letters in words starting after m" in {
  val originalList = List[String]("the", "cow", "jumped", "over", "the", "moon")
  
  originalList.filter(word => word > "m").map(word => word.length).sum shouldBe 14
}
```

If it's hard to read all on one line, you can split the method calls onto separate lines, like this:

```
"We" should "be able to find out the number of letters in words starting after m" in {
  val originalList = List[String]("the", "cow", "jumped", "over", "the", "moon")
  
  originalList.filter(word => word > "m")
              .map(word => word.length)
              .sum shouldBe 14
}
```

That looks much nicer to me.

I just have to say one thing about the `sum` method. If you're eagle eyed, you may have noticed that it doesn't have any parentheses after it. Scala has a rule that if a method doesn't take any parameters and it returns a value, then you shouldn't use empty parentheses when you define or call the method. This is because a method that doesn't take any parameters and returns a value looks just like a variable to the caller of the method, so the caller shouldn't have to distinguish between whether they are calling a method or just referencing the value of a variable. This is what I mean:

```
object Demo {
  val x = 10
  def y() = 15
  def z = 20
}
```

To use these, you would write:

```
Demo.x // 10
Demo.y() // 15
Demo.z // 20
```

From the user's point of view, they're all doing the same thing: giving us an integer. The user doesn't need to know that `y` and `z` are methods, so writing `z` without parentheses lets the user treat it the same as a variable.

Ok, we've seen some of the higher order methods operating on Lists of strings and integers. Well, they're not restricted to working on these built in types. We can just as easily use them to transform Lists of types that we've created ourselves. Maybe we're writing a game, and have a List to keep track of all the monsters. At some point in the game we want to get rid of all the monsters with low health. Here's a monster:

```
class Monster(name: String, health: Int)
```

And an object to handle our monsters:

```
object MonsterHandler {
  def cullMonsters(monsters: List[Monster]): List[Monster] = ???
}
```

Let's write a test for our `cullMonsters` method:

```
class MonsterHandlerSpec extends FlatSpec with Matchers {

  "cullMonsters" should "get rid of all monsters with health below 10" in {
    val originalMonsters = List[Monster](
	  new Monster("Barry", 20),
	  new Monster("Helen", 5),
	  new Monster("Jimmy", 15))

    // Let's check that we just have Barry and Jimmy after culling
    MonsterHandler.cullMonsters(originalMonsters)
                  .map(monster => monster.name) shouldBe List[String]("Barry", "Jimmy")
  }
}
```

Have a go at filling in the implementation for the `cullMonsters` method. Here's my solution:

```
object MonsterHandler {
  def cullMonsters(monsters: List[Monster]): List[Monster] = {
    monsters.filter(monster => monster.health > 10)
  }
}
```

### Maps and case classes

Now we've seen that Lists have useful methods built into them that allow us to transform the data in them, it's time to look at a slightly more complex data structure: the **Map**. Don't get confused between the *data structure* Map, with a big 'M', and the **method** map with a small 'm'. Although they sound the same, they're not related. In fact, you can call the map method on Map data structures, in the same way that you can call the map method on List data structures. It wil all become clear shortly!

A Map is a data structure that stores a mapping between one value and another value. An example of a Map would be a phone book, where people are mapped to phone numbers. If you want to find someone's number, you look up their name in the book, and next to their name is their number.

It's perfectly possible to create a phone book using a good old List. In fact, that's what we're going to do first, before I tell you why Maps are better! We're going to model each pair of values (name and number) using an **Abstract Data Type**. This is just a class with data, but no methods, and helps us to reason about our program more easily by having things grouped and named nicely. In Scala we use **case classes** to represent Abstract Data Types (ADTs). They're extremely similar to normal classes, except Scala adds some useful functionality to them that makes them good for using as ADTs. We'll see some of this functionality, such as pattern matching, later. Create a new class in the `src/main/scala` folder called `PhoneBookEntry`, then replace the generated contents of the file with the following:

```
case class PhoneBookEntry(name: String, number: Int)
```

There's our ADT. It should look very familiar to you. It exactly the same way you'd declare a class, except it has the word 'case' at the beginning, and there is no class body. Here's how you create an instance of your case class:

```
object Program extends App {

  val x = PhoneBookEntry("Ian", 13452) // Note, don't use the 'new' keyword
  val y = PhoneBookEntry(name = "Anna", number = 64584) // You can use named parameters as well
}
```

And we can access the values of our instances just like we do with regular classes:

```
object Program extends App {

  val x = PhoneBookEntry("Ian", 13452)
  val friendsName = x.name
  val friendsNumber = x.number
}
```

So how would we go about using this to create our phone book? We know we're going to use a List, but the users of our phone book shouldn't have to care about how we've implemented it, so we're going to wrap the List inside another class. In that way, users should be able to call methods on our class, and if we decide to change the List to something else (say a Map) then the users won't have to change their code to make it work. It's very common, and good practice to hide the details of how your classes work, and strictly control what the users of your class can see. I think it makes sense to call our class `PhoneBook`, and for this example it will already be populated with names and numbers, and users will only be able to look up people's numbers from it. Here's the functionality described in a test:

```
class PhoneBookSpec extends FlatSpec with Matchers {

  "Querying the phone book with a name" should "return that person's number" in {
    val phoneBook = new PhoneBook
    phoneBook.lookup("Ian") shouldBe 13452
  }
}
```

Note that we're modelling the numbers as `Int`s, so you can't start them with a zero.

If you're feeling brave, stop reading here and have a go at implementing the class yourself. Just make up a few names and numbers, but make sure to include the name/number pair needed to make the test pass.

You know everything you need to do it.

Go on...

OK, great! Hopefully you have something like this:

```
class PhoneBook {

  val entries: List[PhoneBookEntry] = List(
    PhoneBookEntry("Barry", 4637),
    PhoneBookEntry("Jenny", 43256),
    PhoneBookEntry("Rover", 986),
    PhoneBookEntry("Ian", 13452),
    PhoneBookEntry("Spock", 76438)
  )

  def lookup(query: String): Int = {
    // There are many ways of doing this. Yours might be different to mine.
    val listWithCorrectEntry = entries.filter(entry => entry.name == query)
    val correctEntry = listWithCorrectEntry.head
    correctEntry.number
  }
}
```

However you implemented this, the very nature of a List means that we have to visit each element in order to find out whether it's the one we're looking for. The way I've done it, using the `filter` method, means that even if the entry we're looking for is the first one in the List the program will check every single element. There are ways of writing this so that it stops as soon as it finds a matching element, but still it's not a great solution. What if we've got a million entries in our book, and the person we're looking for happens to be at the end? That's a lot of wasted processing time. There's a better way of doing this, and I know that you've guessed it. Maps!

Remember that data is stored in memory, and if the program knows the memory address of the data it can access it directly. Maps work by knowing the memory addresses of all the elements in them, so they can get any element in one go. They use a trick called *hashing* to do this. In this context, hashing is a clever way of taking some value and converting it into a memory address. Hashing the same value will always produce the same address. And hashing different values will usually produce different addresses. We know that Maps map one thing to another (e.g. names to phone numbers). The first thing is called the **key**, and the thing the key maps to is called the **value**. So in our example, we're going to have a Map of names (keys) to phone numbers (values). When a key and value are added to a Map, the computer hashes the key to produce the memory address that the key and value should be stored at. And when you ask the Map for the value of a key, it again hashes the key to get the memory address so it can access the value directly. Neat!

The syntax for creating a Map is similar to that for a List, but one obvious difference is that a Map contains two types, rather than just one in a List. You write the types in square brackets, separated by a comma. The type of the key goes first, followed by the type of the value. So in our phone book the keys are `String`s and the values are `Int`s:

```
val entries: Map[String, Int] = ???
```

There are a couple of ways of specifying the elements of a Map. I like using the arrow syntax, as it makes it clear that a key is being mapped to a value. Recreating our phone book entries as a Map rather than a List, it looks like:

```
val entries: Map[String, Int] = Map(
  "Barry" -> 4637,
  "Jenny" -> 43256,
  "Rover" -> 986,
  "Ian" -> 13452,
  "Spock" -> 76438
)
```

And the simplest way to query a Map is to pass the key to the Map in the same way that you'd pass the index to a List:

```
val friendsNumber = entries("Ian")
```

Now we can refactor our `PhoneBook` class to use a Map instead of a List, and note how we don't have to change the test as it wasn't aware of the implementation details of the class:

```
class PhoneBook {

  val entries: Map[String, Int] = Map(
    "Barry" -> 4637,
    "Jenny" -> 43256,
    "Rover" -> 986,
    "Ian" -> 13452,
    "Spock" -> 76438
  )

  def lookup(query: String): Int = entries(query)
}
```

Not only does that look nicer, but we get constant performance no matter how many entries there are in the phone book. Hooray!

### Options

What should happen if we query our phone book for someone who's not in it? There are several ways we could handle this. It could return an error message of some sort. Or maybe it would just give a default number. Let's try it out and see what happens. Add a test for this scenario:

```
"Querying the phone book for someone who isn't in it " should "do something" in {
  val phoneBook = new PhoneBook
  phoneBook.lookup("Unknown person")
}
```

Run the test, and see what happens. You'll get some scary-looking error messages in the terminal, like this:

```
Exception in thread "main" java.util.NoSuchElementException: key not found: "Unknown person"
  at scala.collection.immutable.Map$Map3.apply(Map.scala:170)
  at ....
  ....
```

What's happened here is that the program has tried to look up a key in the Map, can't find it, so has no idea what to do. Because there's no obvious default way of dealing with this situation, it throws its hands up in the air and the program crashes. This is a common pattern for when you ask for values that can't be found, such as when you try to access the 10th element of a List that only has 9 elements. And it's extremely rare that having the program crash is actually the behaviour you'd want. Scala has a nice data structure called an **Option**, that models values that may or may not be there. Using Options forces you to consider what should happen if a value isn't present.

You can think of an Option as a box, that either contains a value, or doesn't. We have to specify the type of value that the Option can contain, in a similar way to how we specify the type of values that a List or Map contain. For instance, an Option that could potentially contain an Int would be `Option[Int]`. Both the Map and the List data structures have a method called `get`, that will return an Option instead of a value directly. Unless you're absolutely sure that the key is present in your Map, or the index is available in your List, you should use this method to make sure your program doesn't crash. Let's go ahead and change the `lookup` method in our `PhoneBook` class to use the `get` method:

```
def lookup(query: String): Int = entries.get(query)
```

That's not going to work yet. We've declared that the `lookup` method returns an `Int`, but `entries.get(query)` returns an `Option[Int]`. So we need to do something to convert the `Option[Int]` into an `Int`. This is where we're forced to decide what to do if the Option has no value. It's easy if it contains an Int - we just return the Int. If it doesn't contain a value I'm going to return a default value of 0. I'll show you three ways of doing that.

Firstly, we can use the `isDefined` method on the Option, which returns `true` if the Option contains a value, and `false` if it doesn't. Then once we know that the Option contains a value, we can use the `get` method on it to get the value out of it:

```
def lookup(query: String): Int = {
  val optionalResult: Option[Int] = entries.get(query)

  if (optionalResult.isDefined) optionalResult.get
  else 0
}
```

Another way of doing this is to use the `getOrElse` method on the Option. You pass a default value into the method. The method will return the value in the Option if it contains one, otherwise it will return your default.

```
def lookup(query: String): Int = {
  val optionalResult: Option[Int] = entries.get(query)

  optionalResult.getOrElse(0)
}
```

The third way is to use pattern matching on the Option.

### Pattern matching

Pattern matching is like a more powerful way of using `if` and `if else` statements. Here are two simple code snippets, each doing the same thing:

```
val x = 3

val ifResult = {
  if (x == 1) "a"
  else if (x == 2) "b"
  else if (x == 3) "c"
}

// ifResult is "c"

val matchResult = x match {
  case 1 => "a"
  case 2 => "b"
  case 3 => "c"
}

// matchResult = "c"
```

To create a pattern match, put the word `match` after the variable you want to compare things to, then within curly braces is the body of the match expression. The body consists of a series of *case* statements, which start with the word `case` followed by a value to compare with the match variable. If the match variable and the case value are the same, then the expression on the right hand side of the `=>` is evaluated. In this example the expressions on the right of the `=>` signs are just strings, so when the value of `x`, which is 3, matches the case statement `case 3`, the whole match expression returns the string "c", which gets assigned to the `matchResult` variable. The right hand sides of the case statements can be more complicated than just returning a value, and if they go over several lines then wrap them in curly braces, like so:

```
// You can match directly on a value, rather than assigning it to a variable first
val result = "Hello" match {
  case "Hello" => {
    println("Matched Hello")
    1 + 2
  }
  case "Goodbye" => 4
}

// result gets assigned the value of 3
```

The pattern matching syntax can look a bit neater than loads of `if else` clauses, but the real power comes when you combine it with case classes. I said earlier that Scala adds some cool functionality to case classes, and this includes being able to use them in pattern matching. 

I'll write an example using Monsters, Wizards and Players, before showing you how this relates to pattern matching Options. In our game, Wizards should be able to use their magic powers to scry on other characters. If they scry on a Monster, it should print out the Monster's name and health. The Player is magically shielded though, so if the Wizard scrys on it then it goes unnoticed. I'm going to create a `Scryable` trait that the Player and Monsters can extend, and the Wizard will have a `scry` method that takes a single method parameter with a type of `Scryable`. Remember how back in chapter 6 we had a class for Monsters, as we wanted to be able to create many Monsters for the game, but an object for the Player, as there would only ever be a single Player? Well we're going to use a case class for our Monsters now, and a case object for our Player.

```
trait Scryable

case class Monster(name: String, health: Int) extends Scryable
case object Player extends Scryable
```

You can put all of that in the same file if you like, or else create separate files for the trait, Monster and Player. I'm just going to stick everything in the same file for this little example. Now let's create our Wizard.

```
trait Scryable

case class Monster(name: String, health: Int) extends Scryable
case object Player extends Scryable

class Wizard {
  def scry(enemy: Scryable) = {
    // Now we have to work out if the enemy is a Monster or the Player.
    // If it's a Monster, print out the name and health.
    // If it's the Player, print out "Nothing to see here..."
  }
}
```

This is how we're going to call the method:

```
object Program extends App {

  val wiz = new Wizard()
  val monster = Monster("Barry", 20)

  wiz.scry(monster) // Should print "Argh, you found Barry! I've got 20 health"
  wiz.scry(Player)  // Should print "Nothing to see here..."
}
```

Now all we need to do is fill in the `scy` method. You know we're going to use pattern matching. You'll also see a nice feature of pattern matching with case classes, whereby we can deconstruct the case class attributes into variables in one go. Let's see it in action:

```
class Wizard {
  def scry(enemy: Scryable) = enemy match {
    case Monster(n, h) => println(s"Argh, you found $n! I've got $h health")
    case Player => println("Nothing to see here...")
  }
}
```

We're matching against the actual underlying type of the `Scryable` that's being passed into the method. If it's the Monster, we assign the Monster's attributes to the variables `n` and `h`. They get assigned in the same order that they were written when we defined the case class (`case class Monster(name: String, health: Int)`), so the value of the `name` attribute gets assigned to the `n` variable, and the value of the `health` attribute gets assigned to the `h` variable. We then use those variables with string interpolation on the right hand side of the case statement. Have a go at running the program now, and the correct lines should be printed out.

Not only can we deconstruct case classes into their component attributes using pattern matching, but we can also match against specific attributes. Let's say there's a Monster called Jimmy, who the Wizard is friends with. If the Wizard scrys on the Monster with the name Jimmy, it should print out a friendly message instead. Here's how we could make that happen:

```
class Wizard {
  def scry(enemy: Scryable) = enemy match {
    case Monster("Jimmy", _) => println("Alright matey, how's it going?")
    case Monster(n, h) => println(s"Argh, you found $n! I've got $h health")
    case Player => println("Nothing to see here...")
  }
}
```
```
object Program extends App {

  val wiz = new Wizard()
  val monster = Monster("Barry", 20)
  val friend = Monster("Jimmy", 20)

  wiz.scry(monster) // Should print "Argh, you found Barry! I've got 20 health"
  wiz.scry(Player)  // Should print "Nothing to see here..."
  wiz.scry(friend)  // Should print "Alright matey, how's it going?"
}
```

I used an underscore in `case Monster("Jimmy", _)`. This just means, "don't bother assigning this attribute to any variable. I'm not going to use it".  If you wanted to use Jimmy's health you could just name the variable `case Monster("Jimmy", jimmysHealth)`. Also, it's important to put the Jimmy case before the general Monster case. The computer goes through each case statement in order, and stops at the first one that matches.

All this is extremely similar to the way we can pattern match on Options. `Option` is like a trait. It has two concrete implementations that extend the trait. An Option that contains a value is represented by a case class called `Some`, which has a single attribute holding the value. An Option that doesn't contain a value is represented by the case object `None`. There are some more advanced features you need to know to be able to understand the real implementation of Options, so the following is pseudocode. If you're interested in seeing the actual implementation, just type 'Option' in IntelliJ and **Ctrl + Click** on it to go to the source file. 

```
trait Option

case class Some(value: Int) extends Option
case object None extends Option
```

Here's an example of using pattern matching on the Option we get from our Map:

```
class PhoneBook {

  val entries: Map[String, Int] = Map(
    "Barry" -> 4637,
    "Jenny" -> 43256,
    "Rover" -> 986,
    "Ian" -> 13452,
    "Spock" -> 76438
  )

  def lookup(query: String): Int = entries.get(query) match {
    case Some(986) => {
      // For some reason we want to do something weird with this entry
      println(s"You searched for $query. Adding 2 to the number")
      986 + 2
    }
    case Some(number) => number // Just return the number
    case None => 0 // The key wasn't found, so return a default value of 0
  }
}
```

Congratulations! You've learnt a tonne of things in this chapter. We're going to build on everything we've learnt in the next chapter by creating a Top Trumps style card game - plus we'll get to actually take some user input into our programs!