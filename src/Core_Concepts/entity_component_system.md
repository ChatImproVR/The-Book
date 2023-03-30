# Entity Component System
For general information on the ECS architecture, we recommend looking at [Wikipedia](https://en.wikipedia.org/wiki/Entity_component_system).

This article contains information specific to ChatImproVR's ECS, and some of the specific features it has.

## Entities
Entities in ChatImproVR are **universally** unique IDs. As of this writing, this isn't necessarily true in practice, but entities are intended to be treated this way. The point of making universally unique entity IDs is that entities can be transferred client and server, and even between servers without having to worry about collisions. 

Entities are created and deleted plugin-side like so:
```rust
let my_entity = io.create_entity()
    .add_component(Transform::identity())
    .build();
io.remove_entity(my_entity);
```

## Components
Components in ChatImproVR are identified by their unique ID and by their (maximum) size. This unique ID corresponds to one and **only one** data schema. If the component data type changes, one must also change the ID of the component to avoid corrupting data for potentially reliant third party plugins. One of the less ergonomic aspects of ChatImproVR's current implementation is that components are fixed-size. This means that **a component must serialize to a data length less than or equal to the size prescribed in its ID**. The reason for this is that it makes manipulating components in memory much easier, and necessitates the association of a data type with its (fixed) size.

New components data types are declared like so:
```rust
/// Component datatype
/// Implements Serialize and Deserialize, making it compatible with the Component trait.
#[derive(Serialize, Deserialize, Clone, Copy, Debug)]
struct MyComponent {
    a: i32,
    b: f32,
}

impl Component for MyComponent {
    // Here we define the universally unique name for this component.
    // Note that this macro simply concatenates the package name with the name you provide.
    // We could have written "channels_example/MyMessage" or even "jdasjdlfkjasdjfk" instead.
    // It's important to make sure your package name is UNIQUE if you use this macro.
    const ID: &'static str = pkg_namespace!("MyComponent");
}
```

**We can take a shortcut!** The `Component` derive macro does this all automatically!
```rust
#[derive(Component, Serialize, Deserialize, Clone, Copy, Debug)]
struct MyComponent {
    a: i32,
    b: f32,
}
```

Components may be added to an entity plugin-side:
```rust
io.add_component(ent, &MyComponent { a: 0, b: 0.0 });
```

Components on an existing entity may be modified by calling the function above, or by modifying them within a query:
```rust
for entity in query.iter() {
    query.write(
        entity,
        &MyComponent {
            a: 1.,
            b: 2312.0,
        },
    );
}
```

## Systems
Systems in ChatImproVR are contained within WebAssembly plugins. Each system corresponds to exactly one ECS query, so that one sets up a System by specifying which components it is associated with. Systems are executed in stages; the `Init` stage is called when a plugin is initialized, and each frame the `PreUpdate`, `Update`, and `PostUpdate` stages are executed in sequence. 

Systems always have the following function signature:
```rust
fn my_system(&mut self, io: &mut EngineIo, query: &mut QueryResult) {}
```

Systems are in the plugin's constructor:
```rust
sched.add_system(Self::update)
    .stage(Stage::Update)
    .query::<MyComponent>(Access::Write)
    .build();
```

See the [ECS example](https://github.com/ChatImproVR/iteration0/blob/main/example_plugins/ecs/src/lib.rs) for more details.
