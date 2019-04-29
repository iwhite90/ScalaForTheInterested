# Chapter 10
## Input

This chapter is going to be fun! We're finally going to build a program that actually does something interesting. We'll be using a lot of what we've covered before, as well as learning how to take input into a program, both from files and from users. So let's get on, and make a game!

### The game

We're going to make a Top Trumps style card game. Unfortunately fancy graphics are beyond the scope of this book (although you'll be all set to learn how to use proper game engines after finishing it), so this will be a command line program that will be text based. It will read in a file containing the details of all the cards, and create a representation of the deck of cards in memory. It will shuffle the cards and deal half to the player and half to the computer. Then it will draw the top card from the player's hand and print out the details of the card on screen. The player will then be able to choose an attribute from the card to compare with the computer's top card. Whoever loses the round has their card removed, then the hands are shuffled and the next round is played. This goes on until one player loses all of their cards, at which point the game will print out whether the player won or lost.

Start by creating a new project. You can call it whatever you like. I'm going to call mine `TrumpsGame`.

### The model

We're going to use case classes to model the Abstract Data Types in our game. The most obvious thing we need to model for our card game is a **Card**. Our cards will have a character name, and values for the character's strength, intelligence and courage. An example might be:

> Name: Captain Kirk
>
> Strength: 8
>
> Intelligence: 7
>
> Courage: 9

This is really easy to model as a case class. Go ahead and create a new Scala Class in the `src/main/scala` folder called `Card`.

```
case class Card(name: String, strength: Int, intelligence: Int, courage: Int)
```
We'll create some more case classes later, but it's not super clear what we'll need yet, so that's enough for now.

### Reading files

So far all of our programs have done exactly the same thing when we run them. There's been no way to change what they do by passing in external input. Well, all that's about to change. We're going to create a *comma-separated-values* file (csv) that will contain the details of all the cards for the game. This means that we can easily change the cards by changing a file, rather than having to change code. CSV files are text fiels with a specific structure. Each line in the file has a specific number of values, separated by commas. You can think of them as a spreadsheet written out as a text file, and in fact spreadsheet programs let you export spreadsheets as CSV files. So once you've learnt how to do this, you'll be able to read spreadsheets into your programs by converting them to CSVs and following this technique.

A common place to put resource files for programs is in a *resources* folder in your project. Create a folder called `resources` at the same level as the `src` folder in your project (`TrumpsGame/resources`), and create a text file in the folder called `data.csv`. Copy this into your `data.csv` file:

```
Name, Strength, Intelligence, Courage
Kirk, 8, 6, 9
Picard, 6, 9, 9
Spock, 9, 9, 7
Data, 10, 10, 8
```

The first line is the header of the file, describing what each of the columns are. Column one is 'Name', column two is 'Strength', etc. Then each line after that describes one of the cards. To get the cards ready to play with we need to do several things:

1. Read the file into the program
2. Ignore the first line of the file
3. Go through each line of the file, creating a `Card` object from each one

This will give us a List of Cards. Then we need to:

4. Shuffle the cards
5. Deal the cards into two hands, one for the player and one for the computer

I'm going to start by writing a test. All of this logic is related, so we're going to put it in a single class called `CardLoader`. Create this class, and also a `CardLoaderSpec` test class. I'm not going to start by loading the file, but I know that we can convert a file into a List of strings, where each string is a line in the file, so I want to test that we can convert a List of file lines into a List of `Card`s:

```
class CardLoaderSpec extends FlatSpec with Matchers {

  val cardLoader = new CardLoader

  "createCardsFromLines" should "ignore the header line, and create cards from the rest of the lines" in {
    val lines = List(
      "Name, Strength, Intelligence, Courage",
      "One, 1, 2, 3",
      "Two, 4, 5, 6")

    cardLoader.createCardsFromLines(lines) shouldBe List(Card("One", 1, 2, 3), Card("Two", 4, 5, 6))
  }
}
```

Hopefully that makes sense. I've made the assumption that we're going to be able to get a `List[String]` from the file, and so I'm testing that we can pass that List into a `createCardsFromLines` method and get back a `List[Card]`. Next thing to do is to write the `createCardsFromLines` method:

```
class CardLoader {

  def createCardsFromLines(lines: List[String]): List[Card] = {
    val linesWithoutHeader = lines.drop(1)

    linesWithoutHeader.map { line =>
      val values = line.split(",")
      val trimmedValues = values.map(value => value.trim)
      
      Card(trimmedValues(0), trimmedValues(1).toInt, trimmedValues(2).toInt, trimmedValues(3).toInt)
    }
  }
}
```

The `drop` method is part of the List class. It takes an integer and returns a new List with the specified number of lines removed from the start. Here we're dropping 1 line, so `linesWithoutHeader` is a List without the header line.

Next we're mapping over the List of strings. Each element in the List (i.e. a string) is assigned to the `line` variable. The `split` method is part of the String class, and lets us split up a string into a List of strings by cutting the string wherever it finds a delimiter character. In this instance we're cutting the strings every time we find a comma. Using this on the string `"Kirk, 1, 2, 3"` returns a `List[String]("Kirk", " 1", " 2", " 3")`. Note that there's a space before each of the numbers. That's because there's a space after each of the commas in the original string. So we then map over each of these Lists and call the `trim` method on the strings. This just removes any spaces from the beginnings and ends of the strings. Finally our `map` converts the original line into a Card by instantiating a `Card` case class with the values from the List of trimmed strings. Because we're only working with strings at the moment, we need to tell the program that some of those strings are actually integers, by calling the `toInt` method on them.

We should have a passing test. Make sure you understand what's going on here before you move on.

What do we do with a List of Cards? We need to shuffle them and deal out the hands. Each of the hands can just be a `List[Card]`, but I'd like to be able to pass the player cards and the computer cards around together, so I'm going to model a `Deck` class. Again, this is an ADT, so we'll use a case class like this:

```
case class Deck(playerCards: List[Card], computerCards: List[Card])
```

We need to be able to split a List of Cards into a Deck. There's a bit of logic worth testing here, to make sure that the player and the computer get the same number of cards, so we're going to create a test for a new method that will take a List of Cards and return a Deck. Add this to the `CardLoaderSpec` class:

```
"splitCards" should "evenly split an even number of cards" in {
  val cards = List(Card("One", 1, 1, 1), Card("Two", 1, 1, 1), Card("Three", 1, 1, 1), Card("Four", 1, 1, 1))
  val deck = cardLoader.splitCards(cards)

  deck.playerCards shouldBe List(Card("One", 1, 1, 1), Card("Two", 1, 1, 1))
  deck.computerCards shouldBe List(Card("Three", 1, 1, 1), Card("Four", 1, 1, 1))
 }
```

You should be able to work out what's going on in this test. Here's the implementation of the method, in the CardLoader class:

```
def splitCards(cards: List[Card]): Deck = {
  val start = 0
  val mid = cards.length / 2
  val end = cards.length
  val playerCards = cards.slice(start, mid)
  val computerCards = cards.slice(mid, end)
  Deck(playerCards, computerCards)
}
```

Look! We're using another method built into the List class, `slice`, to create the player's hand from the first half of the List, and the computer's hand from the other half. Notice that we're only testing that we can split an even number of cards into two evenly sized hands. We're not sure exactly what would happen if we start with an odd number of cards. If you like you can decide what you'd like to happen if we get an odd number of cards, write a test for it, then update the implementation to make the test pass. For instance, you could say that either the player or the computer gets the extra card. Or maybe one of the cards isn't used for the game. If you don't want to do this, then just be careful to make sure that your data file contains an even number of cards.

Those are all the tests we're going to write for this class. We also need to shuffle the cards, but it's hard to write a test for this that will always pass. We could test that the shuffled cards are in a different order to the original cards, but shuffling is a random process, and as such it's possible that sometimes the cards will be shuffled and the order will not change, which would mean our test would fail. So the final part of our `CardLoader` class will be a method that the Game will call to load the cards, shuffle them, and return a `Deck`:

```
import scala.util.Random

class CardLoader {

  def loadCards(): Deck = {
    val lines: List[String] = ??? // Read the data file into a List of strings
    val cards = createCardsFromLines(lines)
    val shuffledCards = Random.shuffle(cards)

    splitCards(shuffledCards)
  }

  // Other methods omitted
}
```

We're using a handy class from the `scala.util` package called `Random`. It has a method called `shuffle` which takes a List and returns a new List with the elements shuffled. Nice! Follow the code through and check that you understand it. We haven't implemented the part that reads the file yet, but assuming that is going to work, we are then converting our List of file lines to a List of Cards, shuffling those cards, then splitting them into a Deck.

Actually reading a file and converting the lines of the file into a List of strings is actually very easy. We need to import another class from the scala library, called `Source`. This has a method called `fromFile` which takes a string with the location and name of the file to read. Here's how we use it:

```
import scala.io.Source

val lines: List[String] = Source.fromFile("resources/data.csv").getLines.toList
```

There are some intermediate steps you don't really need to know about, but if you're interested, the `fromFile` method returns an object of type `BufferedSource`. The `BufferedSource` type has a method called `getLines` which returns an object of type `Iterator[String]`. And the `Iterator[String]` type has a method called `toList`, which returns the `List[String]`.

Rather than hard coding the file name in this class, I'd like to make the class as generic as possible, so we're going to let the caller of the method pass in the name of the file as a method parameter. I've changed the name of the method from `loadCards()` to `loadCardsFrom(file: String)` to make this clearer. The complete class looks like this:

```
import scala.io.Source
import scala.util.Random

class CardLoader {

  def loadCardsFrom(file: String): Deck = {
    val lines: List[String] = Source.fromFile(file).getLines.toList
    val cards = createCardsFromLines(lines)
    val shuffledCards = Random.shuffle(cards)

    splitCards(shuffledCards)
  }

  def createCardsFromLines(lines: List[String]): List[Card] = {
    val linesWithoutHeader = lines.drop(1)

    linesWithoutHeader.map { line =>
      val values = line.split(",")
      val trimmedValues = values.map(value => value.trim)

      Card(trimmedValues(0), trimmedValues(1).toInt, trimmedValues(2).toInt, trimmedValues(3).toInt)
    }
  }

  def splitCards(cards: List[Card]): Deck = {
    val start = 0
    val mid = cards.length / 2
    val end = cards.length
    val playerCards = cards.slice(start, mid)
    val computerCards = cards.slice(mid, end)
    Deck(playerCards, computerCards)
  }
}
```

### Getting things going

Ok, we've modelled our cards and deck, and have a nice tested class that lets us load cards from a file. I think we're ready to write the entry point to our program and see something working. To start with, we'll just use the `CardLoader` to get us a `Deck`, print out a message welcoming the player to the game, and then describing the player's first card to them. We need a new file called `Game.scala` in the `src/main/scala' folder, in which we'll have a `Game` object extending the `App` trait:

```
objectGame extends App {

}
```

As you know, this is where the program will start running from. We need to add the functionality described above:

```
object Game extends App {

  val cardLoader = new CardLoader

  println("Let's play!")

  val deck = cardLoader.loadCardsFrom("resources/data.csv")

  val card = deck.playerCards.head
  println(s"You've drawn ${card.name}")
}
```

Run the program a few times. You should get different cards, because the deck is being shuffled.

### It's completely logical

Now we're going to get into the really interesting part of the program. Implementing the game logic. This is where we need to keep playing rounds until one player runs out of cards. We'll need the ability for the player to select an attibute to play, then the ability to compare the value of the player's attribute with the same attribute on the computer's card. The logic should be able to tell whether the player or the computer won the round, and discard the losing card from the relevant hand. It will then need to see whether the game is over, and if it is then display either a winning or a losing message. If the game isn't over then it will need to shuffle both players' cards and play another round.

This sounds like it could be pretty complex, so I'm going to neatly wrap up this functionality in its own class. We'll call this the `GameLogic` class. I want a recursive method that the `Game` object can call to start the first round. It will then keep on calling itself each round until the game is over. Once the game is over, it will return some kind of status to the `Game` object, so that it can decide how to notify the player whether they won or lost. A nice way to model statuses like this is to use case objects. They don't need to be case classes, as there will just be a single instance to represent each status. And we can use them in a nice way for pattern matching. We'll have a `Status` trait which the case objects `Win` and `Lose` will extend. Put this in a new file called `Status.scala`:

```
sealed trait Status

case object Win extends Status
case object Lose extends Status
```

Saying this is a *sealed* trait means that anything extending the trait has to be in the same file. Normally you can extend traits in different files to where the trait is written (just like our `Game` object isn't in the same file as the `App` trait). Sealed traits are useful for when you are going to use them in pattern matching, as the computer then knows all the possible cases that could be matched and will warn you if you forget to cover a case.

We now need a `GameLogic` class:

```
class GameLogic {

}
```

and a test file for it:

```
import org.scalatest._

class GameLogicSpec extends FlatSpec with Matchers {

}
```

We'll start by writing two staightforward tests for our `play` method. The method will take the player's cards and the computer's cards as parameters. We only want the method to return a value once the game is over, at which point we want to know the Status as well as how many cards the winner has left, so we can print out a message like, "Hooray! You won with 2 cards left!" Let's create another case class to represent these two things so we can pass them around together:

```
case class GameResult(status: Status, cardsLeft: Int)
```

So the definition of our `play` method should look like this:

```
def play(playerCards: List[Card], computerCards: List[Card]): GameResult = ???
```

The tests for this method are that it returns the right `GameResult` depending on whether the player or the computer has won. What does it look like when one of the players has won? It's when the other player has no cards left. So we know that if we call the method and one of the parameters is an empty list, then the game is over. Here are the tests:

```
import org.scalatest._

class GameLogicSpec extends FlatSpec with Matchers {

  val gameLogic = new GameLogic

  "If the computer runs out of cards it" should "return Win and the number of cards the player has left" in {
    gameLogic.play(List(Card("Player", 1, 1, 1)), List[Card]()) shouldBe GameResult(Win, 1)
  }

  "If the player runs out of cards it" should "return Lose and the number of cards the computer has left" in {
    gameLogic.play(List[Card](), List(Card("Player", 1, 1, 1))) shouldBe GameResult(Lose, 1)
  }
}
```

Try writing the implementation of the method to make these tests pass.

Here's my solution:

```
class GameLogic(playerInput: PlayerInput) {

  def play(playerCards: List[Card], computerCards: List[Card]): GameResult = {
    if (computerCards.isEmpty) GameResult(Win, playerCards.length)
    else if (playerCards.isEmpty) GameResult(Lose, computerCards.length)
    else GameResult(Win, 2) // Remove this line in a minute
  }
}
```

We need that last `else` just to make the program compile. We've declared that the method will return a `GameResult`, so if we didn't have that final statement and both the player and the computer had cards it wouldn't return anything. We'll remove it in a minute.

We can actually add the call to the `play` method from our `Game` object now, then deal with the end game conditions by pattern matching on the `GameResult`:

```
object Game extends App {

  val cardLoader = new CardLoader
  val gameLogic = new GameLogic

  println("Let's play!")

  val deck = cardLoader.loadCardsFrom("resources/data.csv")

  gameLogic.play(deck.playerCards, deck.computerCards) match {
    case GameResult(Win, cardsLeft) => println(s"You won, with $cardsLeft cards left!")
    case GameResult(Lose, cardsLeft) => println(s"Sorry, you lost. The computer has $cardsLeft cards left")
    case _ => println("Whoops, that was unexpected!")
  }
}
```

The `case _` is the default matcher that will get executed if the other cases don't match. In this instance we know that one of the other two cases will match (the player will either win or lose), so we add this just to keep the compiler happy.

If you run the program now, the player should win with 2 cards left. Can you see why?

Now is the time to replace that `else GameResult(Win, 2)` statement with what we actually want to happen if both players have cards. This is what should actually happen:

> Print out the details of the player's top card

> Ask the player to select an attribute to play

> Compare the value of that attribute with the equivalent attribute on the computer's top card

> Decide who won the round

> If the player won, then discard the computer's top card

> If the computer won, then discard the player's top card

> If it's a draw, then keep all the cards

> Shuffle both player's cards, and call the play method again with them

Ah, there's a condition in which the players can draw! This is easy to handle. Let's just add a new case object that extends `Status`:

```
sealed trait Status

case object Win extends Status
case object Lose extends Status
case object Draw extends Status
```

We're going to have some logic for comparing the player and computer attributes to work out who won the round, so I think we'll have a method for this. Let's call it `compareAttributes`. It can go in the `GameLogic` class, and here are the tests for it in the `GameLogicSpec` class:

```
"If the player attribute is greater than the computer attribute it" should "return Win" in {
    gameLogic.compareAttributes(2, 1) shouldBe Win
  }

  "If the player attribute is less than the computer attribute it" should "return Lose" in {
    gameLogic.compareAttributes(1, 2) shouldBe Lose
  }

  "If the player attribute is the same as the computer attribute it" should "return Draw" in {
    gameLogic.compareAttributes(2, 2) shouldBe Draw
  }
```

Again, have a go at implementing this method to make the test pass before reading on.

This is my approach:

```
class GameLogic {

  def compareAttributes(playerAttribute: Int, computerAttribute: Int): Status = {
    if (playerAttribute > computerAttribute) Win
    else if (playerAttribute < computerAttribute) Lose
    else Draw
  }

  // Other methods omitted
}
```

I'm going to replace the dodgy `else` statement with a method called `getPlayerChoiceAndCompareCards`. It will take the player's cards and the computer's cards as parameters, will deal with displaying the card details to the player, getting the player input, and comparing the card attributes. Then it will return the Status of the round. In fact, knowing what the inputs to and output from the method will be, we can finish writing our `play` method:

```
def play(playerCards: List[Card], computerCards: List[Card]): GameResult = {
    println("-------------") // A simple visual way to separate the rounds

    if (computerCards.isEmpty) GameResult(Win, playerCards.length)
    else if (playerCards.isEmpty) GameResult(Lose, computerCards.length)
    else {
      getPlayerChoiceAndCompareCards(playerCards, computerCards) match {
        case Win => play(Random.shuffle(playerCards), Random.shuffle(computerCards.tail))
        case Lose => play(Random.shuffle(playerCards.tail), Random.shuffle(computerCards))
        case Draw => play(Random.shuffle(playerCards), Random.shuffle(computerCards))
      }
    }
  }
```

So now we can see how the recursive loop of the game works. Given both players have cards, we call the `getPlayerChoiceAndCompareCards` method. This will let us know who won the round. It will then start the next round by shuffling the cards, with the losing card removed if it wasn't a draw, and calling the `play` method again. This will go on, round and round, until one of the end conditions is true (one of the players has no cards left), at which point a `GameResult` is returned to the `Game` object. The `GameResult` is used in a pattern match in the `Game` object, and the winning or losing message is printed out.

All we need to do now is fill in the code for the `getPlayerChoiceAndCompareCards` method. We're going to import the import the `scala.io.StdIn` class so we can use the `readInt` method. This waits for the user to enter a number in the terminal and press **Enter**, then reads that number into the program as an integer. Be careful though - if it can't turn the input into an integer (such as the string "Hello") then the program will crash. I'm also going to create another case class to hold the values of the attributes we want to compare, just for convenience sake. It looks like this:

```
case class AttributeValues(player: Int, computer: Int)
```

Here's the whole `GameLogic` class with the `getPlayerChoiceAndCompareCards` method implemented:

```
import scala.util.Random
import scala.io.StdIn._

class GameLogic(playerInput: PlayerInput) {

  def play(playerCards: List[Card], computerCards: List[Card]): GameResult = {
    println("-------------")
    if (computerCards.isEmpty) GameResult(Win, playerCards.length)
    else if (playerCards.isEmpty) GameResult(Lose, computerCards.length)
    else {
      getPlayerChoiceAndCompareCards(playerCards, computerCards) match {
        case Win => play(Random.shuffle(playerCards), Random.shuffle(computerCards.tail))
        case Lose => play(Random.shuffle(playerCards.tail), Random.shuffle(computerCards))
        case Draw => play(Random.shuffle(playerCards), Random.shuffle(computerCards))
      }
    }
  }

  def getPlayerChoiceAndCompareCards(playerCards: List[Card], computerCards: List[Card]): Status = {
    val playerCard = playerCards.head
    val computerCard = computerCards.head
    
    println(s"You've drawn ${card.name}")
    println(s"Enter 1 to play Strength ${card.strength}")
    println(s"Enter 2 to play Intelligence ${card.intelligence}")
    println(s"Enter 3 to play Courage ${card.courage}")
    print("> ")

    val choice = readInt()

    val attributeValues = choice match {
      case 1 => AttributeValues(playerCard.strength, computerCard.strength)
      case 2 => AttributeValues(playerCard.intelligence, computerCard.intelligence)
      case 3 => AttributeValues(playerCard.courage, computerCard.courage)
    }

    val attribute = choice match {
      case 1 => "Strength"
      case 2 => "Intelligence"
      case 3 => "Courage"
    }

    val status = compareAttributes(attributeValues.player, attributeValues.computer)
    status match {
      case Win => println(s"You beat ${computerCard.name}, who had $attribute of ${attributeValues.computer}")
      case Lose => println(s"${computerCard.name} beat you with $attribute of ${attributeValues.computer}")
      case Draw => println(s"It's a draw. ${computerCard.name} had the same $attribute as you.")
    }
    status
  }

  def compareAttributes(playerAttribute: Int, computerAttribute: Int): Status = {
    if (playerAttribute > computerAttribute) Win
    else if (playerAttribute < computerAttribute) Lose
    else Draw
  }
}
```

And there we go. You should have a fully functional game! Give it spin.

That `getPlayerChoiceAndCompareCards` method is a bit long for my liking. I'll leave it as an exercise for you to have a go at refactoring it into smaller methods, with each focussed on doing a single thing. After that have a play around with the program. Try making some changes, such as adding more attributes to the cards. Also see if you can find any problems with the game and fix them. For instance, what happens if a player enters a choice that's not 1, 2 or 3? You might have to make some decisions about what happens in unexpected cases. That's part of the fun of programming!