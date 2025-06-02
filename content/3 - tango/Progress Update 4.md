---
draft: false
---
From last time, I just added two buttons: a "regenerate" button that creates a brand new game, and a "reset" button that clears the tiles you edited. And I added their proper icons as svgs.

![[vivaldi_PThZ8OQWU5.gif]]

I tried thinking of what more features a final product should have. So I came up with the following use cases:

* a specific pattern to practice on, or a specific board
	* may or may not have a timer
	* no regeneration button
	* has a reset button
* a plain, casual, normal infinite game
	* has a timer
	* has a regeneration button
	* has a reset button
* a ranked daily game
	* has a cover over it to start the timer when ready
	* has a timer
	* no regeneration button
	* has a reset button

This made me revisit how I should decouple the code I write from here.

In chess games, the state of a board is just static data -- there's no info if a position is impossible, is a checkmate, what moves are allowed, etc.

Then, there are "rules": code that can evaluate what is allowed, if a move is possible, if a move results in check or mate. But something still has to run this, to actually enforce and execute this legislative code.


Then, there is the "engine": code that can now *listen* to a player's input and *calls* the legislative code on the state of the board. It is stateful, and this is where auxiliary features can be added (a timer, a regeneration button, a reset button, a cover before the game starts.) 
* The engine can potentially use or make up new rules, possibly interpreting legal/allowed moves differently. The main point is that the engine is built *on top of* the rules and board data, the prerequisites.


Finally, there is the "presentation": printing out emojis on a console, or rendering HTML components, or full Rive animations. This listens to the engine's *events* to determine what animations occur or when to render things.


Currently I have an "engine" and "presentation" all merged together under a React component that does a naive re-rendering per move -- the board state is a simple `useState` that holds the entire board state, and the grid iterates through all cells to call `<Tile/>` components. I think I have to refactor my code a bit so they are truly decoupled; to prove this, I'll have one engine and three different presentations that must both hook to the same engine (and work).


* One presentation can live in the console. You can paste code into any JS session, like `node` or your dev console, and play the game by typing in coordinates to click on or commands to refresh/clear the board.
* One will be a simple visual board on the page with no animations (what I have now).
* One will be a very pretty board on the page with all the fancy animations.


Bonus points if we have all of them simultaneously running and showing the same game with the same moves.


So what would my engine look like?

I'll call it "TangoTS". It's an object that can be instantiated within a JS runtime, and you provide it callbacks that should occur whenever you make a move, a brand new game is made, an error is made, a win state is achieved, etc. Any separate renderer you write will provide callbacks here to know when to change itself.


The CLI renderer, "Tangomoji", will use a TangoTS engine to output game state onto a console and accept user input via typed coordinates.

The simple renderer, "TangoHTML", will use a TangoTS engine to re-draw a still grid onto a page and accept user input with button presses.

The pretty renderer, "TangoRive", will use a TangoTS engine to draw a pretty and smooth animated board and accept user input similarly to TangoHTML.


In preparation for the animated version (and partly because I'm still figuring out how to use react-spring appropriately) I've refactored the code so that I now have a separate TangoTS class, where you can pass it into a TangoHTML component. To illustrate the connection, here are two separate components using the same source of truth:


![[vivaldi_EKai21sIa2.gif]]
