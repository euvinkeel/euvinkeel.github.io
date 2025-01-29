---
draft: false
---
I had random generations and random constraints being put on a board, but now I have to guarantee that exactly one answer exists so it is actually possible to solve from the starting configuration: to fill the board without rule violations.

For reference, here are the rules from LinkedIn:

- Fill the grid so that each cell contains either a sun or a moon.
- No more than 2 of the same type may be next to each other, either vertically or horizontally.
- Each row (and column) must contain the same number of suns and moons.
- Cells separated by an **=** sign must be of the same type.
- Cells separated by an **X** sign must be of the opposite type.
- Each puzzle has one right answer and can be solved via deduction (you should never have to make a guess).

# Generating a playable board

I thought about learning about or porting a python library called `constraint`, which is supposed to solve any kind of problem given constraints, inputs, and ranges of inputs.

But I decided not to spend too much time learning a new library, creating the perfect or optimal searching algorithm, and did the most hacky code cowboy thing ever:

- in a (potentially infinite but practically never infinite) loop:
	- it randomly generates empty and filled squares with random constraints between random adjacent cells
	- it generates up to 2 solutions for a given board configuration using recursive backtracking
		- the solution generator does a "random walk": for every blank square, it will either traverse in the order of:
			- filling it with a sun, traverse, fill it with a moon, traverse, then return
			- filling it with a moon, traverse, fill it with a sun, traverse, then return
		- (the order is decided 50/50)
	- why only up to 2 solutions? if there are 2 solutions, that's all we need to know that the board has multiple solutions and the entire board is re-generated. if there is only 1 solution, the board is kept and we break out the loop

and it works!!!!!!!! it probably won't destroy the devices that run it... hopefully

I'll find some way to properly do this optimally later

# Guaranteed Deduction

Now I have to figure out how to make it so that *each step is solvable with deduction*, and not have people try to calculate levels and levels deep into some guess

I thought about hand-writing heuristics to check, like detecting specific patterns within a row or column (two same icons, constraints) and applying the rules I knew. but that didn't sit well with me, because what if I didn't know there was another "trick" to deducing the next move?

I eventually figured that all one-step heuristics depend on only analyzing the cells within a single row or a single column.

This is the rough algorithm I implemented:
- given a row or column that isn't entirely filled, we iterate all possible variations of it where every square is filled
- we check if the row or column is valid: if it is, throw it in some "possible outcomes" list
- at the end, compare every single possible outcome:
	- if there are squares that are the same consistent sign within every single possible outcome, we know those can be deduced: "solvable squares"


In other words, now we have an automatic way to tell if the next step is deducible by humans. We can have something like `isCellSolvable(coords)` to see if a square has only one answer, and can use this in something like a `findSolvableCells(board)` method.


How do we enforce this?

* initialize with a starting valid configuration of a board (with one solution), and enter a loop:
	* call `findSolvableCells(board)`
	* If there are solvable cells, fill them in and repeat loop
	* If there are *no* solvable cells, place a random tile from the solution onto the board and repeat the loop.

Essentially this makes the computer solve the puzzle step by step, filling in "obvious" cells, until it gets stuck. Then it takes a small hint from the solution and keeps going until the board is solved, resulting in a perfectly deducible game!

I'm pretty sure someone can find out the most optimal way to automatically generate a valid board with one solution all in one pass or something like that. But for now, I can focus on other things. And now I have code that can play Tango for me.


I wonder if I can make it solve tomorrow's puzzle.


# Discovering Solvable Patterns

During development, I spent a lot of time trying to figure out why sometimes the board was still too hard for me to solve on my own. When I looked at what suggested moves were printed out, I actually learned some new patterns that definitely have clear moves.

I've tried to write them down below (in no particular formal notation).


Let `S` be from the set `{SUN, MOON}` and let `X` be its opposite sign.

Let `_` be empty space

Let `_*_` be an opposite relation

Let `_=_` be an equal relation

`S _ _ _ _ S` can represent either a row or a column


* `S _ _ _ _ S`
	* `S X _ _ X S`
* `S S _*_ _ _`
	* `S S _*_ X X`
		* (or any pattern with two `S`, less than two `X`, and empty `_*_`)
* `S S _=_ _ _`
	* `S S X=X _ _`
		* (or any pattern with two `S` and empty `_=_`)
* `S _ _=_ _ _`
	* `S _ _=_ _ X`