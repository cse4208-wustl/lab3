# Lab 3

## A New Game

This lab extends your work with procedural, functional, object-oriented, and generic programming styles in C++17. You will (1) implement the deck used for [Uno](https://www.unorules.org/), and (2) create a game of [Go Fish](https://bicyclecards.com/how-to-play/go-fish) that can use the Uno deck, or the Hold 'Em or Pinochle deck from the previous lab.

To do this, you will add new features to your `CardSet` class template so you can iterate through and manipulate its cards without directly exposing the underlying container. You will use these features to move cards between players' hands in your Go Fish game. Along the way you will gain experience with:

- iterators and STL algorithms such as `std::sort`, `std::move`, `std::copy_if`, `std::remove_if`, `std::find_if`, and `std::count_if`
- callable objects, `std::function`, and lambdas used as predicates
- class templates with default type parameters, template specialization, and `static_assert`
- smart pointers (`std::make_shared`) and exception handling

## Reference

Useful references for this lab:

- sequential containers: Lippman Chapter 9
- generic algorithms: Lippman Chapter 10 and Appendix A.2
- associative containers: Lippman Chapter 11
- templates: Lippman Chapter 16
- [C++ Reference](https://en.cppreference.com)

## Programming Guidelines

Review the [Programming Guidelines](docs/programming-guidelines.md) before you begin implementing the lab, and keep them in mind as you develop and test your solution.

## Assignment

Record your observations, design decisions, compile warnings or errors, and any other written responses in `ANSWERS.md` as you work.

Use good modularity as you develop your code: for example, if a function's logic has several separable parts, or is long enough to fill the editor's window, consider breaking it up into smaller functions.

Some details of this lab are intentionally somewhat under-specified. When that happens, choose a reasonable design and explain your decisions in comments in the code and in `ANSWERS.md`.

### Setup

1. Log into one of the Linux Lab machines via `qlogin`, and confirm that the correct version of `g++` (`8.3.0`) is installed in your environment there, as you did in [Studio 0](https://github.com/cse4208-wustl/studio0).

1. Clone your `lab3` repo and work inside that cloned directory.

1. Copy your `Makefile` and all `.cpp` and `.h` files from your previous lab assignment into this repo. You will use, and in some cases modify, them along with new code you write in this lab.

   If appropriate, rename files and update your `Makefile` accordingly, for example so that `lab2.cpp` becomes `lab3.cpp`.

### Refactoring `CardSet`

1. In your `CardSet` declaration and definition, replace the accessor that returns a pointer to the protected vector data member with two non-static accessor methods that return iterators positioned at the vector's `begin()` and `end()`, respectively.

   These iterators can be used to view the contents of the vector and can be passed to sorting and searching functions, for example to identify hand types in different games. Update the places in your code that used the pointer to the protected vector so that they use the iterators instead.

1. Add a `sort()` method to `CardSet` if you did not already do so in the previous lab, or revise it as needed if you did, so that it calls [`std::sort`](https://en.cppreference.com/w/cpp/algorithm/sort).

   Refactor `play()` in both `PinochleGame` and `HoldEmGame` to use the new iterators and the `sort()` wrapper method of CardSet (instead of using `vector<Card<R, S> >`, use `CardSet<R, S>`). Do the same for `holdem_hand_evaluation` and `suit_independent_evaluation`.

   **At this point, build and run your code and verify that it produces the same results as in Lab 2.**

1. Add type aliases for the card type and iterator type in the `public` portion of your `CardSet` class template declaration, for example:

   ```cpp
    using card_type = Card<R,S>;
    using const_iter = typename std::vector<card_type>::const_iterator;
   // ...
   ```
   Update your code to use these new type specifiers, where appropriate.
1. Move the `collect` method from the `Deck` class template up into the `CardSet` class template, and rewrite it to use [`std::move`](https://en.cppreference.com/w/cpp/algorithm/move) from the STL algorithms library.

   Consider using a `std::back_inserter`. Keep in mind that the container you move from is still non-empty after the call to `std::move`, so it must be cleared explicitly.

1. Add a method called `collect_if` to the `CardSet` class template. It should take a `CardSet` called `deck` and a predicate of type `std::function<bool(Card<S, R>&)>`. It collects only the cards that satisfy the predicate, and will be used later to move four-of-a-kinds in the Go Fish game.

   First, use [`std::copy_if`](https://en.cppreference.com/w/cpp/algorithm/copy) to copy every card in `deck` that matches the predicate into the `CardSet` on which `collect_if` was called.

   Then use [`std::remove_if`](https://en.cppreference.com/w/cpp/algorithm/remove) to remove those same cards from `deck`. Note that `std::remove_if` does not shrink the vector; it shifts the remaining values forward and leaves an unspecified region at the end:

   ```text
   [0 0 3 4 5 7 7 7 7 8 9] -> (remove 7s) -> [0 0 3 4 5 8 9 ? ? ? ?]
   ```

   `std::remove_if` returns an iterator to the start of that unspecified range. Pass that iterator and the vector's `end()` iterator to the vector's `erase` method to remove those entries.

1. Add a public method called `request` to the `CardSet` class template. It should take a reference to another `CardSet` and a rank, and return a `bool`.

   Use [`std::find_if`](https://en.cppreference.com/w/cpp/algorithm/find) to look for a card of the given rank in the `CardSet` that was passed in.

   If no such card is found, return `false`. Otherwise, move that card from the passed-in `CardSet` into the `CardSet` on which `request` was called, and return `true`.

   **At this point, build and run your code again and verify that it still produces the same results as in Lab 2.**

### The Uno Deck

1. Create a new C++ header and source file in which you will declare and define types related to an Uno deck.

   Uno has colors rather than suits. In the header file, outside of any class declaration, declare a `Color` enumeration (used in place of suits) with the labels `red`, `blue`, `green`, `yellow`, `black`, and `undefined`.

1. In the same header file, declare an `UnoRank` enumeration with the labels `zero`, `one`, `two`, `three`, `four`, `five`, `six`, `seven`, `eight`, `nine`, `skip`, `reverse`, `drawtwo`, `drawfour`, `wild`, `blank`, and `undefined`.

   In the header and source file, declare and define `operator<<` for `UnoRank` that prints the rank to a given `ostream`, and `operator++` that advances the rank to the next one. Declare and define the same operators for `Color`.

1. In that header and source file, declare and define a class called `UnoDeck` that is derived from `Deck` parameterized by `Color` and `UnoRank`.

   Its constructor should take no arguments and should populate the inherited vector of cards according to the [standard contents of an Uno deck](https://www.unorules.org/how-many-cards-in-uno/), for a total of 112 cards:

   - one `red`, one `blue`, one `green`, and one `yellow` card of rank `zero`
   - two `red`, two `blue`, two `green`, and two `yellow` cards of each rank `one` through `drawtwo`
   - four `black` cards of each rank `drawfour` through `blank`

### The Go Fish Game

1. Create a new C++ header and source file, and in them declare and define a class template called `GoFishGame` with three type parameters: the suit type, the rank type, and the deck type. These should default to `Suit`, `HoldEmRank`, and `HoldEmDeck`, respectively, since Go Fish can be played with many kinds of decks.

   Public members should include:

   - a constructor that takes two arguments: the number of players, and an array of C-style strings listing their names
   - a virtual destructor
   - a virtual implementation of `play()`

   Protected members should include:

   - the deck of cards, whose type is given by the deck type parameter
   - a vector of `CardSet` for the players' `hands`
   - a vector of `CardSet` for the `books` each player has made
   - a `virtual deal()` method

   Go Fish works with any deck that has at least four cards of each rank. The constructor should check that the deck has at least four cards of each rank and throw an exception if it does not. Consider using [`std::count_if`](https://en.cppreference.com/w/cpp/algorithm/count) to check which ranks have at least four cards.

   **You may need template specialization to provide appropriate definitions of the constructor for the different combinations of suit type, rank type, and deck type.**

1. In the `GoFishGame` class template header, add a `static_assert` that uses `std::is_base_of` to check that the deck type parameter is derived from a `CardSet` parameterized by the provided suit and rank types (for example, `CardSet<S, R>`).

1. Add a `collect_books` method to `GoFishGame` that takes a player number, checks whether that player's hand contains a four-of-a-kind, and returns a `bool` indicating whether one was found.

   The method should iterate over the `CardSet` representing the player's hand to find a rank with four cards. If there is none, do nothing and return `false`.

   If there is a four-of-a-kind, call the `collect_if` method you defined in `CardSet` to move all cards of that rank from the player's hand into the `CardSet` holding that player's books.

   `collect_if` requires a predicate. Create a function object of type `std::function<bool(Card<S, R>&)>` called `predicate`, and initialize it with a lambda that compares the rank of the passed card to the rank being removed. Pass this predicate, along with the player's hand, to `collect_if`.

   After moving the four-of-a-kind into the player's books, return `true`.

1. In `GoFishGame`, declare and define a protected `turn` method that implements a single player's turn. It should take a player number and return a `bool` indicating whether it is still that player's turn.

   The method should use `std::cout` and `std::cin` to print the players' hands and books, and then prompt the user for a card rank to ask for and a player to ask. Repeat the prompt until the user provides a valid rank (the rank of a card in the current player's hand) and a valid player number (a player in the game).

   **Make sure the output showing the players' hands and books is compact, so that all players' holdings are easy to see at once in a terminal window.**

   Then use the `request` method to try to obtain a card of that rank from the targeted player. If that succeeds:

   1. repeatedly call `collect_books` for the current player until it returns `false`, and then
   1. return `true`.

   If the request fails, the player must "go fish" by drawing a card from the deck into their hand:

   - If the deck is empty, move all of the current player's cards into the deck and remove the player from the game. Print a message to `std::cout` saying so, record in the game object that this player takes no more turns, and return `false`.
   - Otherwise, return `true` if the drawn card has the rank the player asked for, or `false` if it does not.

1. Implement the `deal` method of `GoFishGame`. Shuffle the deck and deal one card at a time to each player until every player has the right number of cards: seven cards each with two players, or five cards each with more than two players.

1. Implement the `play` method of `GoFishGame`:

   1. Call `deal`, and then, for each player number, repeatedly call `collect_books` until it returns `false`.
   1. Repeatedly iterate over all players remaining in the game, in rounds, until the game ends. In each round, for each player in turn, repeatedly call `turn` for that player until it returns `false`, then advance to the next player.
   1. At the end of each round, print the round number and how many books each player has made, and check for termination. The game is over when all cards have moved from the deck and players' hands into players' books, or when one or zero players remain in the game.
   1. When the game ends, print which player (or players) made the most books.

### Program Integration

1. In the source file containing your `main` function, modify the `create` function so that if `"GoFish"` is followed by a valid deck type (`"HoldEm"`, `"Pinochle"`, or `"Uno"`) and then between two and five player names, it uses `std::make_shared` with `GoFishGame` — parameterized with the suit type, rank type, and deck type matching the requested deck — to initialize the shared pointer that `create` returns.

   **Because the `GoFishGame` constructor may throw an exception, surround that logic with a `try`/`catch` block so that any exceptions are handled within `create` itself.**

1. Modify `main` so that it accepts `"GoFish"` followed by a deck type (`"HoldEm"`, `"Pinochle"`, or `"Uno"`) and between two and five player names as a well-formed command line. In that case, call `create`, and if the returned shared pointer is non-null, return the result of calling `play` on the `Game` object it points to.

### Building and Testing

1. Run `make` and fix any errors or warnings that occur. In `ANSWERS.md`, record the kinds of errors or warnings you encountered (you do not need to list every instance), or note explicitly that you did not encounter any.

1. Run the executable repeatedly and test:

   - incorrectly formed command lines, due to the game name, the deck type, or the number of players, and confirm that the program prints a helpful error message and returns an appropriate non-zero value for each kind of failure
   - that Hold 'Em and Pinochle still work correctly per the Lab 2 specifications
   - that Go Fish works correctly with each of the three deck types, including:
     1. the correct number of cards is dealt to the correct number of players
     1. turns and alternation among players work correctly
     1. scoring and tracking of books work correctly
     1. players leave the game, and the game ends, at the correct times

   Document which trials you ran and what happened in each in `ANSWERS.md`.

1. Update `ANSWERS.md` with:

   - your names
   - your email addresses
   - the lab number
   - whether you encountered warnings or errors while developing the solution, and what they were
   - what the executable did for each trial you ran

## Deliverables

Commit and push all modified and added files to the repo.
