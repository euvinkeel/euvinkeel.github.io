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


Eventually I reduced it down to the version of the library I was using.
Apparently there was a bug that broke one of the API methods which threw a wrench in my whole system, and I'm pretty sure that bug is causing a lot of other mayhem somewhere else. I opened a github issue.

The nature of the bug honestly kind of stops all development effort on this front, because it arises from pretty fundamental features of the library. I'll wait and see, maybe I could help contribute and really get into the nitty gritty of the bitwise math they're doing.