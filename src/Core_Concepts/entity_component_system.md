# Entity Component System
For general information on the ECS architecture, we recommend looking at [Wikipedia](https://en.wikipedia.org/wiki/Entity_component_system).

This article contains information specific to ChatImproVR's ECS, and some of the specific features it has.

## Entities
Entities in ChatImproVR are **universally** unique IDs. As of this writing, this isn't necessarily true in practice, but entities are intended to be treated this way. The point of making universally unique entity IDs is that entities can be transferred client and server, and even between servers without having to worry about collisions.

## Components
Components in ChatImproVR are identified by their unique ID and by their (maximum) size. This unique ID corresponds to one and **only one** data schema. If the component data type changes, one must also change the ID of the component to avoid corrupting data for potentially reliant third party plugins. One of the less ergonomic aspects of ChatImproVR's current implementation is that components are fixed-size. This means that **a component must serialize to a data length less than or equal to the size prescribed in its ID**. The reason for this is that it makes manipulating components in memory much easier, and necessitates the association of a data type with its (fixed) size.

## Systems
Systems in ChatImproVR are contained within WebAssembly plugins. Each system corresponds to exactly one ECS query, so that one sets up a System by specifying which components it is associated with. Systems are executed in stages; the `Init` stage is called when a plugin is initialized, and each frame the `PreUpdate`, `Update`, and `PostUpdate` stages are executed in sequence. 
