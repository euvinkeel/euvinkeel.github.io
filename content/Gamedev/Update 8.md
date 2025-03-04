Added a bit more to the graph and finally started coding some things.

![[Pasted image 20250303174402.png]]

I probably won't post everything about the graph because it changes so often (which is what I like about designing on a whiteboard!)

The current challenge is writing a typescript code generator script that takes this:

![[Pasted image 20250303174524.png]]

... and turns them into autocomplete-friendly method calls that I can use on game systems, like

`actions.spawnPlayer()`

which not only executes the code within the definition's `execute`, but also appends some replication-specific code in the background and deals with translating SIDs/PIDs that I don't have to write in the definition's `execute` everytime I want to make a new action.

---

Okay, I think I got most of the transformer script working.
I'll just leave this very simple diagram of my setup here.

![[Pasted image 20250303235445.png]]

Probably the jankiest thing I've ever done, and I had some help getting started with ChatGPT and this really neat tool to even begin getting a sense of traversing things: https://ts-ast-viewer.com/

Big thanks to [ts-morph](https://ts-morph.com)!