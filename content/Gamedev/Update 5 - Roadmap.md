I didn't do much yet as of writing this: this will be more of a roadmap and an intention setting thing. I won't predict dates, since those are always so inaccurate they're practically useless.

Here's what I've accomplished:
* created and started the project & group
* integrated JECS, got it working
* integrated Jabby + Planck, got it working
* learned about JECS relationships, components, entities API
* did a bunch of typescript-lua hacky shim work to get the above to work
* modeled first batch of game assets in blender

Here's what I have to do, top of my mind:

**Framework goals**
* Make a replication system: create some way to share/distribute state with clients & server.
* Make a reconciliation system: a system so that JECS state can change the Roblox datamodel, or listen to the datamodel as needed for roblox specific event firings
* With the above reconciliation system, make my own basic UI system using only JECS and vanilla roblox UI -- no UI frameworks. (Considered [vide](https://centau.github.io/vide/tut/crash-course/2-creation.html), but I don't want to learn more libraries.)

**Gameplay goals**
* Get tower/building placement working.
* Get enemies spawning + attacking working.
* Make them drop loot, or some way to spawn a reward chest for new items.
* Make enemies infinitely varied with traits and aspects.
* Make an infinite round system to orchestrate the above.


That's all I'll write for now. No more planning beyond those steps. I'm sure I'll have to revisit each of these as I go on, but this will simply serve as a compass for what to work on next. 

