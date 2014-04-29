---
layout: post
title: Refactoring Clojure TTT
---

This week was spent focused on my Clojure Tic-Tac-Toe app. While some of my work involved adding new features (play again option, custom player tokens, etc.) the primary objective was refactoring for clarity and simplicity. Here are some lessons and tips from the experience.

### Don't abuse data structures
My original board data structure was a hash-map keeping track of size and spots: `(:size 3 :spots [nil nil :x nil nil :o nil nil nil])`. This closely mirrored the board class in my Ruby implementation. However, that size integer is problematic for a few reasons. First, its name is misleading--does "size" refer to the length of a board (the number of rows), or the total number of spots? More on this later. More importantly, though, the size integer is completely unnecessary and overcomplicates the board. We can simplify the board to just be a vector, and then if we ever need the board length we can calculate it with `(int (Math/sqrt (count board)))` (without int, Math/sqrt returns a float).

The takeaway here is that even though we can create a hash-map in Clojure to approximate a Ruby object, it's better to only use the minimum amount of data required to pass from one function to another. Keeping data structures lean will help you keep functions and their arguments clean by extension. For example, when I altered the spots vector in the original board data structure to reflect a move being played, I had to pass that function the board size as well in order to re-create a complete board hash-map. After reducing the board to just a vector, I am able to simply pass the vector in to the turn-taking function and return the modified vector. Much cleaner, nothing extraneous.

### Keep namespaces focused
This is the Single Responsibility Principle, plain and simple. Last week I had a namespace called "console-io" that was responsible for everything going on in the console: displaying a board, asking for configuration options, and declaring game results. In hindsight this is obviously problematic: the very fact that I use three commas to describe the namespace is a dead giveaway that too much is going on. Additionally, "console-io" is too vague.

I began by extracting everything to do with presentation into a "console-presenter" namespace. The name alone makes it more clear what this namespace does: it presents things in a console! I then moved most of the remaining functions into two separate namespaces: "console-prompter" (which prompts the user for input and grabs that input) and "console-validator" (which checks to see if some input is valid). I especially liked separating the functions in "console-validator" out from their original location, the "rules" namespace. Now "rules" is specific to the rules of Tic-Tac-Toe and isn't polluted by console implementation details. Definitely an improvement.

### Name things consistently and appropriately
My functions that print to the console are now all named with a consistent prefix, "present-". Consistency makes functions much easier for outsiders to understand: consider the difference between [ "display-board", "declare-result", "say-whose-turn" ] and [ "present-board", "present-winner", "present-current-player" ]. It is immediately clear that the second group of functions behave similarly to one another.

I also improved names that were incorrectly suggesting modifying state. For example, `update-board` sounds like there is a board object whose state is being altered. In Clojure, however, there are no objects that can be in various states; instead, Clojure functions simply accept data input and manipulate it to return different data. The word "update" is often associated with the concept of saving, which is not happening anywhere in my app. The new function name, `take-turn`, better describes what is happening in the game at a higher level and better conveys a sense of transience and flow--some things happen, then at this moment a player takes a turn, then more things happens.