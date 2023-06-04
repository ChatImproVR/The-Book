# Adding Object Behaviors
Up to now, we have worked around the fundamentals of the plugin and creation of objects/entities. This section is the highlight of the plugin where things get interesting overall.

There are several things we need to add behavior for each object. Here is a list what will it be.
- User Input Movement Control for Player
- Random Input Movement Control for Enemy
- User Input Fire Control for Player
- Random Input Fire Control for Enemy
- Bullet Interaction between each entity (spawning, collision)

## Before We Start...
Let's review the code that we have at the current moment. We have a "player", "enemy", and window screen. While the window screen does that matter that much, how can we differentiate between player and enemy? Based on the code, we can say they are graphically different (when we look at it while computers cannot), but what else is different? Sure, the position is different, but that is pretty much. Therefore, we need to add component (characteristics) to make it unique to each other. Before we get start working on these behaviors, let's add characteristics to each entity.

## Creating additional Component (Creating Characteristics)
Until now, we have a high chance that we share mostly similar code structure. However, from here, there is a high chance that the code will look very different because we might add different characteristics to the entity. Therefore, if you do not agree the logic of this tutorial, you can modify in your own way. In other words, from here and now on, I would recommend to understand the concept/syntax of the engine rather than copy and pasting the entire code and call it good. That being said...

### Player Component
First, we will work on the "Player" component. We need to have the current player position so that the server can change based on the user input.
```rust
// Add Player Component
#[derive(Component, Serialize, Deserialize, Copy, Clone)]
pub struct Player {
    pub current_position: Vec3,
}

// Implement Default for Player Component
impl Default for Player {
    fn default() -> Self {
        Self {
            current_position: Vec3::new(0.0, -50.0, 0.0),
        }
    }
}
```

The struct declares the component of the Player whereas the implementation of the Player will initilize the default component. We know we want the player to be at -50 at the Y position when the player spawns (as a default).

### Enemy Component
Unlike the Player component, we want to add another character than just current position of the enemy.
```rust
// Add Enemy Component
#[derive(Component, Serialize, Deserialize, Copy, Clone)]
pub struct Enemy {
    pub current_position: Vec3,
    pub bullet_count: u32,
}

// Implement Default for Enemy Component
impl Default for Enemy {
    fn default() -> Self {
        Self {
            current_position: Vec3::new(0.0, 50.0, 0.0),
            bullet_count: 0,
        }
    }
}
```
The struct will have not only the default position of the enemy, it will have bullet count. The bullet count represents how many bullets from that enemy is on the screen. With that description, the default value of the Enemy is 50 unit in the y value while no/0 bullets on screen when first loaded the scene.

You might asking why not for the Player component does not have a bullet limit? You can certainly add into it, but I choose not to do so. Then when you add the bullet count limit, then why can we not use the same component for enemy and player? Well, the main goal of having different component to differentiate between each entity. In other words, if they share the same struct, the name itself makes them different to each other if you add the component to each entity.

### Bullet Component
From now until a different section appears, I will just add snipt of code. If there is any explaination with certain value, I will add after the code.

```rust
// Add Bullet Component
#[derive(Component, Serialize, Deserialize, Copy, Clone)]
pub struct Bullet {
    from_player: bool,
    from_enemy: bool,
    entity_id: EntityId,
}

// Implement Default for Bullet Component
impl Default for Bullet {
    fn default() -> Self {
        Self {
            from_player: false,
            from_enemy: false,
            entity_id: EntityId(0),
        }
    }
}
```
The `entity_id` represents the parent of the entity. For example, the player entity's id or enemy entity's id will be in the `entity_id` to identify which entity of the player/enemy it came from. We will have more than one enemy, so we need to keep track which enemy entity is it from.

### PlayerStatus Component
```rust
// Add Player Status Component; this is used as a spwan timer for Player
#[derive(Component, Serialize, Deserialize, Copy, Clone)]
pub struct PlayerStatus {
    pub status: bool,
    pub dead_time: f32,
}

// Implement Default for Player Status Component
impl Default for PlayerStatus {
    fn default() -> Self {
        Self {
            status: true,
            dead_time: 0.0,
        }
    }
}
```
This component is for tracking the spawn time for the player and checking the status of the player whether it is dead or not.

### EnemyStatus Component
```rust
// Add Enemy Status Component; this is used as a spwan timer for Enemy
#[derive(Component, Serialize, Deserialize, Copy, Clone, Default)]
pub struct EnemyStatus(f32);
```
As the comment says, this component will track the spawn timer for the enemy.
<!-- 
### Score Component
```rust
// Add Score Component
#[derive(Component, Serialize, Deserialize, Copy, Clone)]
pub struct Score {
    pub score: u32,
    pub second_digit: u32,
    pub first_digit: u32,
    pub second_digit_entity: EntityId,
    pub first_digit_entity: EntityId,
}

impl Default for Score {
    fn default() -> Self {
        Self {
            score: 0,
            second_digit: 10,
            first_digit: 10,
            second_digit_entity: EntityId(0),
            first_digit_entity: EntityId(0),
        }
    }
}
```
I don't want to add this at this point since this implementation will change soon. -->

## Adding the component(characteristics) to the right entity
Let's take a look at the following code.

```rust
// All state associated with server-side behaviour
struct ServerState;

// Implement server only side functions that will update on the server side
impl UserState for ServerState {
    // Implement a constructor
    fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self {
        // Create a player status entity (not player entity)
        io.create_entity()
            .add_component(PlayerStatus::default())
            .build();

        // Create an enemy status entity (not enemy entity)
        io.create_entity().add_component(EnemyStatus(0.0)).build();

        // Create a score entity (TBD)
        io.create_entity()
            .add_component(Score::default())
            .build();

        // Create Player entity with components
        io.create_entity()
            .add_component(
                Transform::default()
                    .with_position(Vec3::new(0.0, -50.0, 0.0))
                    .with_rotation(Quat::from_euler(EulerRot::XYZ, 90., 0., 0.)),
            )
            .add_component(Render::new(PLAYER_HANDLE).primitive(Primitive::Lines))
            .add_component(Player::default())
            .add_component(Synchronized)
            .build();

        // Create Enemy with components
        io.create_entity()
            .add_component(
                Transform::default()
                    .with_position(Vec3::new(0.0, 50.0, 0.0))
                    .with_rotation(Quat::from_euler(EulerRot::XYZ, 90., 0., 0.)),
            )
            .add_component(Render::new(ENEMY_HANDLE).primitive(Primitive::Lines))
            .add_component(Synchronized)
            .add_component(Enemy::default())
            .build();

        // Create the Window entity with components
        io.create_entity()
            .add_component(Transform::default())
            .add_component(Render::new(WINDOW_SIZE_HANDLE).primitive(Primitive::Lines))
            .add_component(Synchronized)
            .build();
    }
}
```
As you might noticed, it is the same implementation like how we add `Render`, `Transform`, and `Synchronized`. The only difference is that if we declare a default setting for the component, we can call default, otherwise, we need to fill out the struct when it is initilize. For example, the `EnemyStatus` does not have a default implementation; therefore, we need to add a `f32` value into that component. Since `EnemyStatus` is a timer of the spawn time, it should initilize as `0.0`. However, we can implement a default value, but in this demonstration, I would like to share that part with you to know there is a difference.

There is one component that you have not seen above; it is a `Score` component. This updated version will be out soon; in fact, we will talk about that in the next page of this tutorial.

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
## User Input Movement
Now we need to create functions from both client side and server side for movement input. The client side will need to send the message to the server whereas the server side need to update the entity. The next two section will describe each side of the code. 

### Before we start working on client side...
We need to add more crates into the plugin to add features such as keyboard/control input, frametime, and random number generator. In the beginning of the code, add/update the following code.
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
Before we get into client side, we need to update the ClientState that will contain the keyboard input value. Update the component part of the `ClientState` as following.

```rust
#[derive(Default)]
struct ClientState {
    input: InputHelper,
}
```

Inside the `new` function of the `ClientState`, we will be using the `EngineSchedule` argument to connect the functions to the engine.

Insert the following lines inside the `new` function.
```rust
impl UserState for ClientState{
    fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self {
// Add player movement input based on keyboard/controller input
        sched
            .add_system(Self::player_input_movement_update)
            .subscribe::<InputEvent>()
            .subscribe::<GamepadState>()
            .subscribe::<FrameTime>()
            .build();
        Self::default()
    }
}
```
The first line is calling the `EngineSchedule` as `sched`. The second line will add the system to the engine schedule. We will write about the `player_input_movement_update` function in the next paragraph. But that function is declared inside the `ClientState`. The third line to fifth line, we will attach other feeatures. The features we are adding are `InputEvent`, `GamepadState`, and `FrameTime`. `InputEvent` is the connector between the client and the keyboard input. `GamepadState` is the connector between **xBox** controller and the client.  Lastly, `FrameTime` will read the frames based on the system setting; each system has their own framerate that it will change based on the frame rate. The last line will build that system with those feature attachments.

>Note: As of now, we only support **xBox** controller at this time; there are some controllers that might work, but we have not tested fully. Therefore, if there is an controller you want us to implement, please check out this [issue](https://github.com/ChatImproVR/chatimprovr/issues/85). 

Now, lets implement the `player_input_movement_update` function. First, we need to implement in the `ClientState`. Within the `ClientState`, we will declare the function. The function default will take three arguments: `self`, `EngineIo`, and `QueryResult`. In this case, we are not using the `QueryResult` argument, so we can ignore the usage of that argument. You should have somthing similar as the following code.

```rust
impl ClientState{
    fn player_input_movement_update(&mut self, io: &mut EngineIo, _query: &mut QueryResult){

    }
}
```
Now, let's fill the function. The following code will have some part of the function.

```rust
impl ClientState{
    fn player_input_movement_update(&mut self, io: &mut EngineIo, _query: &mut QueryResult){
        let mut direction = Vec3::ZERO;

        let Some(frame_time) = io.inbox_first::<FrameTime>() else { return };

        self.input.handle_input_events(io);

        let deadzone = 0.3;
    }
}
```
The direction is declare as mutable since that value will change based on the input. The second line will read the frametime based on the `FrameTime` feature; it use the idea of [subscribe and publish](https://chatimprovr.github.io/The-Book/Core_Concepts/pub_sub.html).In short, it will read the first message that contains the `FrameTime` feature and will save it as `frame_time`. The thrid line initilize the input from keyboard. If you are planning to take input from the keyboard, then you need to declare that statement before calling any keyboard input related function.The value `input` is from the updated struct on the `ClientState`. The last line set a deadzone for the controller. Controller deadzone is the amount your control stick can move before it’s recognized in game.

Now, let's focus on input listener part. Look at the next following code.
```rust
// Block 1
if let Some(GamepadState(gamepads)) = io.inbox_first() {
    if let Some(gamepad) = gamepads.into_iter().next() {
        if gamepad.axes[&Axis::LeftStickX] < -deadzone {
            direction += Vec3::new(-1.0, 0.0, 0.0);
        }
        if gamepad.axes[&Axis::LeftStickX] > deadzone {
                direction += Vec3::new(1.0, 0.0, 0.0);
        }
    }
}
// Block 2
else{
    if self.input.key_held(KeyCode::A) {
        direction += Vec3::new(-1.0, 0.0, 0.0);
    }

    if self.input.key_held(KeyCode::D) {
        direction += Vec3::new(1.0, 0.0, 0.0);
    }
}
```
Block 1 focuses on controller input whereas Block 2 focuses on keyboard input.

Block 1 states that if there is message input that has the `GamepadState`, we will iter `Vec` of input of the gamepad. Inside that `Vec`, if there is a input from the left input from the left stick `gamepad.axes[&Axis::LeftStickX]` and the value is more than the deadzone, it will update the `direction` by one unit to the left (x-axis). Same idea for positive value. We only want the player to only move left or right (no up and down). 

With that same idea of Block 1, Block 2 use the same logic, but instead of gamepad, it will be `a` and `d` keyboard button to move left and right respectivly.

Now the last part of the client side. We need to create a message so that we will send it to the server. Take a look at the following code.
```rust
if direction != Vec3::ZERO {
    let distance = direction.normalize() * frame_time.delta * PLAYER_SPEED;
    let command = MoveCommand(distance);
    io.send(&command);
    }
```
If the direction value is non zero vector, then we will create a message since there was a player input. First, we need to update the value based on the frametime; let it called as `distance`. Next, we will use the custom made `MoveCommand` message and attach the `distance` value. Once that value is attached, then will will send that command to the engine which will transfer to the server. With all that, we have finished writing the movement function on the client side. The following code the complete side of the `ClientState` implementation.
```rust
impl ClientState{
    // Send the player movement input to the server side
    fn player_input_movement_update(&mut self, io: &mut EngineIo, _query: &mut QueryResult) {
        let mut direction = Vec3::ZERO;

        let Some(frame_time) = io.inbox_first::<FrameTime>() else { return };

        self.input.handle_input_events(io);

        let deadzone = 0.3;

        if let Some(GamepadState(gamepads)) = io.inbox_first() {
            if let Some(gamepad) = gamepads.into_iter().next() {
                if gamepad.axes[&Axis::LeftStickX] < -deadzone {
                    direction += Vec3::new(-1.0, 0.0, 0.0);
                }
                if gamepad.axes[&Axis::LeftStickX] > deadzone {
                    direction += Vec3::new(1.0, 0.0, 0.0);
                }
            }
        } else {
            if self.input.key_held(KeyCode::A) {
                direction += Vec3::new(-1.0, 0.0, 0.0);
            }
            if self.input.key_held(KeyCode::D) {
                direction += Vec3::new(1.0, 0.0, 0.0);
            }
        }
        if direction != Vec3::ZERO {
            let distance = direction.normalize() * frame_time.delta * PLAYER_SPEED;
            let command = MoveCommand(distance);
            io.send(&command);
        }
    }
}
```
Now we need to work on the server side.

### Server Side
With the same idea for the client side, we need to attach the function to the engine scheduler inside the `new` function on the `ServerState`. Take a look at the following code.

```rust
impl UserState for ServerState{
    fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self {
        // Attach Player Movement Function to the Engine schedule
        sched
            .add_system(Self::player_movement_update)
            .subscribe::<MoveCommand>()
            .query(
                "Player_Movement",
                Query::new()
                    .intersect::<Transform>(Access::Write)
                    .intersect::<Player>(Access::Write),
            )
            .build();
        Self
    }
}
```
The first line is the same sytax on the client side of adding a `player_movement_update` function to the system. We need to recieve input from the `MoveCommand` message; hence, we need to subscribe the `MoveCommand` message. Lastly, we need to add an **query**. Query is one of the most important concept when it comes to server side implementation;it will fetch all the entities that matches the conditions. The first parameter of the query function takes the name of the query; we will talk the importance of the naming the query in the other systems such as collision. The second parameter is the condition of the query. In this query, we want all the entities that has the `Player` component and `Transform` component. With those component, the function has permission to modify the component. If you want the component not to be modified, then we can simply change the permission from `Access::Write` to `Access::Read`. The last function is building just like the client side did. Now, lets switch our focus to the `player_movement_update` function.

First, we are going to implement the function inside the `ServerState` as the following code. 

```rust
impl ServerState{
    fn player_movement_update(&mut self, io: &mut EngineIo, query: &mut QueryResult){

    }
}
```
From there, we need to add the first condition such that we are going to execute the function if the message inbox has the `MoveCommand` like the following line.
```rust
for player_movement in io.inbox::<MoveCommand>(){

}
```
The `player_movement` will be the iterator of the `io.inbox::<MoveCommand>()`. Since this statement is saying that we have an input from the client, we then need to modify the player movement. Now we need to go every entity that qualifies the condition. In the previous part, we have indicated that all entities that has the `Player` component and the `Transform` component will be modified. There is only one entity has those components which is the Player entity. ßßßß


```rust
for entity in query.iter("Player_Movement"){}
```

If we have multiple quries (not in this case but for future cases), then we can differentiate based on the name of the query. There is more detailed information in [here](../Core_Concepts/entity_component_system.md) or in the system development. From there on, we can add conditions before actually moving the entity inside the query; for example, if the player is about to go out of bounds, then we need to stop that. We will first the current position of the player from the `Player` component.

```rust
let x_limit = WITDH / 2.0;
if query.read::<Player>(entity).current_position.x + player_movement.0.x - PLAYER_SIZE < -x_limit
    || query.read::<Player>(entity).current_position.x + player_movement.0.x + PLAYER_SIZE > x_limit
    {
        return;
    }
```

If the player is about to go out of bound, then we will do nothing with that input; otherwise, we will change the player position based on the input like the following statement.

```rust
query.modify::<Transform>(entity, |transform| {
    transform.pos += player_movement.0;
});
```

At the same time, we will also update the Player current position by modifying the `Player` component value.

```rust
query.modify::<Player>(entity, |player| {
    player.current_position += player_movement.0;
});
```

With all that, we have completed set up the player movements for both server and client. The following code is the final part of the server side code.

```rust
impl ServerState{
    // The function that will handle the player movement
    fn player_movement_update(&mut self, io: &mut EngineIo, query: &mut QueryResult) {
        for player_movement in io.inbox::<MoveCommand>() {
            for entity in query.iter("Player_Movement") {
                let x_limit = WITDH / 2.0;
                if query.read::<Player>(entity).current_position.x + player_movement.0.x - PLAYER_SIZE
                    < -x_limit
                    || query.read::<Player>(entity).current_position.x + player_movement.0.x + PLAYER_SIZE
                        > x_limit
                {
                    return;
                }

                query.modify::<Transform>(entity, |transform| {
                    transform.pos += player_movement.0;
                });
                query.modify::<Player>(entity, |player| {
                    player.current_position += player_movement.0;
                });
            }
        }
    }
}
```
## Random Movement
If we can make movements for the player, then we should add movements for the enemies as well. While we could set it up as client sending a message to the server side, for this purpose, let's set up only on the server side. In other words, we do not need to do use the `MoveCommand` message.

First, we need to attach the system call to the engine. The following would look very similar to the Player movement system.

```rust
impl UserState for ServerState{
    fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self {
// Attach Enemy Movement Function to the Engine schedule
sched
    .add_system(Self::enemy_movement_update)
    .subscribe::<FrameTime>()
    .query(
        "Enemy_Movement",
        Query::new()
            .intersect::<Transform>(Access::Write)
            .intersect::<Enemy>(Access::Write),
    )
    .build();
    Self
    }
}
```
The only differences are that there is no `subscribe` for `MoveCommand` but `subscribe` for `Frametime` and the query filter changed from `Player` to `Enemy`. I will not go more depth about this code since we have seen this pattern before. In fact, most of the attachment to the engine will be the same except the query part which will be highlighted if needed in other systems.

Let's switch our focus to `enemy_movement_update` function that will be implemented in the `ServerState`. Because we are not looking at the inbox messages, we will look at every entities that qualifies the condition (the entity contains the `Transform' and `Enemy` component and mutable).

```rust
impl ServerState{
    fn enemy_movement_update(&mut self, io: &mut EngineIo, query: &mut QueryResult) {
        for entity in query.iter("Enemy_Movement"){

        }
    }
}
```
From there, we want to get the `Frametime`; therefore, we will use this statement. This is the same line we used for the User Input Movement on the client side, but this time only for the server side.

```rust
let Some(frame_time) = io.inbox_first::<FrameTime>() else { return };
```

Next, we need to initilize our random number generators using the `Pcg` crate. Check out [here](https://docs.rs/pcg/latest/pcg/) to learn more about `Pcg`, but it will be something like the following.

```rust
let mut pcg_random_move = Pcg::new();
let mut pcg_random_direction = Pcg::new();
```

I created two seperated random generator so that each part will have a different seed for different character of the movement: one random generator will be the magnitude the whereas the other one will be for the direction.

From there, we need to get values for `x` and `y` positions using the random generator. We will conditions to generate these values like the following code.

```rust
let x = if pcg_random_direction.gen_bool() {
    pcg_random_move.gen_f32() * 1.
} else {
    pcg_random_move.gen_f32() * -1.
};

let y = if pcg_random_direction.gen_bool() {
    pcg_random_move.gen_f32() * 1.
} else {
    pcg_random_move.gen_f32() * -1.
};

let speed = Vec3::new(x, y, 0.);

let direction = speed.normalize() * frame_time.delta * ENEMY_SPEED;
```

Once the `x` and `y` values are automatically generated, then we will declare the direction as a `Vec3` format while considering the `frame_time` and `ENEMY_SPEED`.

With the same logic as `player_movement_update`, we will read the current position of that enemy and check whether that move is valid within bound (and vertical limitations). If it is valid, then we will move accordingly by modifying the `Transform` component and update the new position of the enemy in the `Enemy` component. Otherwise, nothing will happen. That being said, the following code will be the complete side of the random movement.

```rust
impl ServerState{
    // The function that will handle the enemy movement
    fn enemy_movement_update(&mut self, io: &mut EngineIo, query: &mut QueryResult) {
        for entity in query.iter("Enemy_Movement") {
            let Some(frame_time) = io.inbox_first::<FrameTime>() else { return };
            let mut pcg_random_move = Pcg::new();
            let mut pcg_random_direction = Pcg::new();

            let x = if pcg_random_direction.gen_bool() {
                pcg_random_move.gen_f32() * 1.
            } else {
                pcg_random_move.gen_f32() * -1.
            };

            let y = if pcg_random_direction.gen_bool() {
                pcg_random_move.gen_f32() * 1.
            } else {
                pcg_random_move.gen_f32() * -1.
            };

            let speed = Vec3::new(x, y, 0.);

            let direction = speed.normalize() * frame_time.delta * ENEMY_SPEED;

            let x_limit = WITDH / 2.0;
            let y_upper_limit = HEIGHT / 2.;
            let y_limit = HEIGHT / 5.;

            let current_position = query.read::<Enemy>(entity).current_position;

            if (current_position.x + direction.x - ENEMY_SIZE < -x_limit)
                || (current_position.x + direction.x + ENEMY_SIZE > x_limit)
                || (current_position.y + direction.y >= y_upper_limit)
                || (current_position.y + direction.y < y_limit)
            {
                return;
            }
            query.modify::<Transform>(entity, |transform| {
                transform.pos += direction;
            });
            query.modify::<Enemy>(entity, |enemy| {
                enemy.current_position += direction;
            });
        }
    }
}
```

## Fire System
We can implement the same method for User Input for player fire system whereas the enemy will have the same idea for random movement but fire rate. However, we do not have the bullet entity. Let's take a look into these systems for both player and enemy.

### Player Fire Client System
```rust
impl UserState for ClientState{
    fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self {   
        // Add player fire input based on keyboard/controller input
        sched
            .add_system(Self::player_input_fire_update)
            .subscribe::<InputEvent>()
            .subscribe::<GamepadState>()
            .build();
        Self::default()
    }
}
```
With the same idea of movement input, we will subscribe the `InputEvent` and `GamepadState` to take input for both controller and keyboard. This code will be in the `new` function in the `ClientState`.

```rust
impl ClientState{
    // Send the player fire input to the server side
    fn player_input_fire_update(&mut self, io: &mut EngineIo, _query: &mut QueryResult) {
        self.input.handle_input_events(io);

        if let Some(GamepadState(gamepads)) = io.inbox_first() {
            if let Some(gamepad) = gamepads.into_iter().next() {
                if gamepad.buttons[&Button::East] {
                    let command = FireCommand(true);
                    io.send(&command);
                }
            }
        }

        if self.input.key_pressed(KeyCode::Space) {
            let command = FireCommand(true);
            io.send(&command);
        }
    }
}
```
In the code above, we set the client side if the space bar was triggered or the east side button on the right side is triggered. The `gamepad.buttons[&Button::East]` indicates the east button on the right side. You can find more button information in the engine implementation.

### Player Fire Server System
For bullets, we need to implement two functions on the `ServerState`: a function for generating the bullets and a function for moving the bullets. Therefore, we have two system to attach the engine.

```rust
impl UserState for ServerState{
        fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self {
        // Attach Player Fire Function to the Engine schedule
        sched
            .add_system(Self::player_fire_update)
            .subscribe::<FireCommand>()
            .query(
                "Player_Fire_Input",
                Query::new().intersect::<Player>(Access::Read),
            )
            .build();

        // Attach Player Bullet Movement Function to the Engine schedule
        sched
            .add_system(Self::player_bullet_movement_update)
            .subscribe::<FrameTime>()
            .query(
                "Player_Bullet_Movement",
                Query::new()
                    .intersect::<Transform>(Access::Write)
                    .intersect::<Bullet>(Access::Write),
            )
            .build();
        Self
    }
}

```
The first system is reading if there is a `FireCommand` from the client side in order to fire the system. The query for the first system only needs to read the player's current position in order ot shoot the bullet from player's current position from the `Player` component. The second system is moving the player's bullet. The query for this system will be the same idea for any movement system: the system needs `Transform` component and `Bullet` component. Let's take a look at the `player_fire_update` function first.

```rust
impl ServerState{
    // The function that will handle the player fire
    fn player_fire_update(&mut self, io: &mut EngineIo, query: &mut QueryResult) {
        if let Some(FireCommand(_value)) = io.inbox_first() {
            for entity in query.iter("Player_Fire_Input") {
                // Create the bullet entity from the plauyer position (the left bullet)
                io.create_entity()
                    .add_component(
                        Render::new(PLAYER_BULLET_HANDLE).primitive(Primitive::Triangles),
                    )
                    .add_component(Synchronized)
                    .add_component(Bullet {
                        from_enemy: false,
                        from_player: true,
                        entity_id: entity,
                    })
                    .add_component(Transform::default().with_position(
                        query.read::<Player>(entity).current_position
                            + Vec3::new(-PLAYER_SIZE / 2., PLAYER_SIZE / 2., 0.0),
                    ))
                    .build();

                // Create the bullet entity from the plauyer position (the right bullet)
                io.create_entity()
                    .add_component(
                        Render::new(PLAYER_BULLET_HANDLE).primitive(Primitive::Triangles),
                    )
                    .add_component(Synchronized)
                    .add_component(Bullet {
                        from_enemy: false,
                        from_player: true,
                        entity_id: entity,
                    })
                    .add_component(Transform::default().with_position(
                        query.read::<Player>(entity).current_position
                            + Vec3::new(PLAYER_SIZE / 2., PLAYER_SIZE / 2., 0.0),
                    ))
                    .build();
            }
        }
    }
}
```
As you can see the code above, we can create new entities inside the function; therefore, we do not need to create the bullet entities inside the `new` function because we do not need to load the bullet in the scene at the beginning. We created two bullet entities so that the player can shoot from top left and top right side of the player's current position. Now, let's take a look at the player's bullet movement.

```rust
impl ServerState{
    // The function that will handle the player bullet movement
    fn player_bullet_movement_update(&mut self, io: &mut EngineIo, query: &mut QueryResult) {
        let Some(frame_time) = io.inbox_first::<FrameTime>() else { return };

        for entity in query.iter("Player_Bullet_Movement") {
            if query.read::<Bullet>(entity).from_player {
                if query.read::<Transform>(entity).pos.y > HEIGHT / 2. - 2.5 {
                    io.remove_entity(entity);
                }
                query.modify::<Transform>(entity, |transform| {
                    transform.pos +=
                        Vec3::new(0.0, 1.0, 0.0) * frame_time.delta * PLAYER_BULLET_SPEED;
                });
            }
        }
    }
}
```
The first line inside the function is the same line to read the `frame_time` value. For the every bullet from the player, if the bullet is out of bound from the scene, then remove that bullet from the screen; otherwise, we will move the bullet one unit up to the y-axis with the custom set speed for the player's bullet. As you read the code, it is very similar to movement for player and enemy. Let's switch our intention to `Enemy` since I have added some complexity to bullet fire rate.


### Enemy Fire Server System
```rust
impl UserState for ServerState{
        fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self {
        // Attach Enemy Fire Function to the Engine schedule
        sched
            .add_system(Self::enemy_fire_update)
            .query(
                "Enemy_Fire_Input",
                Query::new().intersect::<Enemy>(Access::Write),
            )
            .build();

        // Attach Enemy Bullet Movement Function to the Engine schedule
        sched
            .add_system(Self::enemy_bullet_movement_update)
            .subscribe::<FrameTime>()
            .query(
                "Enemy_Bullet_Movement",
                Query::new()
                    .intersect::<Transform>(Access::Write)
                    .intersect::<Bullet>(Access::Write),
            )
            .query(
                "Enemy_Bullet_Count_Update",
                Query::new().intersect::<Enemy>(Access::Write),
            )
            .build();
        Self
    }
}
```
In the first system attachment, it is the same idea what we did for player's bullet. The only difference is that we are reading from `Enemy` component rather than `Player` component. The second system attachment is very different; rather not having one query, but two quries. The first query will fetch all entities that has the `Bullet` and `Transform` components and modify it whereas the second query will only read from the `Enemy` component. Let's take a look at the `enemy_fire_update` function.

```rust
impl ServerState{
    // The function that will handle the enemy fire update
    fn enemy_fire_update(&mut self, io: &mut EngineIo, query: &mut QueryResult) {
        let mut pcg_fire = Pcg::new();

        for entity in query.iter("Enemy_Fire_Input") {
            if pcg_fire.gen_bool() {
                if query.read::<Enemy>(entity).bullet_count < ENEMY_MAX_BULLET {
                    // Increase the bullet count that are on screen from that enemy by 1
                    query.modify::<Enemy>(entity, |value| {
                        value.bullet_count += 1;
                    });
                    io.create_entity()
                        .add_component(
                            Render::new(ENEMY_BULLET_HANDLE).primitive(Primitive::Triangles),
                        )
                        .add_component(Synchronized)
                        .add_component(Bullet {
                            from_enemy: true,
                            from_player: false,
                            entity_id: entity,
                        })
                        .add_component(Transform::default().with_position(
                            query.read::<Enemy>(entity).current_position
                                + Vec3::new(0., -ENEMY_SIZE / 2., 0.),
                        ))
                        .build();
                }
            }
        }
    }
}
```
When you first look at this function, you will notice a lot of similarity from the `player_fire_update` function such as create an entity as bullet that is from the enemy rather than the player. However, before creating the enemy's bullet entity, we have additional condition on enemy's bullet display limit. We have this limitation because it will be almost impossible to play since it will continously spamming bullets that the player will not even have a chance to score. Therefore, if there is more bullets than `ENEMY_MAX_BULLET` value from each enemy, then it will not generate a new one until the bullet is gone from the scene (either from hitting the player or going out of bound). Therefore, we will read the current bullet amount on the screen from the `Enemy` component. We will modify that value in the next function. At the same time, when we generate the bullet, we will store the `parent entities id`. This value is important in the next function.

```rust
impl ServerState{
    // The function that will handle the enemy bullet movement
    fn enemy_bullet_movement_update(&mut self, io: &mut EngineIo, query: &mut QueryResult) {
        if let Some(frame_time) = io.inbox_first::<FrameTime>() {
            for entity in query.iter("Enemy_Bullet_Movement") {
                if query.read::<Bullet>(entity).from_enemy {
                    if query.read::<Transform>(entity).pos.y < -HEIGHT / 2. + 2.5 {
                        if query
                            .iter("Enemy_Bullet_Count_Update")
                            .any(|id| id == query.read::<Bullet>(entity).entity_id)
                        {
                            query.modify::<Enemy>(
                                query.read::<Bullet>(entity).entity_id,
                                |value| {
                                    value.bullet_count -= 1;
                                },
                            );
                        }
                        io.remove_entity(entity);
                    }
                    query.modify::<Transform>(entity, |transform| {
                        transform.pos +=
                            Vec3::new(0.0, -1.0, 0.0) * frame_time.delta * ENEMY_BULLET_SPEED;
                    });
                }
            }
        }
    }
}
```
This function is similar as `player_bullet_movement_update` function but having an additional bullet counter on screen and updater. Let's look at the query that states for `Enemy_Bullet_Count_Update`. The condition states that find the parent(the enemy) and see if that enemy is alive. If the enemy is alive, then update the `bullet_count` value by 1; otherwise, ignore it since that enemy is dead. Once that is complete, then remove the bullet from the screen. You can customize the option on how to implement the limitation of the bullet, but we need to use the idea of parenting between entities.


## Interaction between Entities
Now we got a system that can shoot and move, but what is fun is that? We need to add collision between each entities. If a bullet hit a player or enemy, then it should get removed. After a certain time is passed from its death, then the player or enemy should respawn. Once that is set up, we have a game-like Galaga in our hand.

### Collision
First, let's work on collision. There are two type of collision in Galaga: player bullet hitting the enemy or the enemy's bullet htting the player. But before that, lets make a collision function so that we do not need to write the same collision code multiple times.

### Collision Function
This function is based off from standard, classic game logic of collision.

```rust
// The function that will handle the collision detection
fn collision_detection(
    obj1_x_position: f32,
    obj1_y_position: f32,
    obj1_size: f32,
    obj2_x_position: f32,
    obj2_y_position: f32,
    obj2_size: f32,
) -> bool {
    // If the object 1 is within the object 2 based on the sqaure hitbox intersection
    if obj1_x_position - (obj1_size / 2.) <= obj2_x_position + (obj2_size / 2.)
        && obj1_x_position + (obj1_size / 2.) >= obj2_x_position - (obj2_size / 2.)
        && (obj1_y_position - (obj1_size / 2.) <= obj2_y_position + (obj2_size / 2.))
        && (obj1_y_position + (obj1_size / 2.) >= obj2_y_position - (obj2_size / 2.))
    {
        // Return true if the object 1 is within the object 2, or vice versa
        return true;
    }
    // Otherwise, return false
    return false;
}
```
We do not need to fully understand how the logic works, but we need to understand the input/argument of the function. The `x` and `y` positions of the object/entity is define as the center (not at the bottom left which most engine does). The `size` argument identifies the legnth between left side to the right side. In other words, we are defining the entity/object as a sqaure. If these two squares overlap each other, then we consider as collision and return that they are hitting each other; otherwise, it is not a collision.

### Player Bullet to Enemy
```rust
impl UserState for ServerState{
    fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self{
        // Attach Player Bullet to Enemy Collision Function to the Engine schedule
        sched
            .add_system(Self::player_bullet_to_enemy_collision)
            .query(
                "Player_Bullet",
                Query::new()
                    .intersect::<Transform>(Access::Read)
                    .intersect::<Bullet>(Access::Write),
            )
            .query(
                "Enemy",
                Query::new()
                    .intersect::<Enemy>(Access::Write)
                    .intersect::<Transform>(Access::Read),
            )
            .build();
        Self
    }
}
```


```rust
impl ServerState{
    // The function that will handle the collision from player bullet to enemy
    fn player_bullet_to_enemy_collision(&mut self, io: &mut EngineIo, query: &mut QueryResult) {
        for entity1 in query.iter("Player_Bullet") {
            if query.read::<Bullet>(entity1).from_player {
                for entity2 in query.iter("Enemy") {
                    let current_player_bullet = query.read::<Transform>(entity1).pos;
                    let current_enemy = query.read::<Transform>(entity2).pos;

                    // If the bullet hit the enemy
                    if collision_detection(
                        current_player_bullet.x,
                        current_player_bullet.y,
                        BULLET_SIZE,
                        current_enemy.x,
                        current_enemy.y,
                        ENEMY_SIZE,
                    ) {
                        io.remove_entity(entity1);
                        io.remove_entity(entity2);
                    }
                }
            }
        }
    }
}

```
### Enemy Bullet to Player


```rust
impl UserState for ServerState{
    fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self{
        // Attach Enemy Bullet to Player Collision Function to the Engine schedule
        sched
            .add_system(Self::enemy_bullet_to_player_collision)
            .query(
                "Enemy_Bullet",
                Query::new()
                    .intersect::<Transform>(Access::Read)
                    .intersect::<Bullet>(Access::Write),
            )
            .query(
                "Player",
                Query::new()
                    .intersect::<Player>(Access::Write)
                    .intersect::<Transform>(Access::Read),
            )
            .query(
                "Player_Status_Update",
                Query::new().intersect::<PlayerStatus>(Access::Write),
            )
            .query(
                "Enemy_Bullet_Count_Update",
                Query::new().intersect::<Enemy>(Access::Write),
            )
            .build();
        Self
    }
}
```

```rust
impl ServerState{
        // The function that will handle the collision from enemy bullet to player
    fn enemy_bullet_to_player_collision(&mut self, io: &mut EngineIo, query: &mut QueryResult) {
        for entity1 in query.iter("Enemy_Bullet") {
            if query.read::<Bullet>(entity1).from_enemy {
                for entity2 in query.iter("Player") {
                    let current_enemy_bullet = query.read::<Transform>(entity1).pos;
                    let current_player = query.read::<Transform>(entity2).pos;

                    // If the bullet hit the player
                    if collision_detection(
                        current_enemy_bullet.x,
                        current_enemy_bullet.y,
                        BULLET_SIZE,
                        current_player.x,
                        current_player.y,
                        PLAYER_SIZE,
                    ) {
                        if query
                            .iter("Enemy_Bullet_Count_Update")
                            .any(|id| id == query.read::<Bullet>(entity1).entity_id)
                        {
                            query.modify::<Enemy>(
                                query.read::<Bullet>(entity1).entity_id,
                                |value| {
                                    value.bullet_count -= 1;
                                },
                            );
                        }
                        io.remove_entity(entity1);
                        io.remove_entity(entity2);
                        for entity3 in query.iter("Player_Status_Update") {
                            query.modify::<PlayerStatus>(entity3, |value| {
                                value.status = false;
                            });
                        }
                    }
                }
            }
        }
    }
}
```
### Respawning

## Summary/Current Code Progress