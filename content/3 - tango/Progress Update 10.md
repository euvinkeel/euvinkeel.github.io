Friday - i’m gonna wrap up my tango project
* make it accessible on mobile
* make a small switch to enable actual random puzzle generation
* add super basic, quick analytics

----

After some struggling, I made it responsive for portrait screens! The board should scale based on screen width. (Couldn't find a good way to do this with CSS only.)

I'm getting a bunch of errors from Rive components saying a really vague message `Problem loading file; may be corrupt!` but as far as I can tell, this error fires for almost anything and my board game works perfectly fine.

To be more performant, the game reuses some pre-generated board states when you click on the regenerate button. But if you need a *brand new* generated puzzle, simply press & hold on the regen button and it'll make a new one. May take a few milliseconds. (or seconds.)


Also, I added PostHog analytics! I've used it before for my previous game projects, but those games weren't using web frameworks or Javascript. Now I get to actually use PostHog as its creators intended: for websites. Super quick and easy.
![[Pasted image 20250207145125.png]]



I think that'll be it for this tango project.
The final version is up at https://euvinkeel.github.io/tango-trainer... and I'll put a retrospective later.