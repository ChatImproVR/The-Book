# Plugins
One of the most important features behind the engine is the ability to develop a **plugin**. Plugins have the ability to talk to other plugins, and the host. Plugins implement game logic.

Plugins are split into Client and Server sides. This acheived in code like so:
```rust
// Declare structures containing state
// Note that the names do not matter, but are informative
// make_app_state!() only cares about the order of it's arguments
struct ServerState;
struct ClientState;

// Expose the these structures to the engine. Order matters!
// This is analogous to main() in a regular Rust program.
make_app_state!(ClientState, ServerState);

// Implement constructors
impl UserState for ClientState {
    fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self {
        Self
    }
}

impl UserState for ServerState {
    fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self {
        Self
    }
}
```

While both the server and client states are declared in the same plugin, only one will be picked at runtime. The client-side code runs independently on each client, and the server-side code runs in a single instance on the server.

ChatImproVR plugins are written in WebAssembly. See the next section for details!

# Tips and tricks
If you are implementing a plugin which will behave the same on the client and server, 
you may use a type alias to avoid duplicating code:
```rust
// Implement your plugin state as usual
struct PluginState;
impl UserState for PluginState {
    // ...
}

// Use a type alias to create an alternate name for the state in order to pass both to make_app_state!() without causing an error
type ServerState = PluginState;
make_app_state!(PluginState, ServerState);

//make_app_state!(PluginState, PluginState); // This would cause an error!
```
