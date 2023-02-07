# Names
In order to communicate between plugins, we need some sort of naming scheme for data types. This naming scheme should be unambigious. We should allow plugins to be designed to use interfaces rather than talk exclusively with just one implementation or provider.

We use strings to identify resources (read: Component and Message datatypes) in ChatImproVR. The convention is to use the pattern `MyNamespace/MyResourceName`. We provide the `pkg_namespace!()` macro to automatically prepend your package/crate's name to the given name, allowing you to easily define resource names within the context of your plugin.

Great care should be taken to ensure that the resource name is **Universally Unique**. 

One and only one data format can be associated with a resource name. **If the data format of a resource changes, then a new name must be picked**. This is to ensure dependent plugins will not consume garbled data. ChatImproVR does not use a self-describing data format (for efficiency), so collisions can cause data to appear corrupted from the consumer's point of view.
