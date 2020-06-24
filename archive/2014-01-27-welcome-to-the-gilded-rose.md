---
layout: post
title: Welcome to the Gilded Rose
---
I recently began working on the Gilded Rose Kata in Ruby. This is a fairly famous exercise originally written in C# but translated into several different languages*. Unlike the other katas I've worked on so far, the Gilded Rose Kata begins with pre-existing code, and is primarily an exercise in refactoring.

The premise of the Gilded Rose Kata begins with a store selling various items. Each night they update the qualities and sell-by dates of the items in their inventory using a method their previous developer wrote for them. There are several rules that dictate how these values are updated--for example, most items decrease in quality each day, but "aged brie" increases in quality, as do concert tickets as the day of the concert gets closer (but it then drops to 0 after the concert). The idea is that the store now plans to sell a new kind of item with yet another unique set of rules, and you are tasked with updating their system to accommodate these new items.

The code you inherit is such a mess that it takes several moments just to figure out what exactly is going on for the old stuff that works. My first reaction was disgust at how ugly it all looked, but quickly turned into a kind of fear as I wondered how in the world I might update the code without breaking something. This immediately reinforced the value of tests--how else would I be sure that, while re-writing the code for readability and extendability, I didn't break something that, though ugly as sin, got the job done correctly? Naturally, the previous developer didn't write any tests, so the first thing I did was write several tests to provide as comprehensive a test suite as I could right off the bat. This was actually a fairly difficult task, as it's not very easy to tell after the fact what exactly should get a test and how those tests should be structured. I eventually settled on a set of specs that broadly covered all the cases I could think of so that I could get moving, even though there probably would have been more tests if the code was written in TDD style.

Once I had some tests to provide a bit of a safety net, it was time to move on to refactoring the code. I can't tell you how long I stared at the code just wondering where to begin. I decided to first aim for improved readability. The Ruby version of the kata comes with a bunch of nested "if not" statements (ex. `if item.name != "Aged Brie"`), so my first step was to simply rewrite all the if statements as positive declarations, as those are generally easier to follow. From there, lots of patterns began emerging and things began falling into place. Much like the other katas like Prime Factors and Roman Numerals, in which specific rules gradually give way to general, all-encompassing algorithmic patterns, it is very cool to see the Gilded Rose code start to show patterns and repetitions that can be refactored into much simpler and more straightforward code.

I'm not done working on the Gilded Rose yet--I reduced the code down to something much more workable, but I haven't yet added the new item functionality. I found a video online of someone performing the kata in Ruby, and this person has a very nice looking solution that is similar to mine, although I think one thing he did is actually incorrect. Interestingly enough, I believe this person's mistake can be traced back to an incorrect test in their suite that falsely suggests the code is functioning properly. Once again, this illustrates the value of Test Driven Development--when you strictly follow the rule of writing tests for the most basic functionality possible and writing only the least amount of production code that can make that pass test, you will be prepared and covered for all exceptions and edge cases.


*for better or worse. Some have argued that the kata should only be done in the original language, since if the store's existing system was in C# you wouldn't rewrite the whole thing in, say, Ruby. It's a good point, but I think there's value in the exercise for developers of all languages and skill levels, not just C# devs.