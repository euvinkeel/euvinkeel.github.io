Today's Monday, so I thought I'd lay out exactly what I need done by the end of this week.

1) Have a working animated rive board by Thursday.
2) If I don't, have any working playable board hosted on a Github page by Friday.

I expect the bulk of the work to be learning how to best handle dynamic components in React (or in spite of React), designing some code architecture to reflect that, and implementing it.

Last week I encountered the following issues:
* my `TangoTS` API was designed around callbacks (e.g. `addChangeCallback` or `addWinCallback`) to allow other code to trigger animations
* `useEffect` hooks were confusing and, combined with callback patterns and using Rive components/hooks (some of which are `null` initially and rely on `useEffect` to wait until they were loaded in)
* for regenerated boards to flip into view (and out of view), I had to learn how react-spring's `useTransition` worked so each board (and state) is treated as ephemeral
* all the above combined together resulted in a lot of trial & error at midnight

I'd rather not repeat the above, and will instead explicitly write out all of my thoughts & reasoning. It's a bit embarrassing to reveal how little you know about these frameworks and hooks, but I'd rather show my ignorance publicly so that I'm way more thorough in my reasoning (and maybe others can correct them).

---

What system of state management do I have right now?

I have:
* `TangoTS` class
	* contains an internal `_boardState` property, which holds a grid of tile states + rows + columns.
	* contains `addChangeCallback`, `removeChangeCallback`, `addWinCallback`, `removeWinCallback` where code can pass in a (`key`, `callback`) in order to run code when certain game events occur (such as any changes to the board, win state being achieved)
	* changing the `_boardState` is done *immutably*, with `_boardState` being replaced every time it's changed -- this is so that when the board changes, callbacks get `oldBoardState` and `newBoardState` as arguments
	* contains `changeTileAtIndex` to change a specific tile, `regenerateBoard` to create a new board configuration (with the same rows and columns), `resetBoard` to clear player-editable tiles

* `TangoRive` React component
	* Accepts a `tangoTsAPI` prop in order to drive its animations.
	* Displays a `TangoRiveBoard` for the actual game, plus auxiliary components like buttons and a timer.
	* Uses `react-spring`'s `useTransition` and a `useEffect` hook to hold a `BoardState[]` array, called `activeBoards`, that holds exactly one `BoardState` at a time (which is always whatever the `TangoTS` object has as its `_boardState`.)
		* The reason why we hold an *array* with one item instead of just one actual `BoardState` value is because of how `useTransition` works. It keeps track of ephemeral items in an *array*. It plays an animation (e.g. a fade in) whenever an item is added to the array, and another animation (e.g. a fade out) whenever it's removed from the array.
		* Identity is determined by `key`, which it "automagically" determines on its own. This is important when it comes to the "old board flips out, new board flips in" animations playing simultaneously: two `TangoRiveBoards` may be on-screen playing their animations at the same time, and must have their own consistent identity.
		* Upon `TangoTS` calling `changeCallback`s, if a brand new board is generated, `activeBoards` is replaced with a brand new array with the new board state. If not, the current `activeBoards` array at index `0` is set to the new board state.
			* this makes it so that normal moves do not trigger the replacement of the entire board with fade in/fade out animations, while regenerations do

* `TangoRiveBoard` React component
	* This displays the board grid (the one doing the flipping and fading in/out) with each of its tiles being an animated `Rive` component for individual icon changes.

Currently, there are a lot of flaws: after mounting, the individual tiles in `TangoRiveBoard` have to wait for some `rive` hook to load before they're usable, they somehow can't hook onto the `TangoTS` `changeCallback` if the new board is regenerated since the `rive` hook is always null somehow even though I see the `console.log()`s printing them as non-null objects while simultaneously printing them as null objects right after that, probably because of how `useEffect` works or how function closures work...

It's all so confusing to type out or even put into words, so this is a sign that I've got to learn more about these hooks & revamp my design choices.

# Rive
The thing that drew me to Rive was their state machine feature. Essentially, Rive allows you to use a visual editor to create a state machine and define how animations play as state changes. This allows designers to create their own "module" of defined parameters and behaviors into a single `.riv` file, which is passed on to the developer to be implemented in whatever runtime environment they develop in.

In practice, because the logic of the state machine has already been defined by the designer, it should be trivial for the developer to simply hook events together. In order to use Rive components in React, most examples have DOM element events (like onClick or onMouseEnter) to trigger a code block that changes "Rive state" *imperatively* and *mutably*. To do this, you use a `useStateMachineInput` hook and set its `value` property.

Furthermore, it may not be possible to access such state upon mounting until later (from [the docs](https://rive.app/docs/runtimes/react/parameters-and-return-values#usestatemachineinput:~:text=The%20return%20value%20which%20is%20the%20state%20machine%20input%20may%20not%20be%20immediately%20available%20due%20to%20the%20need%20for%20the%20rive%20instance%20to%20resolve%20first.%20You%20may%20want%20to%20use%20a%20useEffect%20to%20watch%20for%20when%20the%20rive%20instance%20and%20the%20return%20value%20of%20the%20useStateMachineInput%20hook%20has%20value)):
>The return value which is the state machine input may not be immediately available due to the need for the `rive` instance to resolve first. You may want to use a `useEffect` to watch for when the `rive` instance and the return value of the `useStateMachineInput` hook has value

My approach was to have every tile do a hacky `setInterval` loop, checking repeatedly if the object returned by `useStateMachineInput`. But that never really helped.

I think the biggest problem here is that the React runtime was built for very specific use cases, and mine was a step too far outside of what it was meant for. I think I'll try looking at the plain JS web runtime.

...
![[vivaldi_LiJ4wq170d.gif]]

...and now it works perfectly. Turns out I had a good enough idea of react hooks that I could implement the plain web runtime just fine within a react component. Now, it 1) initializes to the correct tile state, 2) properly changes on click even when it's a newly regenerated board, 3) appropriately adds and removes callbacks from `TangoTS`. When I spam-regen the board, the amount of callbacks attached to `TangoTS` stay consistent!

However, there's a memory leak. Apparently with each new board regeneration, the old ones stay as detached nodes. I really don't like how I'm beholden to the black box that is `useTransition` and I don't like trying to work with them, so I think I'll attempt to make my own state without that hook.

I'd need to keep track of potentially multiple board states (spamming the regeneration button should allow you to see multiple boards flipping in and out of existence at once), their lifetimes, and handle instantiation + cleanup.

I'll figure it out tomorrow.

