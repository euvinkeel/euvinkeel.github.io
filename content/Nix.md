---
title: Nix
draft: true
---


----
# links

https://news.ycombinator.com/item?id=42764762
From https://news.ycombinator.com/item?id=42766316:
	> My policy is to never let pipeline DSLs contain any actual logic outside orchestration for the task, relying solely on one-liner build or test commands. If the task is more complicated than a one-liner, make a script for it in the repo to make it a one-liner. Doesn't matter if it's GitHub Actions, Jenkins, Azure DevOps (which has super cursed yaml), etc.

Then, I found out about [this blogpost](https://determinate.systems/posts/nix-github-actions/) which detailed how using Nix inside an action can essentially solve the problems of including/installing dependencies instead of relying on picking the right github action *and* easily run this locally because of Nix's reproducibility.