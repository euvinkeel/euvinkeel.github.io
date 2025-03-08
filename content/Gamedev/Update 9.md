---
draft: false
---
![[RobloxStudioBeta_O2x6xP0Md5.gif]]

what's this?
this is what peak replication looks like

![[RobloxStudioBeta_mVoXK7eCiV.gif]]

jk probably, but i like of what i got here

the gifs above represent clients catching up to server state as it changes random stuff over time. basically red means their client world (the entities that have a shared existence on the server) is not the same as the server's (they compare hashes). green means they've caught up, and see what the server sees; the entities, component data, and relation pairs are all synced

the emojis printed onto the console are derived from the hashes; just a more visual way of seeing them on screen

the past few (several) days (weeks) have been me designing a way to represent the world, how partial updates/diffs should work, how client actions/server actions should update those, and how they should be replicated to the player

naturally i wrote a lot and overthought a lot

![[Pasted image 20250308001115.png]]

these are all probably solved problems, but for the tools and code i use i just had to reinvent a lot of stuff. i could make it so much more efficient (that's the next step) but it's so good to see things work

![[Pasted image 20250308001530.png]]

also [clumsy](https://jagt.github.io/clumsy/) is a cool tool to ensure your tests undergo some really rough times

![[RobloxStudioBeta_1Iez9M6dgy.gif]]

maybe i need a break