* Decided the goal is to make an all-in-one batteries included React component that housed a full playable tango game (like how chess sites can embed a full playable chess board anywhere for you to practice specific configurations)
* been looking at too many articles about styling, react state, re-rendering, and i just need a working game
* Decided to make a system where the implementation of the game can be run, rendering-agnostic
	* didn't want to use react hooks as the sole manager of state
* start implementing game logic with a really basic re-rendered board (but functional, and works)
* wrote a bunch of utility methods for changing board state, seeing if it breaks any rules of tango, highlighting erroneous squares, enforcing constraints on two squares, and checks for a simple win state (LLMs were really helpful in quickly making these)
* ![[vivaldi_yNJpKutBnf.gif]]
* will probably publish source code and try to make it polished later

[[2025-01-27]]
now i have a working tango implementation without animations!

time to make it pretty.
animations must react to a lot of changes:
* when a tile changes (i have the rive animation for that)
* when a win state is achieved
* when a rule is violated (delay error highlighting by a few seconds)
* when the board changes dimensions?
	* really advanced and harder to visualize

it's not just animations, though. say that a webpage has to react to what happens on this tango board - should it log the solve time in a database, give the user some reward or popup, or do nothing?

would it be event driven? maybe instantiate a whole `TangoGame` object and supply it some callbacks? i think that's the way to go.