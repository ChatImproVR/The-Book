# Adding Object Behaviors
Up to now, we have worked around the fundamentals of the plugin and creation of objects/entities. This section is the highlight of the plugin where things get interesting overall.

There are several things we need to add behavior for each object. Here is a list what will it be.
- User Input Movement Control for Player
- Random Input Movement Control for Enemy
- User Input Fire Control for Player
- Random Input Fire Control for Enemy
- Bullet Interaction between each entity (spawning, collision)

## Before We Start...
Talk about adding more components to the entities so that it is differnet than others (make them and how to attach them)

## Communication from Client to Server
The most common method that we will be using is communication from client to server. We have in depth documentation regarding [how it works](/Core_Concepts/client_and_server.md). The method we will be using is called [subscribe/publish method](/Core_Concepts/pub_sub.md). In short summary, the client will send a message to the server, and the server will update based on that message call. This method involves serializing and deserializing the message: the client will serialize the message before it sends to the server, and the server will deserialize the message to read the message. Therefore, we will need to use the following library.

```rust
use serde::{Deserialize, Serialze};
```

Insert the following line where the other libraries are located.

### Creating a Movement Input Message Container
Before we start writing out the message container, we need to structure what message we want to send to the server from the client. We want to have the information on what direction/vector the object needs to move. The message can also contain on which object will be moving, but we know that the only object that will be moving from input from users is the player object; hence, it is not required to have. However, (this is optional) if we are planning to expand Player versus Player gameplay, then we do want to mention it. The following code will be the message container that we will be using for this tutorial.

```rust
// Add movement command as message from client to server
#[derive(Message, Serialize, Deserialize)]
#[locality("Remote")]
struct MoveCommand(Vec3);
```
The first line declare what micro we will be using for this component/container. Because we are making a message container for movement input, it will contain `Message`, `Serialize`, and `Deserialize` micro services. The second line `#[locality('Remote')]` states that how we have specified the `Locality` of this message type. `Local` messages are sent to other plugins on this host. `Remote` messages are sent to the remote host. For example, a `Remote` message sent from a client would be received _only_ at the server. The last line will be the structure of the message container. Because we are sending just the direction, it will be a struct that only contains as a `Vec3`. We can change the format differently as below.

```rust
// Add movement command as message from client to server
#[derive(Message, Serialize, Deserialize)]
#[locality("Remote")]
struct MoveCommand{
    direction: Vec3,
}
```
This format will specify which attribute we want to call for later in the server side. As of now, we will stick with the first format, but there will be a note if there is a difference if someone else use the format above.

### Creating a Fire Input Message Container
With the same approach for Movement Input Message, we also need to establish the fire input message container as well. The only input we need from the is whether the "shoot" button has been triggered or not. Therfore the implementation of this message container will be the following:

```rust
// Add fire command as a message from client to server
#[derive(Message, Serialize, Deserialize)]
#[locality("Remote")]
struct FireCommand(bool);
```
## Movement
Now we need to create functions from both client side and server side for movement input. The client side will need to send the message to the server whereas the server side need to update the entity. The next two section will describe each side of the code. 

### Before we start working on it...
We need to add more crates into the plugin to add features such as keyboard/control input and frametime. In the beginning of the code, add/update the following code.
```rust
// Add libraries from the cimvr_engine_interface crate
use cimvr_engine_interface::{make_app_state, pcg::Pcg, pkg_namespace, prelude::*, FrameTime};

// Add libraries from the cimvr_common crate
use cimvr_common::{
    desktop::{InputEvent, KeyCode},
    gamepad::{Axis, Button, GamepadState},
    glam::{EulerRot, Quat, Vec3},
    render::{Mesh, MeshHandle, Primitive, Render, UploadMesh, Vertex},
    utils::input_helper::InputHelper,
    Transform,
};
```

### Client Side
Inside the `new` function of the `ClientState`, we will be using the `EngineSchedule` argument to connect the functions to the engine.

Insert the following lines inside the `new` function.
```rust
// Add player movement input based on keyboard/controller input
        sched
            .add_system(Self::player_input_movement_update)
            .subscribe::<InputEvent>()
            .subscribe::<GamepadState>()
            .subscribe::<FrameTime>()
            .build();
```
The first line is calling the `EngineSchedule` as `sched`. The second line will add the system to the engine schedule. We will write about the `player_input_movement_update` function in the next paragraph. But that function is declared inside the `ClientState`. The third line to fifth line, we will attach other feeatures. The last line will build that system with those feature attachments.



### Server Side

## Fire

## Internal Functions within the Server

### Random Generated Values

### Movement & Fire

## Interaction between Entities

### Collision

### Respawning

#### Adding more entities

## Summary/Current Code Progress