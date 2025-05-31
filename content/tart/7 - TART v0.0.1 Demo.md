Once again, I've been rethinking how a hierarchical training system should be structured and how people should contribute to one. It's looking a lot more decentralized and drastically different from what I currently have.

At the same time, I can't afford to *not* release anything since part of my motivation for this project was to put something on my resume.

So as of now, the only use of my current TART code is as a playable semi-functional demo. The FSRS logic isn't actually used, it's more of just demo sessions that demonstrate question-answer feedback over a WebSocket connection with markdown & image rendering. There are no true topological orderings or spaced repetition algorithms here, and the questions are served at random under a demo curriculum about NYC. Naturally, a lot of this was vibe-coded (thanks v0!) and I've learned some limits and strengths of using LLMs for structuring and architecting projects like these.

https://tartdemo.vercel.app

![[Pasted image 20250531143838.png]]

![[Pasted image 20250531143825.png]]

![[Pasted image 20250531143819.png]]

I've made it into a resume bullet point about the technologies I've learned to use with it (I think CloudFlare workers are really cool and I'd love to do more with them). Here's some stuff that this project actually helped me grow in:

* Websockets
* Postgres + Drizzle
* systems design to some light extent
* learning about CloudFlare R2 and Workers and Workflows
* learning about MathAcademy's pedagogy
* learning about the TS-FSRS library (even though it's not implemented here)
* Restructuring how I take notes forever
* prooompting and vibe coding to some extent

---

I'll also lay out my learnings and new understanding about why this project isn't sustainable for actual use.

* The note-taking system was too rigid. Having to manually connect notes in a visual editor can get very chaotic.
![[Pasted image 20250531145049.png]]

* Additionally, making connections means I have to be aware of other notes that could possibly be related, and I just don't have the mind bandwidth for that.
* I often just copy and paste information from sources I read from, and completely ignore making actionable "challenges" from them.
* Encompassing weights are weird to manually set percentages for because of how speculative and arbitrary it is, despite what MathAcademy defines the weights to represent (some random chance) that I most certainly can't choose a number and stand behind it
* No one else is going to set this up and share their knowledge freely like this unless they know Obsidian and use my `tart-ingest` CLI tool I vibe coded to upload to a specific database.
* Everyone who's making their own hierarchical knowledge graphs is essentially walling off their own private curriculums when all human knowledge and domain expertise can mesh together in really messy ways, and I think a daily trainer that acts as your second brain should truly do away with separating "courses" or "curriculums".
* Contributing to such a hopelessly complicated and intricate knowledge system should not be manually curated or connected by a lone human.

The next project I make in this area should address all the above.