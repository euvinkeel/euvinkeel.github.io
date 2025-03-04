Added a bit more to the graph and finally started coding some things.

![[Pasted image 20250303174402.png]]

I probably won't post everything about the graph because it changes so often (which is what I like about designing on a whiteboard!)

The current challenge is writing a typescript code generator script that takes this:

![[Pasted image 20250303174524.png]]

... and turns them into autocomplete-friendly method calls that I can use on game systems, like

`actions.spawnPlayer()`

which not only executes the code within the definition's `execute`, but also appends some replication-specific code in the background and deals with translating SIDs/PIDs that I don't have to write in the definition's `execute` everytime I want to make a new action.