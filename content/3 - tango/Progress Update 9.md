I studied a [book](https://www.amazon.com/Beyond-Cracking-Coding-Interview-Successfully/dp/195570600X) for most of the day.

I figured that I should quickly wrap up this project and decide my next ventures ASAP, since my interest is waning a bit.

I had plans to implement "Tangomoji", a separate rendering of the Tango game that only used console logging to show how different renderers could all use the same TangoTS module. But I think I'll skip that for now.

I'll focus on:
* adding a timer that starts upon first input
* making the page cleaner and more presentable
* adding the game rules at the bottom
* hosting it on its own github page

And after that's all finished, I'll write a postmortem and thoughts for the future.

----

I'm pretty pleased with how the site turned out.

![[vivaldi_sco0EwrCL1.gif]]

Let me see if I can shoehorn in Nix w/ Github Actions to make some build process.
https://determinate.systems/posts/nix-github-actions/

---

After some wrestling with ChatGPT (i wasn't about to re-learn a bunch of things from fundamentals at 1 am) I got deployment working, changed the favicon, and added social links.

Tomorrow I'll ensure it works on mobile, and then the entire project is complete!

It's live now at euvinkeel.github.io/tango-trainer.