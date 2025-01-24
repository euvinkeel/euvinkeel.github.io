---
draft: true
---

#project

When it comes to projects, the only ones I end up actually completing tend to be games or tools I find myself repeatedly thinking about and missing over the course of days or weeks.

After playing [LinkedIn's Tango game](https://www.linkedin.com/games/tango) for a few mornings, I found myself wanting to play more than one puzzle a day. Shortly after, I found [another clone of the game online](https://www.tangogame.org) and have been practicing there ever since. It's almost like a training ground that prepares me to speedrun the official Tango puzzle so I can be told that I'm "smarter than 90% of CEOs".

I'm not quite sure how they figure that out, but it's nice to see in the mornings.

There are some things that are lacking in the existing clone, though. Some nitpicks:
* official puzzles have less squares filled in, and deductions rely more on the equals/opposite signs between squares
* some of the clone puzzles can't be solved through one-step deduction, like how the official ones are carefully designed for
* the style of the boards look pretty different

To get in the groove of making websites and projects, I decided to make my own clone of [LinkedIn's Tango game](https://www.linkedin.com/games/tango).

* using vite, typescript, react, and rive.app for animations
* I have some previous experience with react, but always felt missing knowledge and super shallow depth
* barely touched web dev
* visual mockup/what i'd like to see
	* important to determine how dynamic things have to be and how to structure code around it
* planning process thoughts

* traditional easy way to render stuff is too trivial to be interesting
* slowly realizing highly dynamic behavior requires me to dive into how browsers handle animations

---
# Draft

* I need a board display component.
	* custom dimensions
	* specify tiles
	* specify constraints between adjacent tiles
* I need a way to generate valid puzzles with only one solution and can be deduced step by step.
	* This one is tricky, so I'll come back to it later

* Use Typescript and React to manage state -- no third party state management libraries
* Static page only

* Board state: a big 2d array. can also define width and height just by looking at amount of rows and length of those rows. make immutable
* each cell in that array has a tile type state:
	* none
	* sun
	* moon
* also, board can have a list of constraints that have two cell coordinates and a constraint type: x1 y1, x2 y2, constraintType
* i also want dynamic animations for tile changing state

gonna try to make the basic react component for an animated board (will fade and grow into view), and its tiles. the tiles will be animated with react, but i'll take the opportunity to try and implement animation at the lowest level i can with the board since it should be simple
animation is important to me because basic rendering just feels too unpolished and basic, and usually highly dynamic stuff like animations require a rewrite of the whole system anyway. i'm used to really interactive and dynamic stuff from gamedev, so i expect to be capable of it on the web

In react, how would I handle the board reacting to state?
Normally, I would expect to use a `useState` hook for my game board component. It could simply hold the state there and render each tile with a `.map()`, and just work. But I'm having trouble thinking about it this way because I want the board to squish and stretch itself if the board changes dimensions.

But when I think about "re-rendering", I think about the board being deleted and re-constructed which is not good for smooth, dynamic animation. How does `useState` work? When does re-rendering occur in React, and how can I prevent it?

Maybe it's the nature of websites. Most websites consist of the same animations: simple tweens, fade-ins, scrolling, a little icon change when you mouse over something. Web frameworks seem to package these animations as dedicated components, or attribute names instead of offering something more programmatic.

But in gamedev, UI elements are like any other game object and can be programmatically moved anywhere. It can get messy when you have hierarchies defining parent-child relationships and you want to implement drag-and-drop, but it definitely feels more interactive and free. They can react to any input, move with tweens or spring calculations, fade in and out of existence.

In websites, most visuals are determined with CSS. It's great at defining where things are upon page load, but it feels like the wrong tool to use for the dynamic stuff described above. It tells elements where to be centered, where they belong in the class hierarchy. Maybe a preset can tell it to wiggle and bounce a bit, but that seems to be about it.

So React seems to be the wrong tool for a game board.

Maybe React is the wrong tool for this. How would this be achieved in plain Javascript, without using libraries to handle animations?


Why do we have to call `requestAnimationFrame`? Why isn't there some event loop we hook into and attach a callback of our own there, and define render priority?

The browser definitely has its own event loop, but developers only get an animation specific method.
I guess we're only allowed a certain priority by the browser. We ask the browser to call that callback right before the next repaint, so we can make the proper calculations of any values for the animation. In the [docs examples](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame), they simply change a DOM element's `style.transform`.
```js
element.style.transform = `translateX(${shift}px)`;
```
What rubs me the wrong way is that we're assigning a string value every frame. Why aren't there strict typed numbers we're setting? Wouldn't that make the most sense if the DOM is our API into the webpage, that it would provide these common sense numerical values? Why are we *setting a string*, to then be parsed by some CSS parser and then apply a numerical change?

The answer for programmatic animation seems to be with the [Web Animations API.](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API)

I came across some arguments for why basic CSS animations are needed for performance, especially on mobile devices. Javascript isn't the fastest language, it can get complex, and it makes sense to use CSS animations because simple transitions are easier to be hardware accelerated by the GPU... probably. Again, my intuition and understanding here was very vague, so I read some more:
https://css-tricks.com/myth-busting-css-animations-vs-javascript/

And here's something I found out to answer the above:
>Declaring your animations in CSS allows the browser to determine which elements should get GPU layers, and divvy them up accordingly. Super.

So that's why CSS is really important at build time.

>**But did you know you can do that with JavaScript too?** Setting a transform with a 3D characteristic (like `translate3d()` or `matrix3d()`) triggers the browser to create a GPU layer for that element. So the GPU speed boost is **not** just for CSS animations – JavaScript animation can benefit too!

The article was a really good read. It basically laid out every flaw I've thought about website animations and more.

Searching for more about the topic on Hacker News lead me to this talk:
https://news.ycombinator.com/item?id=39598211
https://nolanlawson.com/2023/01/17/my-talk-on-css-runtime-performance/
https://www.youtube.com/watch?v=i793Qm6kv3U - react talk
https://www.youtube.com/watch?v=7YhdqIR2Yzo - react youtube series

React is describing its own DOM, its own tree of elements. Very quick to modify this internal data tree of JS objects.
That's what's being created when `render` is called. Nothing to do with the actual DOM yet, this is only the virtual DOM.
The diffing algorithm is important -- it's where it automatically figures out how to optimally transform the tree (the `key` field is important for efficient updates as well)

The actual DOM, aka reconciliation/mounting/unmounting, is handled by **other libraries called renderers** depending on what platform you're displaying stuff on: html websites with React DOM, mobile apps with React Native. **React is the abstraction and renderers are the implementation**

How does the renderer package come in? React *communicates with the renderer*.
Every renderer will set its own field on component instances upon creation: `updater`.
Hooks like `useState` use a "dispatcher" object (set inside React as `__currentDispatcher`), which is forwarded all calls of `useState`.

so when React makes a component *instance* (an object with the fields `$$typeof`, `key`, `ref`, `props`, it has its own lifecycle: we use hooks like `useState` and `useEffect` to do stuff with that. So they're mutable?

[[2025-01-23]]
11:23
okay
when i create a board, it should tween its size and reveal its children
HOW though
and how should each tile be sized what should it be based off of

tiles should not be resized by the board
tiles should be sized based on the window size somehow
if the board changes size, it will simply reveal more tiles
i think if it's horizontal mode, then change the tiles based on height of window
if it's vertical mode, change based on width of window?
so maybe a big upper div that is the "game container", and the board and tiles reside there
i could just use react-spring or implement my own spring for animations

how should i handle global state

okay turns out that `render` methods are done by the renderer go figure fdsfs

it's time to really get into the weeds of what `useEffect` and `ref` are

## What's `useEffect`?

`useEffect(setup, dependencies?)`

Used to execute code outside of the React framework, and have side effects. (network calls, animations?)
The `setup` argument is a callback, called when the component is rendered. This `setup` function can also *return a cleanup function.* It's run when the component is added to the virtual React DOM, but further re-runs are determined by what you pass into `dependencies`.

`dependencies` is a *list of values referenced inside `setup`*, including props, state, variables, functions declared within the component. If omitted, **every re-render** will re-run `setup`. That's why empty arrays are often passed to prevent `setup` from re-running too much, like if a textbox changes with every keystroke.

When the component is added to the DOM (virtual DOM, or actual DOM? since this is all within React and not React-DOM, it must be the virtual DOM), the `setup` function is run.
https://react.dev/learn/you-might-not-need-an-effect

## What's `useRef`?
A ref is a value that isn't needed for a component's visuals: in other words, a value that if changed **will not trigger re-renders**. e.g. storing *local, component-scoped* info between re-renders. These are changed within event handlers or `useEffect`s, and cannot be read or changed *during rendering, aka inside the component body*. That's for state, not refs.

You can pass the ref object from calling `useRef`  -> into the `ref` attribute of the component JSX.
When React-DOM creates that component instance in the actual DOM, you can access that DOM node with the ref's `.current` value to use the DOM API.


okay. to keep things simple, i won't animate the board at all. instead i will simply animate the tiles and their state changes. if the board changes size then it'll just refresh rerender like react intended without fancy animations. maybe a fadein.

how should i think about scaffolding a web project?
first:
* determine what's definable at runtime versus build time - what is static, what is dynamic?
* boards can be instantiated dynamically.
* the board's size (height and width) are layout-expensive, is static
* the board's tiles are instantiated with the board
	* but their displays *are* dynamic and animated
* boards will fade in and grow in with fast css animations
* tiles are handled by rive state machines

learning more about CSS
https://www.youtube.com/watch?v=i1FeOOhNnwU
* everything is blocks, start from top and go horizontally
* flexbox
* avoid absolute positioning

in the end, i just want a better tango game generator.
but also one that can test smaller bite sized patterns.

STEPS:
- [ ] Make a div that squishes and stretches. and fades in.

## TODO
- [ ] Create a React display for boards and tiles
- [ ] Create a full 6x6 puzzle generator with exactly one right answer
- [ ] Make win states/error detectors for invalid puzzle configurations
