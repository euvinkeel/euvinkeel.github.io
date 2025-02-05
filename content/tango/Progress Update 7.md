Tuesday:

A lot of my problems with animation came from trying to wrangle & learn library behavior. First it was Rive, and now it's react-spring's `useTransition`. Currently, it's a black box where I have no idea how things are allocated or cleaned up. All I know is that I have a memory leak since disappearing boards are not being cleaned up.

I think I'm moving towards the need to make my own custom & imperative solution to handling the lifetimes of these components. This means `TangoRiveBoard`s should be dynamically allocated outside of the React rendering pipeline, so I'm going to have to do more `useEffect` trickery to do everything in plain JS.

But before I give up on `useTransition`, let me try to see what the actual source code for `useTransition` was.
https://github.com/pmndrs/react-spring/blob/next/packages/core/src/hooks/useTransition.tsx

...actually, seeing all the examples and using the devtools for memory, it seems like they all suffer from detached nodes and memory leakage.
Every time something new is created and supposedly destroyed, it has a chance of being a detached node or actually being cleaned up properly.

Now I definitely have to implement my own custom render logic.

I think the answer is simpler. In React, you just assign a `key` attribute to tell React whether to re-render a board or to simply leave it as is. That's easy if I leave basic animate-on-mount code, but how would I have animate-on-dismount?

Have internal state where I can have an array of some structs with `boardState` and `fadingOut`. At any given moment, `TangoRive` will use `map()` to render all boards that are either entering or exiting (so a board can be in the process of transitioning out, but will still be included in the render array) and each persistent board will have its own generated id to use as a key.

---

okay, so even with that simple approach of not using any other library to handle ephemeral boards, I still have a memory leak by React somehow not cleaning itself up despite my keys being properly unique (given that the animations work fine).

It's a bit jittery because of how inefficient my board generation algorithm is.

![[vivaldi_1zooD2mCzA.gif]]

The tiles are warped because of how Rive initializes itself. If I were to call its own `resizeDrawingSurfaceToCanvas` method at any moment, either the perspective of the board leaves it slightly warped anyway *or* if it's flat enough, it still visibly blinks and resets in an ugly way:

![[vivaldi_gOM6vuKPf5.gif]]

I've determined that animations and overall state management would be so much easier if I just do away with the whole "board flipping into view" animations.
This means having a static board with static tiles just instantiating a flat view of Rive tiles.

I think I'll take that compromise. Let's see how much I can do with just that for today.

---

I'll take this animation.
I had to ensure generating boards didn't interrupt the animation too much. So to temporarily solve this, I just put a bunch of pre-generated boards in a file. Whenever a regeneration happens, it just takes a board state from there.

![[vivaldi_87Uj9INqkF.gif]]

This blogpost is getting too long, so I'll just leave this with two different renderings working on the same board:

![[vivaldi_L2oVEZxyvN.gif]]