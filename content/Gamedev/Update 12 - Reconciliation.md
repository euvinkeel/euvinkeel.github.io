tl;dr - got reconciliation working!

everything here is a result of the following:
* replicating existence with SIDs & diffs
* client catch-up with partial diffs for eventual consistency
* populating entities with additional non-replicated components both clientside and server-side
* position + velocity updates
	* uses both unreliable events and the reliable diffing system (occasionally), both in batches
* client visual fx/reconcilation for spawning and deleting slime entities

![[RobloxStudioBeta_bGZis13oA3.gif]]

---

Most of this week was thinking about how to handle general reconciliation.

I had my first grasp on the concept of reconciliation from [this documentation page.](https://matter-ecs.github.io/matter/docs/BestPractices/Reconciliation/) Matter was my first ECS game framework, so it shaped a lot of how I view game development now. This is what I refer to when I use "reconciliation":

>Reconciliation, in this context, means taking state from one form and turning it into another. In our case, we want to reconcile our Lua state into Instances in the Data Model, so that users can see and interact with it. A key idea and benefit of reconciliation is that it's possible to reconcile the same state in multiple different ways. If we have enemies in our world at certain positions, we can reconcile them into the world with character models, but also onto a minimap with red blips. It's the same state being converted into two different ways to view the data.

I found that I had to think about both *reconciliation* and *replication*, because they have some implications about eachother.

## Replication

To replicate things like enemies or NPCs, you don't want to send huge diffs for each one. This means that you shouldn't be putting enemy type, enemy movement mode, health, status, color, appearance all on the network to be sent to the client; normally, an event would be fired to let the client know that an enemy spawned, and both the client and server infer what behaviors that specific enemy has and what it looks like.

Take our little slime here:

![[Pasted image 20250321134950.png|200]]

This slime can have a lot of aspects within a client's JECS world. It has some base speed, a color, damage/strength, position, and some target it's attacking.

To replicate each slime in a multiplayer game, you need some shared identifier to refer to a specific slime to communicate information about it: e.g. a server setting its position.

My replication system's Shared ID diff approach takes care of the shared identifier problem without ever touching the roblox datamodel. And because position + velocity can change so quickly, there are dedicated unreliable streams that the server sends & clients listen to, updating any entity with a position/velocity component.

## Reconciliation

So what about the other aspects of a Slime? They can all be inferred provided that I include some small component data about the type of enemy. Inside its `Enemy` component, I could set it to `EnemyTypeEnum.Slime` and the client and server can populate that shared entity with local component data.

If I ever needed to apply some procedural modifier, that could be defined per enemy and included within the diff so it could be populated differently (like different colors, abilities, etc)

This step has to happen in a predictable order. On the server, if it spawned a slime, it should immediately populate that shared slime entity with its own server-side components so that its other systems can act on those. On the client, if it observes a new slime from the diff, it should also populate that entity before any of its normal game systems deal with slime entities.

Because the server mutates its shared entities via action calls, we could have a server-side "hook" for any action: code that runs immediately after calling. So for the `spawnSlime` action, we'd have this:

```ts
// (these code snippets are heavily modified just to illustrate the point, so they technically aren't real snippets!)
spawnSlime: {
	execute: (spawnPosition: Vector3): LocalID => {
		const id = world.entity();
		world.set(id, cps.S_Enemy, EnemyBaseType.Slime);
		return id;
	},
	serverhook: (spawnPosition: Vector3, newId: LocalID | false) => {
		rt.world.set(entity, rt.cps.Position, spawnPosition ?? new Vector3(0, 0, 0));
		rt.world.set(entity, rt.cps.Velocity, new Vector3(0, 0, 0));
		rt.world.set(entity, rt.cps.Walker, {
			hipHeight: 3,
		});
	},
},
```

The actual shared components are denoted by `S_` inside `execute`, meaning they'll be in the diff sent to the client. However, all the inferable data will be applied via `serverhook` immediately after `execute`.

That solves the server-side issue. For the client, we can't rely on reading actions since they might join way later and miss actions; they'll be relying on diffs to catch up. So the inferable data will be applied right after applying the diff via some polling system:

```ts
// Client observes new entities with the shared component, S_Enemy
for (const [id, enemyType] of iterateNewEntitiesWith(cps.S_Enemy)) {
	print("OBSERVER: new shared enemy entity ", id);
	newBody = assets.models.slimebody.Clone();
	newBody.Parent = Workspace;
	rt.world.set(entity, rt.cps.Position, new Vector3(0, 0, 0));
	rt.world.set(entity, rt.cps.Velocity, new Vector3(0, 0, 0));
	rt.world.set(entity, rt.cps.Walker, {
		hipHeight: 3,
	});
}
```

Now, if the client were to call an action that spawns a new shared entity (given that it'll be validated by the server) it would also need its own clientside hook. However, because the server doesn't read diffs, another hook should run on the server.
So server-side actions that the client can't do are solved with a server hook and a client poller for diffs.
Either-side actions that the client can do are solved with a server hook, a client hook, and a client poller for diffs.
Client-side actions that *only* the client can do are solved with a server hook and a client hook...

now that i've written it out, that sounds like a lot.

Is there a way to define both polling and hook behavior for whenever there's an existence of an entity with some given `S_` component? That's the only problem we're solving. I don't want to write a new query loop and a server hook for every single action that spawns an enemy.

How about we use the `OnAdd` or `OnSet` hooks provided directly with JECS? I wrote it off because I didn't want to have immediate hooks when we add the `S_` component because we might need to add other `S_` components afterward, and managing the order of setting components feels icky.

But instead of setting an `OnAdd` hook on the `S_` components themselves, we could define a new class of components called `T_` components.

After the slime is completely set up within the `spawnSlime` execute function:
```ts
const id = world.entity();
world.set(id, cps.S_Enemy, EnemyBaseType.Slime);
world.set(id, cps.S_Health, {
	curr: 100,
	max: 100,
});
```

... we could just append a local `S_T_` component telling anyone who observes it to apply some specific template method by enum:

```ts
world.set(id, cps.S_T_Template, TemplateCode.Slime);
```

This way, an action called `spawnGiant` can slap on `S_Enemy` and define a different template logic than `spawnSlime` even if they share an `S_Enemy`:

```ts
world.set(id, cps.S_T_Template, TemplateCode.Giant);
```

because the real important component to look for is the `S_T_Template` component.
The template function can just look at the JECS state of the entity it's on to infer everything else, even defining client-side or server-side logic:

```ts
export const templateCodeToFunction: Record<TemplateCode, (entity: LocalID) => void> = {
	[TemplateCode.Slime]: (entity: LocalID) => {
		if (IS_CLIENT) {
			print("Clientside template logic here");
		} else {
			print("Serverside template logic here");
		}
		rt.world.set(entity, rt.cps.Position, new Vector3(0, 0, 0));
		rt.world.set(entity, rt.cps.Velocity, new Vector3(0, 0, 0));
		rt.world.set(entity, rt.cps.Walker, {
			hipHeight: 3,
		});
	}
}
```

So whether something is spawned via server-side action call, client-side action call, server-side action replay, or client-side applying a diff, the presence of a `S_T_Template` will always ensure an entity gets populated before other game systems can react/edit them.

And it works pretty well!

Here's what happens when a ton of enemies are already spawned in when the player joins: the initial handshake is intentionally delayed, but the client still catches up with the server and handles local population + creation animations + death particle animations.

![[RobloxStudioBeta_J74QME5w3v.gif]]


Visually, I don't have anything other than this. Hopefully these systems should be a good speed multiplier in the future as I focus more on my own developer experience.
