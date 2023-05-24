# Engine development
The current implementation of ChatImproVR's engine is `chatimprovr`. It's assumed that you've read the previous sections, and understand how to write plugins for ChatImproVR.

ChatImproVR is split into a few different parts. The **Engine** is the core of a ChatImproVR **Host**. It is responsible for managing the ECS, facilitating pub/sub messaging, and loading plugin code. The engine contains no code responsible for interacting with the outside world; it has no networking cabablity nor does it render scenes or take user input. That is the responsibility of the rest of the **Host**'s code. 

The **Client** contains a number of subsystems which allow the **Engine** to interact with the user:
* `desktop`: All interfacing with desktop platforms; keyboard and mouse input. Contains the desktop mainloop
* `vr`: All interfacing with virtual reality platforms; controller and headset motion. Contains the VR mainloop
* `ui`: Simple GUI for use from within plugins
* `render`: Putting meshes on the screen, assigning shaders, etc.

Each of these subsystems interacts with the **Engine** by making ECS queries and checking the message **Inboxes** that they've subscribed to. In this way, creating subsystems is very much like writing plugins.

The **Server** is a lot simpler, except that it may connect to many **Clients** and is intended to be a headless program. It has a few simple subsystems which e.g. report which clients are connected.

