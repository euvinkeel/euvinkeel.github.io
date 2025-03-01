---
draft: false
---
Time to come up with a replication system.
All of the below will probably not make much sense, nor will it use industry standard terms. This is more for myself.

# Requirements
I want a replication system that does not rely on the Roblox Datamodel; just JECS entities.
* Datamodel-agnostic (do NOT rely on roblox instances or attribute replication as a source of truth)
* Shared entities that replicate their final state and existence onto every client.
	* I'm not going to even bother touching selective replication.
* Client-created entities that become part of the server canon.
* A way to log events with context.

# Previous attempts
* previous game 1 primarily observed roblox datamodel attributes and instances. if it observed an instance created in the datamodel, it would create an entity.
	* messy, had to come up with unique attribute names and i had a huge list of them as layers upon layers on top of regular remotes, mess

* previous game 2 primarily used remotes, vanilla style. still used instances and attributes but the remotes interfaced directly with the ecs system
	* but still messy since i never used universal id system, and so i had to reimplement some chopped universal id system since i had client-spawned-and-replicated entities anyway but it was only applicable to bullets/balls and not everything else, obstacles and blocks and stuff were replicated differently....

* previous game had a universal shared id state-sync system - blindly copy state per replication tick
	* lacked important sub-tick events that happened all within a single replication tick

* previous game had a universal shared id state-sync AND comprehensive event logging, with every data change sent so the server painstakingly reinvented and replayed every client's changes to state and reconciled them
	* wayyy too slow. and desyncs happened pretty often, too. skill/implementation issue? probably.

# Terms
To use my own terms:
## SID
* An **SID** stands for **server identifier** (or **shared identifier**) - a `i32` identifier for an entity that exists on the server and is replicated to all clients.
	* The real entity on the server's world **is equivalent** to the positive SID, giving us `2147483647` possible SIDs.
	* The corresponding entity on a client's world **is not equivalent** to the SID. If it is a shared entity, the SID is accessed via an `SID` component.

Obvious use cases are player and character entities that must be present in everyone's JECS worlds. Others include NPCs, enemies, coins, etc.

## PID
* A **PID** stands for **pending identifier** (or **personal identifier**) - a `i32` temporary identifier for an entity that is created on the client and is *pending approval* from the server to become replicated, hopefully being promoted to a shared entity.
	* The real entity on a client's world **is equivalent** to the negative PID, giving us `2147483648` possible PIDs.
	* If accepted by the server, the server will have made its own counterpart and responded with a new SID.
	* If rejected by the server, the server will include this in response and the client will despawn the pending entity.

Use cases for PIDs are mostly "optimistic updates", or the existence of an entity that needs to be changed and interacted with without waiting for the server.
* if someone wants to throw a grenade, it must spawn immediately
* if someone shoots a bullet, it must become rendered immediately

## Action
* An **action*** can refer to:
	* its **owner**: the player entity that sent the action. If it's 0, it originated from the server. Or maybe the server can have its own SID. Who knows
	* its **struct representation**: its name, owner SID, and the arguments associated with it (can be basic data or refer to other SIDS/PIDS)
	* its **mutation**: a callback to execute a semantic mutation in shared JECS state (as opposed to a non-semantic snapshot), stored in a lookup module
		* If the action creates entities on the server, calling it should return new SIDs.
		* If the action creates entities on the client, calling it should return the new local entity & new PIDs.
		* a created entity on the server must return the original SID
		* but on the client it must return the PID and the local ID...
		* for the client, they should be able to pass in 
			* the SID must be negative to indicate it is not the local id
			* the PID must be positive
			* clients can just pass in the PID as normal along with their other regular local ids, and the SID must be translated in the background by the engine running mutate: this must be 
	* its **validation**: code that looks at the JECS world state and determines if this action mutation is allowed to happen, also stored in a lookup module
	* its **event**: whenever the action is called locally or received from the outside, it should emit an event (passing out the struct) so game systems can hook onto each call regardless of where the event came from

In both client and server code, action **mutations** may be called in gameplay systems like

`const newBulletEntity = actions.fireGun(gunEntity, position, direction);`
`actions.broadcastServerMessaage("sup")`
`actions.kickPlayer(playerEntity, "stop hacking bro")`

## Snapshot
* A **snapshot** is either a partial or complete representation of some JECS state at a single point in time.
	* It holds **created/existing entities**, **component-data**, and/or **relation pairs**
	* It may also contain a list of entities that were **deleted**.
	* More specific to an individual client, it may also contain **rejected PIDs** that should be despawned.
	* Players who join a new server get a *complete* snapshot, while existing players tend to get *partial* snapshots.
	* Players can also send a snapshot to update a specific, client-owned entity if they have permissions from the server.
		* e.g. some player-owned preferences/settings component-data, their character's positions (still monitored by the server)
* A **snapshot** can also refer to the **action** that applies the snapshot data (say, `applySnapshot`.

# Solution
I'll use a mixture of both snapshot-based and event-based replication.
Let's map out what happens on the client when receiving + sending, and what happens on the server for receiving + sending.

# Possible Failures
* When a player joins, what happens if the shared state changes way too quickly, so the outgoing queue is overfilled when the client is finally done and gets the DONE signal?
	* The server can clear the outgoing queue, raise an error for systems to handle (maybe it cleans up a ton of stuff and logs something to analytics), and simply tries sending a complete snapshot again of the hopefully cleaner world.
* When a player accumulates too many incoming events and snapshots in time, what happens?
	* get a faster computer lol
	* they're kicked for being too poor

I should probably have warning signs and gauges for these things.

## Client
### (Init)
Before any gameplay systems run, new players perform a handshake with the server to let it know when it's ready to receive the initial snapshot.
The player will begin all *replication* systems but not run any other gameplay systems, and starts sending the server a "go-ahead" signal (intermittently, with exponential-ish backoff)
Upon the go-ahead, the server will generate a snapshot of the entire world and send it to the player.
The player's replication systems should build the game state (it could be batched or gradual, think about that later.)

As all of this happens, the server may be calling actions as usual - all the enqueued actions (and enqueued partial snapshots) will start piling up for the new player.

The server sends the player a DONE signal, and sends them all the enqueued actions left in their queue.

The client receives the DONE signal, starts its gameplay systems, and begins the regular replication process below.

### Client Receiving
On a client's replication tick, the client looks in its incoming queue.
That queue has some packages.
Each package contains:
* PID rejections, `RejectedPIDs`
* PID approvals `ApprovedPIDs` and the new SID they've been assigned
* a list of action structs from the server
For every package:
* Loop through all PID rejections and delete those entities.
* Loop through all PID approvals, attach their official SID. Future actions will now refer to them by their SID.
* Loop through all action structs:
	* For every action struct received, clients will fire the **event** for that action but **not mutate** the JECS state.
	* (exception: If that action was a snapshot action, the clients will **mutate** the JECS state.)

An important note is that event structs may contain entity arguments that no longer exist; these cases should be easy to handle gracefully because meaningful shared mutation should not be handled here.
Actual mutation of state happens by consuming snapshots, not by replaying a ton of actions.

After this, the client runs its gameplay systems which may call more actions, enqueueing them & accumulating all before the client's next replication tick.

### Client Gameplay
Each gameplay system can call an action. Upon calling an action:
* the **event** is fired
* the **struct** is enqueued
* any affected entities are added to the `AffectedSIDs` list, no order needed.
* any created PID entities are added to the `CreatedPIDs` list *in order*.
	* the order must be chronological since the server must match the PID with future references to that PID

### Client Sending
On a client's replication tick, the client looks in its outgoing queue of **structs**.
The client sends out the following together:
* The list of structs in order
* `AffectedSIDs`
* `AffectedPIDs`

## Server

### (Init)
For all existing and future players...
* it immediately starts listening for a go-ahead signal
* it concurrently loads save data
* Upon GO-AHEAD and successfully loading data, it:
	* makes a snapshot of the entire world and starts sending that to the player, batched or not
	* creates an outgoing queue (that won't work yet), accumulating actions and more partial snapshots while the above is happening
	* creates an incoming queue for that player
	* creates a player's personal "snapshot" that will accumulate all changes in the world for that specific player, and will be continually sent -> cleared -> built-up and sent again after each action list
* When the entire snapshot has been sent out,
	* it creates the shared player entity in its JECS world, officially recognizing it as a player ready to participate in the game
	* it flips the switch to start sending out everything in the outgoing queue next replication tick

## Server Receiving
On the server's replication tick, the server looks through each one of its incoming queues for each player, in any order
Each player's queue has the following packaged together:
* a list of action structs
* `AffectedSIDs`
* `CreatedPIDs`
An internal index cursor starts at 0 at `CreatedPIDs`.
Iterating through every item in that queue at that moment in time:
* The server pops out the action **struct**.
* The server calls the **validator** for that action, seeing if it's allowed considering the **owner** attached to the struct.
* If approved, the server...
	* Calls the **mutator** for that action.
		* If that mutator returned SIDs, map the new SID to the next PID in `ApprovedPIDs`, increment the index cursor up.
			* From now on, if the PID is encountered, mutations should operate on the matching SID.
		* If the above fails, log error & **treat as rejection**.
	* Fires the **event** for its own gameplay systems.
	* Enqueues the **struct** for every player's outgoing queue (other than the owner.)
* If rejected, the server...
	* Clears the entire incoming player queue.
	* Adds everything after the `CreatedPIDs` cursor to `RejectedPIDs`.
	* Merges the current snapshot of all entities in `AffectedSIDs` to send to the offending player's personal merge snapshot.
	* Continues to the next player's package queue.

### Server Gameplay
Each gameplay system can call an action. Upon calling an action:
* the **event** is fired
* the **struct** is enqueued
* any affected entities are merged into every client's personal snapshot

### Server Sending
On the server's replication tick, the server looks through each one of its outgoing queues for each player, in any order.
The server converts the personal snapshot for every player into an `applySnapshot` action, and enqueues it into the action struct queue.
Per player, the following is sent out:
* `ApprovedPIDs`
* `RejectedPIDs`
* The list of action **structs**


# Next Steps

okay, so that's a lot of system stuff that i iterated on a bit
i need to figure out actionable next steps, maybe even draw a diagram. who knows