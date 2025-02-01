---
draft: true
---


I already have TangoHTML (will probably take some refactoring), and I think Tangomoji will be pretty trivial. So for now, I need to learn the best way to make TangoRive.

I tried to visualize what specific animations I wanted.

* the board should flip into view using 3D transforms and fade in
* every tile should have a smooth animation to transition to any state (sun -> moon, moon -> sun, sun -> blank, etc)
	* each tile will track its own coordinates for changes and animate itself
* upon regenerating a new puzzle, the board should flip into view again kinda like when it initiated it
* upon resetting, the board should NOT flip, but every (editable) tile will animate itself into the blank state
* upon a win state, the board should do a rolling wave across every tile: changing the tile colors to be brighter, rotating them, and they all settle back into their place, maybe a shine animation plays. (so just have a win animation for every tile icon that gets called based on some timeout + offset for the rolling wave effect)
* upon an error state *being present for at least 1 second*, the erroneous tiles should flash red.
	* this should be some kind of filter/additional layer *on top* of the icon so they can animate independently. if the error gets corrected, the red should fade away.

so what does all of this imply?
* I'll have to use [CSS 3D transforms](https://www.w3schools.com/css/css3_3dtransforms.asp) for the board
	* When the board rotates, create new tiles and don't reuse the old ones, as if the new tiles were on the back of the board and that back of the board is rotating into view
	* if the board size changes state, this flip has to happen - boards should not shrink or grow, old boards will disappear and new boards will just flip into view
* I'll use rive files for each tile
	* tiles must track react to changes on some consistent coordinate they're assigned to


I understand that the way to animate stuff programmatically with JS is to change the CSS style, but I was wondering what would be the most performant. Changing CSS properties by making a template string and then assigning a string that has to be parsed in the background just doesn't sit well with me, when I'm used to setting strict numbers to move things around. Why am I typing out the string "transform:" to set numeric values?


Anyway, because I really like using springs for smooth animations, I looked into react-spring. Reading documentations for libraries that you've sort of worked with in a completely different context can be a challenge because you're trying to re-wire all of your assumptions. That said, I found really cool example projects that I could learn from.

https://codesandbox.io/p/sandbox/mdovb?file=%2Fsrc%2FuseMedia.ts%3A6%2C27

https://codesandbox.io/p/sandbox/6zchkl?file=%2Fsrc%2Fcomponents%2Fhooks%2FuseWindowResize.ts%3A3%2C31

https://codesandbox.io/p/sandbox/v1i1t?file=%2Fsrc%2FApp.tsx

This board should be responsive to its container's size and smoothly resize itself. I yoinked this from an above example:

https://www.npmjs.com/package/react-use-measure

this is how i want to call stuff:

```typescript
cosnt api = TangoTS(
	onWin: (timeElapsed: number) => {
		console.log(`wow, congrats for finishing in ${timeElapsed} seconds!`);
	},
	onChange: (boardState: BoardState) => {
		if (analyzeBoardForSpecificPattern(boardState)) {
			console.log("There's a pattern!");
		}
	},
	config: {
		isCoveredBeforeStart: true,
		rows: 6,
		columns: 6,
		enableTimer: true,
	}
)

// if the site ever wants to present a new challenge, it's exposed through setNewBoard
api.setNewBoard({
	rows: 4,
	columns: 4,
	constraints: [ ... ]
	tiles: [ ... ]
});

// or it can demonstrate stuff.
api.changeBoardTiles(
	[
		[[2,1], TileIconType.SUN],
		[[1,1], TileIconType.MOON],
		[[5,5], TileIconType.MOON],
	]
)

// it can also expose its regenerate or reset functions
api.regenerate();
api.reset();

// and it may give rewards upon winning within the board with the passed in callback to `onWin`

// to render in app...
<TangoRive tangoTsApi={api}/>
// the api should have these

tangoTsApi.setWinCallback(() => {
	// animation code here
})

```


https://labs.factorialhr.com/posts/hooks-considered-harmful
https://www.zhenghao.io/posts/memo-or-not


