---
title: Hardpoint System
category: design-docs
---

# Hardpoint-based Equipment System

**Updated:** 2024-09

A major aspect of the gameplay vision for Pioneer is to move away from the mass-limited equipment system, replacing it with a volume-based capacity metric that provides more flexibility when deciding how to outfit your ship. Another aspect is to provide the player with visible feedback about the state of their ship, including rendering visible weapons in the correct locations on the hull.

When considering the physical volume and placement of equipment, it becomes apparent that some equipment is purely internal, with the assumption that it can be mounted anywhere; other equipment (like ­hyperdrives and shield generators) generally must be mounted in specific areas[^1] and have a 'slot limit' on how many can be installed; and some equipment (weapons, sensors, and scoops) need to be mounted on the exterior of the craft to function.

To manage this distinction, we plan to use a **hardpoint-based model** for external equipment.

Hardpoints are defined by the ship designer and associated with a physical point on the craft's exterior, with an associated type[^2] that controls what items can be mounted in that hardpoint. To simplify authoring of content and ensure item compatibility between ships without clipping, hardpoints are grouped into specific size classes which define spatial constraints on both the ship model and the equipment item model.

All equipment consumes some amount of internal capacity (for mounting, power couplings, ammunition storage / fuel processing, etc.) but items that mount in hardpoints consume drastically less due to most of their volume being outside the craft, or being stored in a dedicated space inside the hull.

In the future, we plan to add some additional properties to hardpoints controlling things like power draw, data bandwidth (for targeting or sensors), and heat dissipation. These properties will provide more levers for tweaking ship balance and allow each ship type to fit a specific gameplay role.

Slot-based internal equipment uses a slightly more relaxed model; slot groups are defined by a size and count, but the details of which individual slot contains the item are abstracted away internally by the equipment system. Internally-mounted equipment items allocate their volume from a common internal capacity pool, and an empty slot consumes no volume.

Finally, certain equipment items may be "slot-less"; they could be mounted anywhere in the ship provided there is enough equipment capacity to hold them. Cargo bays are the primary example; it may be possible to convert a ship's equipment space directly into cargo space by installing the requisite number of cargo bays.[^3]

[^1]: Specific primary equipment items that participate in physicalized damage must be represented by colliders authored in the ship model. These colliders are associated with a specific slot identifier (e.g. hyperdrive-1 or shield-3).

[^2]: Hardpoints may allow more than one type of item to be mounted, though no more than a single item can be mounted at a time. Additionally, hardpoints designated this way must meet the size and clearance constraints of all item types it supports.

[^3]: This is still under discussion; another proposed solution is to have a "ship refitting" process that converts equipment capacity into cargo space, or to define ships with a fixed number of cargo bays that have additional equipment mounted in them.

## Weapon Hardpoints

Weapons here refer to fixed or gimballed mounts for gun-style weapons (e.g. lasers, ballistics, etc.); missiles are handled separately, and turrets are handled via a separate hardpoint type.

Weapons are best used for combat within 1-5km, depending on projectile velocity and target aspect ratio.[^4]

- Weapon hardpoint sizes increment by ~2.75x volume and ~1.4 uniform scale
  - S1: 1.00 scale
  - S2: 1.40 scale
  - S3: 1.96 scale
  - S4: 2.75 scale
  - S5: 3.85 scale
- The overall effectiveness of a weapon scales by 1.6x per size increment. This can include power draw, sustained damage per second, etc.
- Weapons are slightly more power-efficient as the size class goes up (e.g. 1x S3 weapon draws ~2.4x the power of an S1 instead of 2.56x)
- Different weapon types in the same class can have different DPS, but need to be balanced by heat build-up and power draw
- Potentially sensor emissions? Firing "noisy" weapons make it easier for missiles to track...

[^4]: Target aspect ratio is a categorization of target's velocity relative to the attacker; targets that are approaching are said to be "hot", targets moving perpendicular to the attacker's velocity are "beaming", and targets flying away from the attacker are "cold".

## Missile Hardpoints

Missiles are the primary means of combat beyond the 1-2km gun range. They can be mounted directly on appropriately-tagged hardpoints, or mounted in racks or pods.

A hardpoint can specify that it is able to mount a missile, an ordnance rack, an ordnance pod, or any combination of the above. Ordnance racks and pods occupy the primary hardpoint, and provide a 'slot group' that can mount missiles of a specific size. Support for mixing different types of missiles in the same rack is an optional feature and most likely will not be targeted initially.

- Missiles templates increment volume by 2.6x (1.375 uniform scale) 
- There needs to be a tradeoff between running multiple small missiles vs. a single large missile:
  - Total missile damage / DPS scales by 1.8x per size increment
  - Missile armor penetration / effective impact scales by 2.1x per size increment
  - Larger missiles are more effective against large/hard targets
  - Smaller missiles are better for engaging agile/unarmored targets
- Missiles do not scale as large as weapon hardpoints do; instead "volume of fire" is utilized
- Missile size classes:
	- S0: 1.1m, 90mm diameter; unguided rockets / A2G guided rockets
	- S1: 1.9m, 180mm diameter; short-range guided missiles
	- S2: 2.6m, 250mm diameter; medium-range guided missiles
	- S3: 3.6m, 340mm diameter; long-range smart missiles
	- S4: 5.0m, 470mm diameter; "naval" missiles / torpedoes
	- S5: 6.8m, 640mm diameter; large anti-capital missiles

- Missiles are mounted in "missile" hardpoints, typically externally. A hull configuration can directly provide missile hardpoints, or it can allow flexibility by providing pylon or missile bay hardpoints.

- Pylon hardpoints are external hardpoints that mount external ordnance launchers. A pylon can mount a launcher for a single missile of its size class, or a missile rack containing several smaller missiles. Pylons have two templates, a larger general-purpose template which allows the use of single-missile launchers and large ordnance pods, and a smaller template only compatible with standardized missile racks.

- Pylon hardpoints scale according to the same metric as weapon hardpoints (this is intentionally slightly greater than scaling of individual missiles):
  - S1: 1.00 scale
  - S2: 1.40 scale
  - S3: 1.96 scale
  - S4: 2.75 scale
  - S5: 3.85 scale

- Missile racks use power-of-two scaling (2/4/8) and start with 2 missiles of size N-1. They fit into the "pylon.rack" subset of the pylon template.

- Rocket pods are special weapons which must mount on a pylon hardpoint. They are mostly used to hold an odd number of S0 dumb-fire rockets for ground-attack work. S0 rockets are not tracked as missiles directly through the equipment system, but are considered "ammunition" for the launcher pod "weapon".

- Pylon hardpoints using the larger general-purpose template may hold a single missile in a missile rack of its size class in addition to being able to hold an ordnance rack / pod. This requires the hardpoint area conform to the weapon template sizing for an equivalently-sized weapon as well as the ordnance rack template.

- Missile bay hardpoints are separate equipment types. They are intended for manufacturer- and hull-specific missile launchers mounted in dedicated equipment volume inside of the hull. Missile bays typically mount several missiles of the same size class and generally hold more missiles than a standardized missile rack of equivalent size, at the cost of being restricted to a single hull or manufacturer.

## Internal Equipment

Most internal equipment falls into one of three categories:

- Freely configurable Equipment
- Slot-mounted Equipment
- Computer Modules

Freely configurable equipment is the simplest type of equipment. All equipment items define a volume they consume when mounted, and an amount of mass they add to the ship's weight. A freely-configurable equipment item has no special requirements for placement or connections, and other than power, system bandwidth (when implemented), or specific module-type limits, has no restrictions as to how many can be mounted. Any size of this type of equipment can be mounted in the hull, provided there is enough equipment volume to fit it.

Slot-mounted equipment adds an extra restriction to freely configurable equipment. In addition to consuming equipment volume, power, and system bandwidth, slot-mounted equipment must have specific considerations made as to its placement and connections to the hull. Each hull specifies a list of equipment slots which specify the type of equipment, the size(s) that can be mounted, and how many slots are present.

An example of this type of equipment is a hyperdrive or shield generator - the former requires a close connection to the engines, and the latter needs to be connected to emitters on the hull of the ship. The size of an equipment slot hides away other considerations which would be too annoying to track - e.g. the maximum througput of the shield emitters, or the hyperdrive.

Computer modules are a special type of equipment item - they consume no equipment volume and are direct "upgrades" of the ship's capability. In return, they're mounted into a limited number of slots in the ship's computer, and consume a proportionate amount of system bandwidth. Ship computers are specified on a per-hull basis and provide a number of Small, Medium, and Large module slots. The ship's computer cannot be easily replaced without significant modification to the ship.

## Cabin Slots

Cabins are a special-case equipment item, consuming no "equipment space" volume but being assigned pre-planned slots by the ship designer. A cabin slot can hold several different types of "plugs": extra crew accommodations, passenger bays, workshop areas for in-flight repair of ship components, or even an "empty bay" item which has very low mass cost but no benefits of its own. Perhaps enterprising captains could even purchase "decoy" cabins which hide a smuggler's hold behind a seemingly-innocuous blank wall?

The actual dimensions of a cabin slot may vary from ship to ship, but the volume allocated to a cabin slot should be approximately the same across all slots of the same size:

- **S1**: 12.5m³ - 1 person
- **S2**: 37.5m³-40.0m³ - 3 people
- **S3**: 120m³-128m³ - 8 people
- **S4**: 375m³-410m³ - 22 people
- **S5**: 1125m³-1320m³ - 60 people

## Hull Variants

Ship hulls are associated with a set of metadata called a "hull configuration". That configuration defines hardpoints, internal slots, equipment space, cargo space, fuel tank size, and more.

Variants of the same hull can exist as additional "hull configurations" of the same base hull. Players can either buy the specific variant of the ship they want, or can potentially buy a different variant of the same hull and pay a mechanic to retrofit the ship into the specific configuration they want.

For example, a hull configuration might remove a ship's internal missile bay but add an additional external weapon hardpoint, reconfigure the in-system engines to make room for a larger hyperdrive, or replace a bulk freighter's cargo hold with fighter launch bays.

Access to ship variant "blueprints" can be a gameplay element, with certain hull configurations (for example, a search-and-rescue configuration of a cargo ship) being rare and less likely to be available for sale at the ship market.

Hull variants are expected to be predefined on a per-ship basis, rather than freely generated at runtime. The internal equipment system already provides a large amount of customization potential, and hull variants are intended to augment that system to handle customizations that would require major changes to the ship's external mesh.

Hull Variants are intended as a future expansion to the equipment system.
