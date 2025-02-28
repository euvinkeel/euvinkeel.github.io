---
draft: true
---

Time to come up with a replication system.

I want a replication system that does not rely on the Roblox Datamodel; just JECS entities.

# Previous attempts
* previous game 1 primarily observed roblox datamodel attributes and instances. if it observed an instance created in the datamodel, it would create an entity.
	* messy, had to come up with unique attribute names and i had a huge list of them as layers upon layers on top of regular remotes, mess

* previous game 2 primarily used remotes, vanilla style. still used instances and attributes but the remotes interfaced directly with the ecs system
	* but still messy since i never used universal id system, and so i had to reimplement some chopped universal id system since i had client-spawned-and-replicated entities anyway but it was only applicable to bullets/balls and not everything else, obstacles and blocks and stuff were replicated differently....

* previous game had a universal shared id state-sync system - blindly copy state per replication tick
	* lacked important sub-tick events that happened all within a single replication tick

* previous game had a universal shared id state-sync AND comprehensive event logging, with every data change sent so the server painstakingly reinvented and replayed every client's changes to state and reconciled them
	* wayyy too slow. and desyncs happened pretty often, too. skill/implementation issue? probably.


# Requirements
* Datamodel-agnostic (do NOT rely on roblox instances or attribute replication)
* Shared entities that replicate their final state and existence onto every client.
	* I'm not going to even bother touching selective replication.
* Client-created entities that become part of the server canon.
* A way to log events with context.

To use my own terms:
* An **SID** stands for **server identifier** (or **shared identifier**) - a `u32` identifier for an entity that exists on the server and is replicated to all clients.
	* The real entity on the server's world **is equivalent** to the SID.
	* The corresponding entity on a client's world **is not equivalent** to the SID. If it is a shared entity, the SID is accessed via an `SID` component.

Obvious use cases are player and character entities that must be present in everyone's JECS worlds. Others include NPCs, enemies, coins, etc.

* A **PID** stands for **pending identifier** (or **personal identifier**) - a `u32` temporary identifier for an entity that is created on the client and is *pending approval* from the server to become replicated, hopefully being promoted to a shared entity.
	* The real entity on a client's world **is equivalent** to the CID.
	* If accepted by the server, the server will have made its own counterpart and responded with a new SID.
	* If rejected by the server, the server will include this in response and the client will despawn the pending entity.

Use cases for PIDs are mostly "optimistic updates", or the existence of an entity that needs to be changed and interacted with without waiting for the server.
* if someone wants to throw a grenade, it must spawn immediately
* if someone shoots a bullet, it must become rendered immediately

* An **action*** is a struct consisting of some event name/identifier ("fireGun", "spawnBullet", "eat") and some arguments, representing a specific mutation to be replicated on everyone's JECS state. Arguments may include basic data, structs, or *SIDs/PIDs*.
	* If the action creates an entity on the server, it returns the entity with an SID.
	* If the action creates an entity on the client, it returns the entity with a PID.

Actions are called in gameplay system code like `const newBulletEntity = actions.fireGun(gunEntity);` or `actions.broadcastServerMessaage("sup")`. Some are client-only, some are server-only. The way these are received and acted on depend on who's reading them.

* A **snapshot** is either a partial or complete representation of the final state.
	* It holds created/existing entities, component-data, and/or relation pairs recorded at a single point in time.
	* It may also contain a list of entities that were deleted. More specific to an individual client, it may contain rejected PIDs that should be despawned.
	* Players who join a new server get a *complete* snapshot, while existing players tend to get *partial* snapshots.
	* Players can also *send* a snapshot to update a specific, client-owned entity if they have permissions from the server.
		* e.g. some player-owned preferences/settings component-data, their character's positions (still monitored by the server)
	* Snapshots can be applied via some `applySnapshot` action.

# Solution
I'll use a mixture of both snapshot-based and event-based replication.
This relies on some assumptions:
* On all clients, all shared components are defined and created before they begin their replication.
* All clients get and apply a complete snapshot to fully populate their JECS world with shared entities + component-data + relations.

On a replication tick, the server looks in its big replication queue. It gets a big package from each client, containing the following:
* an action queue
* all affected SIDs
* all created/affected PIDs
* client 
Servers receive a queue of actions from each client (the order of clients doesn't matter much.) Each action represents an important change in state, and the server must verify then apply them in sequence. If a client were to have 2 bullets but called `fireGun` 3 times, the server should successfully accept & execute `fireGun` 2 times but reject the next `fireGun`. Then it should discard the rest of the actions from that client (made under the false pretense that `fireGun` was called 3 times).

This occurs for all clients who have sent

Servers must go through each individual action and mutate state accordingly.
Clients must go through each individual action and mutate state accordingly.