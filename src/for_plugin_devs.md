## How does ChatImproVR work?
**ChatImproVR is based** on plugins. **Plugins** run on both the **Client** and the **Server**. Each plugin adds functionality to the virtual world by providing a number of **Systems**.

We refer to the **Client** or the **Server** a **Plugin** is running on as the **"Host"**. **Systems** communicate with the **Host** via **Channels** and via the Entity Component System (**ECS**). 

The **Host** can communicate with external APIs, and with the **Remote**. From the **Client**'s perspective, the **Remote** is the **Server**. From the **Server**'s perspective, the **Remotes** are the connected **Clients**.

## Channels
**Channels** are modeled after the familiar Pub/Sub pattern. **Systems** subscribe to **Channels**, and receive **Messages** via their **Inbox**. Each **Channel** is referred to by it's **Unique ID**. This ID corresponds to exactly one responsibility and associated datatype. _If the underlying datatype changes, the ID must also change_. This is to ensure compatability between plugins. Otherwise, plugins may consume corrupted data.

Each channel is either **Local** or **Remote**. The **Local** **Channels** can only send **Messages** within their **Host**. Conversely, **Remote Channels** can only send messages to the **Remote**. This is to help avoid confusion as to where a message might have originated, and to make optimizing communication easier.

## Entity Component System
ChatImproVR's **ECS** is very similar to other game engines' architectures (Bevy being a notable example). Essentially, the **ECS** acts as a global database, where **Entities** are simply keys and **Components** are sparsely populated columns in the metaphorical database. **Systems** make **Queries** to the **ECS**, which are usually joining one more more **Components**. 

Each **Host** has it's own **ECS**. The **Server**'s **ECS** is intended to contain the state of the world, while the **Client**'s **ECS**' are intended to contain snapshots of the world and locally visible objects such as graphical user interfaces and markers, as well as predictions of motion or action.

One important difference from other engines is that ChatImproVR handles **Components** via their **Unique ID**, instead of by their datatype. Just like **Channels**, _If the underlying datatype changes, the ID must also change_. Another important difference is that **Entities** are intended to be _universally unique_. This means that **Entities** could be transferred between **Servers**, so items in the virtual world can be transferred throughout the metaverse.

## Scheduling
Execution of **Systems** on each **Host** is broken up into a number of **Stages**. Currently, the following stages are available (listed in order of execution):
* **Init**: Executed once, right after the plugin is initialized.
* **Pre-Update**: Executed before each update, once per **Frame**.
* **Update**: Executed once per **Frame**.
* **Post-Update**: Executed after each update, once per **Frame**.

## Synchronization
There is a special **Sychronized** component which, if attached to an **Entity** on the **Server**, will be forcibly copied to the **Clients**'s **ECS** each **Update**. This mechanism is intended to make it very easy to create content which is visible to all **Clients** immediately. 
