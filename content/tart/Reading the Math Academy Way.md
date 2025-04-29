So for the past few weeks, most of my efforts were diverted from my gamedev project and into this project. Essentially, I was trying to make my own ["MathAcademy"](https://mathacademy.com) but for any curriculum that I create, for whatever I learn. Instead of presenting only flashcards, users would be presented "skills" web apps (as of now, more like React components) and their learnings would be demonstrated and tested solely based on bite-sized minigames or challenges.

![[Pasted image 20250429144454.png]]

I was reading more about the [FSRS algorithm](https://github.com/open-spaced-repetition/ts-fsrs) and how to implement it in my project, representing each skill as a separate card. This was shaping up to be a "mini-game" Anki, with certain "mini-game cards" locked or unlocked based on demonstrated performance by the user.

I even had my own demo website up and running, though a lot of effort was spent towards learning what Drizzle ORM and Upstash were.

![[Pasted image 20250429144148.png]]


At one point, it occurred to me that I should really be investigating the actual specifics of what MathAcademy does for their algorithm. And so for the past few days, I've been reading "The Math Academy Way": https://docs.google.com/document/d/1LLZK_34Oer9LwuqAv-pqxfXlR8n7V8zJ_MO323R7egI/edit?tab=t.0#heading=h.6pzpmilj161j

Aside from all the (very true) ramblings about how screwed the educational institutions are, I gleamed some valuable insights. As an Anki user, one thing I that I've overlooked all this time was that Anki treats flashcards as an unsorted set of independent, unconnected pieces of info. The FSRS algorithm they use seems to treat each card that way - a success on one card sets it aside for later, and other cards independently manage their own forgetting curve.

So when I read about the "Fractional Implicit Repetition" system MA (MathAcademy) uses, it inspired me to re-think the TART system I've been ideating for the past few weeks.

And when I read about the immense amounts of effort it takes to devise such a highly connected, thorough, and expert-validated curriculum (ignoring all the previous attempts at such a system I've found in [this hacker news post](https://news.ycombinator.com/item?id=40954571)), I knew that a different approach had to be made with any curriculum that I could devise by myself. There was no way I could devote resources to creating such a useful and streamlined curriculum worth paying for.

Would there? Maybe people's standards are shockingly low for what learning resources they pay for.

And I certainly don't need people to pay me to motivate myself to create this system, because I have a strong hunger for not just a better learning experience, but also a better automatic memory retention system.

If I could somehow redirect the time and effort spent into writing my own Obsidian notes + linking those pages + adding to my Anki decks... redirecting them into a more natural, interconnected way of encoding my learnings, into the TART system... maybe that'd be a worthwhile goal.

One thing I'm still unsure of is how I'd shift from creating cards/writing down stuff into actually making skill-based tests. I think that's a really good way to test and develop active recall, but it would also be way more complex to define what essentially amounts to a mini-game than it is to create a card you passively read from.

I think a lot of "skill tests" can be handled by preset modules, like a "free-response" module or a "click at the right coordinate on this picture" test.