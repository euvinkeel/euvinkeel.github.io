---
draft: false
---
so that was a fake blogpost last time because turns out, i didn't solve replication! in fact, for the past few days i've been stun-locked on wrestling the weirdest bugs from JECS.

it had to do with how relationships were affecting some really internal stuff in the world state. i used relation components to track what entities affected eachother whenever a server or client called an "action" in the replication system i was making. blah blah blah and i also tried to map it all out on excalidraw.

![[Pasted image 20250310235502.png]]

some Saturday scribbles.
![[Pasted image 20250310235702.png]]

that was Sunday.

![[Pasted image 20250310232654.png]]
![[Pasted image 20250310232617.png]]

then today.

![[Pasted image 20250310232704.png]]
![[Pasted image 20250311000229.png]]


Apparently there was a bug that broke one of the API methods which threw a wrench in my assumptions and thus my whole game.

Eventually I had a small reproducible example (couldn't tell you how the bug happens, it just does -- the library is super dense and optimized for performance after all) and I reduced it down to the version of the library I was using. Though, even if I did use earlier versions of JECS and applied my own hacky fix on top of the API, I'd still get the same issues except even more hidden and worse than if it had just failed outright. which i don't feel like typing all out

I opened a github issue. The nature of the bug honestly kind of stops all development effort on this front, because it violates pretty fundamental features of the library. I'll wait and see, maybe I could help contribute and really get into the nitty gritty of the bitwise math they're doing so I can actually suggest fixes.

i need to get gud at low level bit stuff