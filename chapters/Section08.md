# Chapter 8
## Lists and Arrays

In this chapter we're going to have our first look at a *data structure*. Data structures allow us to... well... structure related bits of data in useful ways. There are loads of common data structures, including: arrays; singly-linked lists; doubly-linked lists; sets; maps; queues; stacks; heaps; graphs etc etc. They structure collections of data in different ways, for different use cases.

For instance, a **queue** is a *first-in-first-out* data structure. You can think of it like a pipe into which you can push a series of objects. Then you can get objects back out of the pipe in the order in which you put them in. Some of these examples I'm going to write in pseudocode, which just means it looks like code, but won't actually run. It's used in books like this to illustrate concepts without getting bogged down in the details. In pseudocode, a queue would look something like this:

```
queue.push("a")
queue.push("b")
queue.push("c")
// queue now contains ("a", "b", "c")

val first = queue.getElement()
// first = "a", queue now contains ("b", "c")

val second = queue.getElement()
// second = "b", queue now contains ("c")

val third = queue.getElement()
// third = "c", queue is now empty
```

Whereas a **stack** is a *last-in-first-out* data structure. You can think of it like a stack you might make out of books. You can put new books on the top of the stack, and when you take a book from the stack you have to take it from the top. As a comparison with the queue, it would look like this:

```
stack.push("a")
stack.push("b")
stack.push("c")

// stack now contains ("a", "b", "c")

val first = stack.getElement()
// first = "c", stack now contains ("a", "b")

val second = stack.getElement()
// second = "b", stack now contains ("a")

val third = stack.getElement()
// third = "a", stack is now empty
```

To really get a good feel for data structures, it's useful to have an understanding of how programs actually store and access data. I'm going to use an analogy, in which we're going to pretend that our program is running inside a telephone exchange, and there are a hundred houses representing our computer memory. The houses each have a telephone number, number 1 being the first house, and number 100 being the last. Our program can ask the exchange to call up any house and ask it to either store some data or tell the program what data is currently stored in it.

*Disclaimer: computers don't contain tiny houses and telephone exchanges. This is delibrately simplified!*

What data can these houses store? They could be simple bits of data like strings and integers, or they could be more complex objects like milk cartons and monsters.

We can now see what's actually happening when we assign data to a variable. Let's step through what happens in this assignment:

```
val x = "Hello World!"
```

The program asks the telephone exchange for a house that's not currently being used. The exchange has a little book where it keeps track of which houses have data, and which don't. Let's say House 5 is currently empty. So the exchange tells the program, "you can use House 5".

"Great!", says the program. "Please could you phone up House 5 and tell it to store the string 'Hello World'".

"No problemo!", says the exchange. And so house 5 ends up storing our string.

The program has its own little book, where it keeps track of which houses are storing the values for each of its variables. So it updates its book to record that the value for variable `x` is stored in house 5.

Now the program comes across the following line of code:

```
println(x)
```

"Hmm", it says, scratching its chin. "Now where did I put the value for variable `x`?" It looks it up in the book, and sees that it's in House 5. So it asks the exchange to get the value from House 5.

"Yo!", says the exchange to House 5. "What data you got?"

"I'm holding onto the string 'Hello World', like you told me to", says House 5.

"Coolio, I'll just pass that along to the program", says the exchange.

"Thanks guys," says the program. Now I can print out "Hello World!"

And there you go. That's just how computer memory works!

Just to prove that I'm not making this up, let's write a little program. We'll need a class. It doesn't really matter what it is. You can use a `MilkCarton`, a `Monster` or write a new one. It doesn't need to have any attributes or methods. We're just going to use it to create some object instances of it. Assuming you've got something like:

```
class Monster {
}
```

create the main part of your program like so:

```
object Program extends App {
  println(new Monster)
  println(new Monster)
}
```

Run the program, and you should see something like the following in the terminal, although the random letters and numbers at the end will be different for you:

```
Monster@7dc7cbad
Monster@d2cc05a
```

Looks a bit strange doesn't it? Those weird letters and numbers are actually the phone numbers of the houses where the objects are stored. Or more accurately, they are the memory addresses of the memory registers where the objects are stored. Because we've created two different `Monster` objects, they have to be stored in two different locations in memory.

> If you're interested, those addresses are printed in *hexadecimal*. How computers work in binary, and different binary notations, are really very interesting, but not core to this book. There is also plenty of great material for free online about this subject, so if you're so inclined I'd encourage you to do some reading about binary and hexadecimal at some point.

We've seen that the `println` method definition in the `Predef` object is:

```
def println(x: Any): Unit
```

So you can pass anything into it to be printed to the terminal. But what does it mean to print out a `Monster` object, or a `MilkCarton`? The writers of the `println` method decided that the most sensible thing was to have a default implementation that just prints out the type of the object, followed by an `@`, then the object's memory address. This isn't really very useful, and not what you'd normally want, so they've provide a way to override this default. When `println` receives an object, it first looks to see whether that object has a method called `toString` defined on it. The method has to return a string, and that is what will be printed out. If the method isn't present, then the default is used. So we could get our `Monster` to print out something more useful like so:

```
class Monster {
  override def toString() = "I'm a Monster!"
}
```

Run the program again, and you'll get the following output:

```
I'm a Monster!
I'm a Monster!
```

And to make it even more useful, let's change it so we can distinguish between the two objects:

```
class Monster(name: String) {
  override def toString() = s"I'm a Monster called $name!"
}
```

Change your program to pass in different names to your `Monster`s in the constructor parameters, and run the progrm again.

That little exercise was a illustration of memory addresses, but now let's get back to data structures. We now know enough to be able to talk about two of the most basic, but also most commonly used data structures: arrays and lists.

### Arrays

An array is a sequence of memory addresses that are next to each other. Using our houses analogy, if we wanted an array that can hold 5 objects of pieces of data we'd ask the exchange to find a block of 5 empty houses with sequential numbers. It might find that houses 11, 12, 13, 14 and 15 are empty, so it sets them aside for our array, and it returns the number of the first house in the array: 11. Now the program just needs to store a single variable referencing the start of the array, rather than having to have a variable for every element in the array. You can see how this would be useful when you start having hundreds or thousands of elements in the array. It can then very quickly access any element of the array by asking the exchange to do some simple maths. Say it wants the first element, it will ask the exchange for the number of the first element (11). If it wants the second element, it will ask the exchange to add 1 to the number of the first element (11 + 1 = 12). If it wants the fifth element, it will ask the exchange to add 4 to the number of the first element (11 + 4 = 15). The number it asks to add on to the number of the first element is called the **index** of the array. Arrays are zero-indexed data structures, because to get the first element of the array you ask to add 0 to the number of the first element.

Let's do an example, modelling a shopping list with an array. The syntax for creating a new array is `new Array` followed by the data type that the array will contain within square brackets, followed by the size of the array within parentheses. I know I want to buy 5 items, so I'm going to have an array with 5 elements, each holding a string describing the items:

```
val shoppingList = new Array[String](5)
```

Now I want to add a string to each element of the array. We reference individual elements in the array by specifying the index of the element in parentheses after the name of the array. So we can assign a string to the first element in the array (i.e. at index 0) like this:

```
shoppingList(0) = "Apples"
```

And we can retrieve the value of the element like this:

```
val x = shoppingList(0)
println(x) // Prints "Apples"

// Or without having to use an extra variable to hold the data
println(shoppingList(0)) // Prints "Apples"
```

Let's write a little program to fill out all five elements of our shopping list, then print out the middle element:

```
object Program extends App {

  val shoppingList = new Array[String](5)

  shoppingList(0) = "Apples"
  shoppingList(1) = "Toothpaste"
  shoppingList(2) = "Potatoes"
  shoppingList(3) = "Newspaper"
  shoppingList(4) = "Bread"

  println(shoppingList(2)) // Prints "Potatoes"
}
```

A useful feature of arrays is that it's easy to change the data in specific elements. Let's say we don't want potatoes after all, but want to swap them for Easter eggs. We can just assign a new string to the third element of the array:

```
object Program extends App {

  // You can use this syntax for creating an Array and populating its elements at the same time
  val shoppingList = Array[String]("Apples", "Toothpaste", "Potatoes", "Newspaper", "Bread")

  println(shoppingList(2)) // Prints "Potatoes"

  shoppingList(2) = "Easter eggs"

  println(shoppingList(2))  // Prints "Easter eggs"
}
```

Adding, retrieving and changing data in arrays is really quick. Because the elements are all in a continuous block of memory, given the address of the first element in the array and the index of the element it's really simple for the program to add them together to find out the address of the element, and then can access that element directly. Our example of setting the third element to "Easter eggs" would involve the following steps:

> 1. Get address of the first element in the shoppingList array (say address 25)
> 2. Add index of 2 to this address to get the address of the third element (27)
> 3. Set data at memory address 27 to "Easter eggs"

That's only 3 steps. But how good would an array be for adding new elements somewhere in the middle. So instead of swapping potatoes for Easter eggs, I want to add Easter eggs. And I'm going to say that the array is in priority order, so if I don't have time to buy all the items I know that the most important ones are at the beginning of the list. And I *really* love Easter eggs, so much so that they come under apples in importance to me. In order to add Easter eggs between apples and toothpaste, the program first has to make sure that there is enough space in the array to take another element. If our array is already full, the computer will need to find another block of contiguous memory addresses that has space for six elements, and copy across the existing five elements first. It then needs to 'make space' for the new element at index 1 by shifting down all the following elements one at a time:

> 1. Get value from index 4 (Bread)
> 2. Put Bread in index 5
> 3. Get value from index 3 (Newspaper)
> 4. Put Newspaper in index 4
> 5. Get value from index 2 (Potatoes)
> 6. Put Potatoes in index 3
> 7. Get value from index 1 (Toothpaste)
> 8. Put toothpaste in index 2
> 9. Finally got space in index 1, so put Easter eggs in index 1

And it's even worse than it looks. For each of these 9 steps, it has to do the 3 steps described above to actually get or set the data in each element. So that's 27 steps to add an extra element at index 1. Phew!

Now I'm going to describe another data structure that also holds sequences of data, and lets you reference individual elements by index, but which would only take 3 steps to add an element at index 1.

### Lists

On the surface a list seems very similar to an array, and let's you do many of the same things with sequences of data. But the implementation of a list is very different, which gives it different performance characteristics to an array. Whereas an array needs to find a contiguous block of memory addresses to hold its elements, a list can store them all over the place. Using our houses analogy, the first element of the list could be in House 4, the second in House 10, the third in House 2 etc. So if the list elements are spread out all over the place, how does the program know the addresses for the elements? In a similar way to an array, the program just needs to keep track of the address of the first element in the list. That's because each element in the list contains not only the data we want it to hold, but also the memory address of the next element. In pseudocode, we could write out the implementation of a list using two classes, like this:

```
class ListElement {
  val data = ???
  val addressOfNextElement = ???
}
```

```
class List {
  val addressOfFirstElement = ???
}
```

Now if we want to get access to the third element in the list, the computer can take the following steps:

> 1. Get the address of the first element from the List
> 2. Get the ListElement at that address (1st element)
> 3. Get the address of the next element from the ListElement
> 4. Get the ListElement at the next address (2nd element)
> 5. Get the address of the next element from the ListElement
> 6. Get the ListElement at the next address (3rd element)
> 7. Do whatever we want to do with the data in the 3rd ListELement

Whoa, that's a lot more steps to access an element than with an array. That's right, an array can consistently access any element in it in three steps, whereas the number of steps increases with increasing element index for a list. This is an example of a trade off, where some data structures can have worse performance than others for certain things, but then be better than them for other things. Let's show you the benefit lists have over arrays when inserting new elements.

Say we've got our shopping list of five items in a list rather than an array, and we want to add Easter eggs between apples and toothpaste. The steps would be like this:

> 1. Get the address of the first element from the List
> 2. Get the apples ListElement at that address
> 3. Create a new ListElement, with data = "Easter eggs", and set its addressOfNextElement to the address of the toothpaste ListElement (the addressOfNextElement from the apples ListElement we got in step 2). Apples and toothpaste will now both be pointing to Easter Eggs as their next element
> 4. Save the Easter eggs ListElement to memory, and take a note of its address
> 5. Set the apples ListElement to point to the Easter eggs ListElement (now apples will be pointing to Easter eggs, which will be pointing to toothpaste)

That was 5 steps for a list, versus 27 for an array. There's a lot more subtlety in the differences between arrays and lists, so deciding which one to choose for a particular use case can be a bit of an art. However, at this stage in your programming journey it's probably enough just to have an understanding that differences exist. Unless you're writing programs to crunch through massive amounts of data, you're unlikely to notice any difference in the performance of your programs.

Let's write our shopping list using a List instead:

```
object Program extends App {

  val shoppingList = List[String]("Apples", "Toothpaste", "Potatoes", "Newspaper", "Bread")

  println(shoppingList(3)) // Prints Newspaper
}
```

The syntax is pretty similar to Arrays, but note that you don't use the `new` keyword, and instead of passing in an integer to specify the size of the Array, we pass in all the elements of the List. To retrieve an element at a specific index, the syntax is the same as with an Array. There's a very important difference to note though: Arrays are mutable and Lists are immutable. That might sound a but funny, as we used the word `val` to declare each of our shopping lists:

```
val shoppingListArray = new Array[String](5)
val shoppingListList = List[String](...)
```

Doesn't that mean they're both immutable? In actual fact what we're saying is that the *variables* `shoppingListArray` and `shoppingListList` are immutable. We're not saying anything about the mutability of the actual Array or List themselves. Let me demonstrate with a quick example:

```
val immutable = new List[Integer](1, 2)

immutable = new List[Integer](3, 4) // Won't work. We can't reassign something to a var

var mutable = new List[Integer](1, 2)

mutable = new List[Integer](3, 4) // This is fine. mutable is now a List containing (3, 4)
```

Ok, that showed how the `val` and `var` keywords applied to the mutability of the variables. Now let's see the difference in mutability between Arrays and Lists:

```
val array = new Array[Integer](2) \\ Using a val, so variable array is immutable

// Elements in the array are mutable. We're able to change them, like this:
array(0) = 1
array(0) = 2

// But we can't assign a new Array to the array variable
array = new[Array](3) // Won't work

var list = List[Int](1, 2) // Using a var, so variable list is mutable

// The list itself is immutable, so we can't change it
list(0) = 3 // Won't work

// But we can assign a new List to the list variable
list = List[Int](3, 4)
```

If you think about it, it makes sense. An Array is just a sequence of memory address. Once the Array has been created, those addresses don't change. So the addresses are immutable. But the program can access and change the data held in those memory addresses, so the data is mutable. A List is more than a sequence of addresses. It is modelled as Scala classes (like `List` and `ListElement`). The designers of Scala made the decision that the attributes of these classes should be `val`s, so once a `ListElement` is created we can't change its data or the address of the next element. This may seem restrictive, but it's actually a good choice for the language. There are many benefits to keeping things immutable, and purely functional programs don't have mutability at all. However, different languages work differently, and if you pick up another language like Java you'll find that its Lists are mutable.

This makes for a bit of a conundrum. I said that one of the benefits of Lists was being able to insert new elements within them. And they won't be that useful if you can't potatoes to Easter eggs. But how can we do this if you're not allowed to alter the List? The answer is actually to take your List and use it to make a new one. There are some really simple ways to do this, which we'll come to soon, but I'm actually going to show you a *hard* way first. No, it's not because I'm sadistic! It's because you'll learn about a really important and fundamental way of dealing with immutable data structures.

### Recursion

Ooh, I got a tingle just typing that title! Most beginner's programming books wouldn't go near this subject with a barge pole, but I've got faith in you! The problem we're trying to solve is one of *iteration*, also called *loops*, or just *doing something repetitively*. Let's start with a really simple example. Say we want to print out "Hello World!" twice. We'd probably just write:

```
object Program extends App {
  println("Hello World!")
  println("Hello World!")
}
```

Seems ok. But what if we wanted to print it out a hundred times? Or a thousand? We definitely don't want to copy and paste that a thousand times! We're going to use a recursive method to get this working in less than 10 lines of code. The first thing to realise is that you can call methods from within other methods.

```
object Program extends App {

  def printName(name: String) = printHello(name) // Calling printHello method from within printName

  def printHello(x: String) = println(s"Hello $x")

  printName("Ian")
}
```

The trick with a recursive method is that it calls *itself*. And because there's a call to itself within its body, every time it calls itself it calls itself again. And again and again in a loop. The other thing most recursive methods have is a way of knowing when to stop, otherwise they'll just keep looping forever. I know it can be a bit difficult to get your head round this, and see if you can make sense of it before reading the description below.

```
object Program extends App {

  // You need to specify the return type for recursive methods.
  // Here the method doesn't return anything, so we specify a return type of Unit
  def loop(counter: Int): Unit = {
    if (counter > 0) {
	  println("Hello World!")
	  loop(counter - 1)
	}
  }

  loop(1000)
}
```

I find it easiest to understand methods like this by walking through the steps it takes in my head. To start with, we've got a method called `loop` that takes an integer as a method parameter. We've called the parameter `counter`, but it doesn't matter what it's called. The last line of the program is where we start everything off by actually calling our method. I don't want to step through 1000 loops in my head, so lets try and work out what happens if we call `loop(2)` instead.

> 1. The parameter 'counter' is assigned the value 2
> 2. The if statement checks whether 'counter' is greater than 0. 2 is greater than 0, so the body of the if statement is executed
> 3. We print out "Hello World!"
> 4. The method calls itself, passing in a value of counter minus 1. 2 minus 1 is 1, so it's calling loop(1)
> 5. The loop method is called passing in a value of 1, which is assigned to the 'counter' parameter
> 6. 1 is greater than 0, so the body of the if statement executes
> 7. We print out "Hello World!"
> 8. The method calls itself passing in a value of 0
> 9. The method is called with a value of 0, which is assigned to the `counter` parameter
> 10. 0 is not greater than 0, so the body of the if statement is not executed
> 11. There's no more code in the method body, so the method finishes

Hopefully that makes sense. If not, go back and walk it through in your mind a few times. The two crucial things to remember about recursive methods are:

> 1. They call themselves
> 2. They have some condition that tells them whether to call themselves again or to stop

If you've got that, great! It's definitely not a beginner's concept, so well done!

We're going to use recursion to do some stuff with Lists now. They are great for doing something with the first element of the List, then calling themselves to do the same thing with the second element, then calling themselves for the third element, etc. And we can tell them to stop when they get to the end of the List. The first things we're going to do is to take a List of integers, and use recursion to add up all the elements of the List.

> Don't feel that you need to remember all the syntax for working for data structures. If you forget whether it's 'new List()', 'List()[String]' or whatever, it's very easy to look it up. It's more important to understand the concepts.

We're going to use some of the attributes built into Lists to help us. The first element of the List is represented by the `head` attribute, and the rest of the List is represented by the `tail` attribute. The following example will demonstrate this:

```
object Program extends App {

  val wholeList = List[Int](1, 2, 3)
  val listHead = wholeList.head
  val listTail = wholeList.tail

  println(wholeList) // Prints List(1, 2, 3)
  println(listHead) // Prints 1
  println(listTail) // Prints List(2, 3)
}
```

And we can use the attributes `Boolean` attributes `isEmpty` and `nonEmpty` to check whether the List is empty or not, like so:

```
val emptyList = List[Int]()
println(emptyList.isEmpty) // Prints true
println(emptyList.nonEmpty) // Prints false

val listWithElements = List[Int](1)
println(listWithElements.isEmpty) // Prints false
println(listWithElements.nonEmpty) // Prints true
```

So now we've got all the tools we need to add up the elements of a List, using recursion to iterate through the elements, `head` and `tail` to deconstruct the List, and `isEmpty` or `nonEmpty` to create the condition in which the recursion ends. We haven't written a test in this chapter yet, so let's take a test-driven approach to this. I'm going to create an object called `ListOps` in which we can put methods to operate on Lists, so go ahead and create a `ListOps` object and a `ListOpsSpec` test class. Our first test will look like this:

```
class ListOpsSpec extends FlatSpec with Matchers {

  "sumList" should "return the sum of the elements in a List" in {
    val list = List[Int](1, 2, 3)
    ListOps.sumList(list) shouldBe 6
  }
}
```

```
object ListOps {

  def sumList(list: List[Int], total: Int = 0): Int = {
    if (list.nonEmpty) {
	  val firstElement = list.head
	  val restOfList = list.tail
	  val newTotal = total + firstElement
	  sumList(tail, newTotal)
	}
	else {
	  total
	}
  }
}
```

Try stepping through this code in your head to understand what's going on. There's nothing new here that you haven't covered already. When you're ready we'll go through it together.

> 1. We're calling `sumList` with a single parameter - a List containing (1, 2, 3). This gets assigned to the 'list' parameter
> 2. Because we're not passing in a second parameter, the `total` parameter takes the default value of 0
> 3. We check that the List is not empty. It isn't, so the body of the if statement gets executed
> 4. We deconstruct the List into two variables. `firstElement` is assigned the value 1, which is the head of the list. `restOfList` is assigned the tail of the List, which is a new List containing the elements (2, 3)
> 5. We add the first element to our running total, so `newTotal` is assigned the value 0 + 1
> 6. We call sumList, passing in the tail of the List and the `newTotal`. It's the equivalent of calling sumList(List[Int](2, 3), 1)
> 7. The List containing (2, 3) is assigned to the `list` parameter, and the value 1 is assigned to the `total` parameter
> 8. `list` isn't empty, so we go into the body of the if statement
> 9. We deconstruct the `list`. `firstElement` is assigned the value 2, `restOfList` is assigned a new List containing a single element (3), and `newTotal` is assigned 1 + 2 = 3
> 10. We call `sumList` again
> 11. `list` is assigned the List containing a single element (3), and `total` is assigned the value 3
> 12. `list` isn't empty, so `firstElement` is assigned the value 3, `restOfList` is assigned a new List with no elements, `newTotal` is assigned 3 + 3 = 6, and we call the method again
> 13. `list` is now empty, so `list.nonEmpty` is false. We don't exeucute the body of the if statement, but go to the else branch instead
> 14. The else branch just returns the value of the `total` parameter, which is 6

There's quite a lot going on there, but if you go through it line by line hopefully it makes some sense. I wrote it in quite a verbose way to make it easier to understand, but if you like you can do it in fewer lines of code by getting rid of the variable assignments within the if statement body:

```
def sumList(list: List[Int], total: Int = 0): Int = {
  if (list.nonEmpty) sum(list.tail, total + list.head)
  else total
}
```

Use whichever you prefer, but as you get used to it you'll probably end up using the more compact version.

See if you can write another recursive method to multiple the elements of a List. In oher words, write a recursive method to make this test pass:

```
"multiplyList" should "return the multiple of the elements of a List" in {
  val list = List[Int](1, 2, 3, 4)
  ListOps.multiplyList(list) shouldBe 24
}
```

If you run into problems with it not giving you the right answer, you might find it helpful to print out the values of variables as it iterates by using `println`. For instance, putting a call to `println(total)` in the method will print out the running total each time the method is called, and will help you to check if it's behaving as your expect.

If you need a hint, don't assign a default value of 0 to the `total` parameter. Anything multiplied by 0 is 0.

Let's finish the chapter by implementing the same thing for Arrays. Unlike Lists, Arrays don't have `head` and `tail` attributes. We can only access the elements of Arrays by referring to them by index. So we're going to have to take a different approach. Start by creating an `ArrayOps` object and an `ArrayOpsSpec` test class. The tests will be pretty much the same as the tests we wrote for the `ListOps`, but lets start with multiplication this time:

```
class ArrayOpsSpec extends FlatSpec with Matchers {

  "multiplyArray" should "return the multiple of the elements of an Array" in {
    val array = Array[Int](1, 2, 3, 4) // Using syntax to populate an Array on creation
	ArrayOps.multiplyArray(array) shouldBe 24
  }
}
```

You need to know one final thing before you can solve this. Arrays have an attribute called `length` which tells you how many elements they have.

```
val x = Array[Int](1, 2, 3, 4)
println(x.length) // Prints 4

// Arrays are zero indexed - the first element has index 0.
// So the last element has index of length - 1.
println(x(0)) // Prints 1
println(x(x.length - 1)) // Prints 4

// If you try to access an index that is outside the Array, you'll get an error
println(x(-1)) // Index too low, won't work
println(x(x.length)) // Index too high, won't work
```

If you feel up to it, try writing the implementation before reading on. 

Remember, you can't deconstruct the Array into a head and a tail, so you'll need another way to iteratively access the next element of the Array each time you call the recursive method.

You've seen how we can pass in a running total as a method parameter. How about passing in a parameter representing the index of the element we want to access as well?

My solution looks like this:

```
object ArrayOps {

  def multiplyArray(array: Array[Int], total: Int = 1, index: Int = 0): Int = {
    if (index < array.length) multiplyArray(array, total + array(index), index + 1)
	else total
  }
}
```

Finish up by writing a test and implementation for `ArrayOps.sumArray`, then we're done. Awesome job!