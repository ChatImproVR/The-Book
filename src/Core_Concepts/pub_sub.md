# Pub/Sub channels
ChatImproVR uses the [Publish-Subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern). 

## Local communication:
Messages are sent between stages, not within stages. Messages may only be received once:
![Local communication diagram](./local_communication.svg)

Messages may be sent and received each stage, even to the stage on the next frame:
![Local communication diagram 2](./local_communication_2.svg)

Messages are broadcasted to all subscribing plugins, including your own plugin:
![Local communication diagram 3](./local_communication_3.svg)

Systems' order of execution is the same as the order in which they were declared:
![Local communication diagram (inside)](./local_communication_inside.svg)

## Remote communication
All Remote messages are sent at the end of each frame.
![Remote messages](./remote_communication.svg)
