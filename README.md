# A collection of features feather needs

I started this document 2021-May-9 when the current head of feather-rs was the commit [b63fd95](https://github.com/feather-rs/feather/commit/b63fd95da0170eaa89f7241f0749bea0987262f8). This document is written to help people discover what features they can help implement, and to show what features are missing for them to be implemented. This is not a complete lit of what needs to be done, but a starting point for newcomers. 

*NOTE* Before you start implementing these features you should open a issue on GitHub, and roughly outline what you plan on doing. That way you don't waste your time. The discord is also a way to get in contact. 

If we look at [feather/server/src/packet_handlers.rs](https://github.com/feather-rs/feather/blob/b63fd95da0170eaa89f7241f0749bea0987262f8/feather/server/src/packet_handlers.rs#L69) we see that there are 33 packets the feather server does not handle. 

-------------------
## TabComplete

I am working on this.

-------------------
## ClientPlayPacket::TeleportConfirm
https://wiki.vg/Protocol#Teleport_Confirm

As you can see the documentation for this packet is somewhat lacking. It just says the client should respond to a client bound "Player Position And Look" by sending this packet. It does not specify what should happen on failure to do so. It would be helpful if someone could figure out if we need to do anything with this packet.

-------------------
## ClientPlayPacket::QueryBlockNbt(_)

When a player presses Shift+F3+I the overview specifies what 
block the players is looking at. This is done by sending this request to the server. The client specifies a xyz of a block
it would like to query. 

To implement this feature properly we would have to check for line of sight, else a hacked client could use this feature for x-ray and player radar. The minimal viable solution is to just check that the block in question is within a lets say 5 block radius of the caller.

Allowing a client to query any arbitrary position is probably a bad idea anyways, because timing of the response could tell if a chunk is loaded or not. Therefor we should never allow someone to call this on a position further away then max render distance. 

The file [feather/common/src/world.rs:122](https://github.com/feather-rs/feather/blob/b63fd95da0170eaa89f7241f0749bea0987262f8/feather/common/src/world.rs#L122) has a function for querying the world about a block position. And it is accessible through the variable [game](https://github.com/feather-rs/feather/blob/b63fd95da0170eaa89f7241f0749bea0987262f8/feather/server/src/packet_handlers.rs#L25).

-------------------
## SetDifficulty
This packet is only used in singleplayer as far as [wiki.vg](https://wiki.vg/Protocol#Set_Difficulty) knows. If this is wrong then open a issue on github, and give the reason you believe this is not correct. 

-------------------
## ClientStatus
This packet has two variants [wiki](https://wiki.vg/Protocol#Client_Status). It is sent when the player presses spawn after death, and when a player enters settings menue. 

In my opinion (Miro in Discord) this should trigger an [event](https://github.com/feather-rs/feather/blob/main/docs/architecture.md#events). Ask around before implementing it. See [feather/common/src/events.rs](https://github.com/feather-rs/feather/blob/main/feather/common/src/events.rs) for how to implement an event. 

-------------------
## Windowing

There are a few window related packets. I (Miro) am not certain about how close the existing solution is to handle these packets. The current implementation for window handling can be found in [feather/common/src/window.rs](https://github.com/feather-rs/feather/blob/b63fd95da0170eaa89f7241f0749bea0987262f8/feather/common/src/window.rs#L1) 

We probably want plugins to be able to define window interactions. We therefor need to implement some sort of system for that. Ask in the discord for ideas on how to implement this feature. This is probably not a straight forward task.

Amber has currently a open pullrequest that on first glance looks related to this: [link](https://github.com/feather-rs/feather/pull/382)

#### WindowConfirmation
[wiki](https://wiki.vg/Protocol#Window_Confirmation_.28serverbound.29)

#### ClickWindowButton
[wiki](https://wiki.vg/Protocol#Click_Window_Button)

#### CloseWindow
[wiki](https://wiki.vg/Protocol#Close_Window_.28serverbound.29)

-------------------
## PluginMessage
[wiki](https://wiki.vg/Protocol#Plugin_Message_.28serverbound.29)

-------------------
## EditBook
Book handling is a tricky issue when it comes to minecraft servers as there have been several book related issues. [YouTube]("https://www.youtube.com/watch?v=JDGx8zeCsUA"). 

I would keep an eye on the function "handle_player_block_placement" in [/feather/server/src/packethandlers/interaction line 17]("https://github.com/feather-rs/feather/blob/b63fd95da0170eaa89f7241f0749bea0987262f8/feather/server/src/packet_handlers/interaction.rs#L17") Once it is properly implemented it would be possible to somewhat copy that code. 

The inventory system is currently being updated, and this would depend on that. 

-------------------
## QueryEntityNbt
Used when Shift+F3+I is pressed while looking at an entity. 
[wiki](https://wiki.vg/Protocol#Query_Entity_NBT)

I believe that this feature is somewhat cumbersome to implement, because feather stores entity related metadata as components in the ECS. You would have to implement a
custom "to nbt" for every entity type. 

-------------------
## GenerateStructure
[wiki](https://wiki.vg/Protocol#Generate_Structure)
This looks like a fun thing to implement, but not currently in the scope of feather. This feature would enable [jigsaw blocks](https://minecraft.fandom.com/wiki/Jigsaw_Block). 


-------------------
## KeepAlive
[wiki](https://wiki.vg/Protocol#Keep_Alive_.28serverbound.29) 
This is a simple feature one could implement for feather. It is used to check for dead connections were the client is not responding. The old version of feather used to have this implemented, but that code was lost in the recent refactoring/transition to 1.16. The old code can be found in "feather/old/".

-------------------
## LockDifficulty
Same as SetDifficulty, so if you believe that wiki.vg is wrong ans this packet is not only used in singleplayer, then open a issue on github.

-------------------
## VehicleMove
The [wiki.vg](https://wiki.vg/Protocol#Vehicle_Move_.28serverbound.29) is not very specific about what constitutes a vehicle. A helpful check would be for someone to figure out in what circumstances this packet is sent. Does it happen in more then just minecarts and boats? What happens when someone rides a horse or a pig? 

To implement this feature we need working physics. [Issue](https://github.com/feather-rs/feather/issues/357). 

-------------------
## SteerBoat
[Wiki](https://wiki.vg/Protocol#Steer_Boat)

This should trigger a "Entity Animation (clientbound)" packet to be sent to nearby clients. 


@TODO


## PickItem

[wiki](https://wiki.vg/Protocol#Pick_Item)
```
Used to swap out an empty space on the hotbar with the item in the given inventory slot. The Notchain client uses this for pick block functionality (middle click) to retrieve items from the inventory. 
```

To implement this feature we need the inventory system to be finished. 


## CraftRecipeRequest
[Wiki](https://wiki.vg/Protocol#Craft_Recipe_Request)
```
This packet is sent when a player clicks a recipe in the crafting book that is craftable (white border). 
```
To implement this feature we need to have a list of every Minecraft recipe. We also want this list to be viewable and editable by plugins.

As far as i (Miro) know feather currently does not have any knowledge of recipes. And implementing this feature would require multiple steps.

#### a) Add recipes to libcraft 
In [/libcraft](https://github.com/feather-rs/feather/tree/main/libcraft) we have definitions on minecraft related data. This is probably were we want to add a definition containing all vanilla recipes. (Ask someone on discord to make sure). 

[minecraft-data](https://github.com/PrismarineJS/minecraft-data) is a awesome project that collects minecraft related data. They keep up to date sources for stuff like recipes. You would have to implement a rust macro that converts the json into rust structs. 
This is somewhat difficult for beginners, as macros are somewhat tricky. 

#### b) Add a recipe component to quill. 
Quill defines how the server interacts with plugins. Currently plugins must be written in rust, but in the future we hope to make plugins writeable in any language that compiles too WebAssembly. Therefor the plugin api is a C like interface. 


## PlayerAbilities
[wiki](https://wiki.vg/Protocol#Player_Abilities_.28serverbound.29)
The vanilla client sends this packet when the player starts/stops flying.

To implement this here is what i believe you need to do. Remember to open a github issue.
I might have it wrong in what order things should happen. Maybe the event should happen first.  

#### a)
Create a IsCreativeFlying component in [quill](https://github.com/feather-rs/feather/blob/b63fd95da0170eaa89f7241f0749bea0987262f8/quill/common/src/components.rs). 

#### b)
Create a is_creative_flying system in [feather/server/src/systems](https://github.com/feather-rs/feather/tree/main/feather/server/src/systems) that changes the IsCreaiveFlying component on players that send the packet if they are in creative mode, else they are kicked. 

#### c)
Maybe we want to trigger an event when this value changes. Its almost the same steps as a and b, but you should look at [quill/common/events](https://github.com/feather-rs/feather/tree/main/quill/common/src/events)

And for an example on how to trigger the event go to [feather/server/src/packet_handlers/interaction.rs](https://github.com/feather-rs/feather/blob/b63fd95da0170eaa89f7241f0749bea0987262f8/feather/server/src/packet_handlers/interaction.rs#L89), or go directly to the [method](https://github.com/feather-rs/feather/blob/b63fd95da0170eaa89f7241f0749bea0987262f8/feather/ecs/src/event.rs#L39). 
Note however that the main maintainer Caelum, is
[rewriting the ecs](https://github.com/feather-rs/feather/issues/402), so the method might move to a different file, but the function probably will be the same.

---------------------
## EntityAction
Very Similar to PlayerAbilities, but more cases to handle. 
```
0 	Start sneaking
1 	Stop sneaking
2 	Leave bed
3 	Start sprinting
4 	Stop sprinting
5 	Start jump with horse
6 	Stop jump with horse
7 	Open horse inventory
8 	Start flying with elytra 
```
-----------

# SteerVehicle
A good start to implement this feature is to do the same as for PlayerAbilities and EntityAction. Just add a event that is triggered for a player entity (as in ecs) when this packet is received. 

-------------
# SetDisplayedRecipe
I don't understand the [wikis](https://wiki.vg/Protocol#Set_Displayed_Recipe) description. Some clairification is probably a good start. 

---------------
# SetRecipeBookState
I don't understand the [wikis](https://wiki.vg/Protocol#Set_Displayed_Recipe) description. Some clairification is probably a good start. 

-----------------------

# NameItem
[wiki](https://wiki.vg/Protocol#Name_Item)
```
Sent as a player is renaming an item in an anvil (each keypress in the anvil UI sends a new Name Item packet). If the new name is empty, then the item loses its custom name (this is different from setting the custom name to the normal name of the item). The item name may be no longer than 35 characters long, and if it is longer than that, then the rename is silently ignored. 
```

To implement this 

-----------------------------

# ResourcePackStatus

@todo

--------------------

# AdvancementTab
This packet tells if advancement tab has been opened by the client. A rudimentary implementation of this feature is to do the same as for PlayerAbilities and just trigger a event for it.
Most likely the sever is always just going to ignore it anyways.

---------------
# SelectTrade
```
When a player selects a specific trade offered by a villager NPC. 
```
This feature also depends on the window system that is undergoing change. Ask on the discord is you want this feature implemented.

---------------------

# SetBeaconEffect
This packet does not contain any information about what beacon its talking about. My guess is that it must be inferred by the window that the client currently has open. This feature therefor depends on us knowing what window a player is currently looking at. I don't know how to do this atm, so you should ask on the discord. 

----------------------------

# UpdateCommandBlock
@TODO


-----------------------

# UpdateCommandBlockMinecart
@TODO


-----------------------
# UpdateJigsawBlock
@TODO

-------------------------
# UpdateStructureBlock
@TODO

---------------------------
# UpdateSign
@TODO

-----------------------------
# Spectate
```
Teleports the player to the given entity. The player must be in spectator mode. 
```
-----------------------------------

# UseItem

[wiki](https://wiki.vg/Protocol#Use_Item)
To implement this feature we need a working inventory system. 
To implement this we need to do the same basic thing as for 
PlayerAbilities. This should also just trigger a event.
And then we implement systems that react on those events. 

-----------------------------


