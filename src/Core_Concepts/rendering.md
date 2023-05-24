# Rendering
The rendering interface in ChatImproVR works as follows:

1. Plugins define a `MeshHandle`, which is just a name for the mesh we will upload. This name will be used in the `Render` component.
```rust
const CUBE_HANDLE: MeshHandle = MeshHandle::new(pkg_namespace!("Cube"));
```

2. Plugins send an `UploadMesh` message containing the `MeshHandle` and the desired mesh data. This tells the rendering engine about the mesh data, so that we may later reference it by its `MeshHandle`. 
```rust
io.send(&UploadMesh {
    mesh: cube(),
    id: CUBE_HANDLE,
});
```

3. To render a mesh, create an entity with the `Transform` and `Render` components. The `Transform` component specifies the position and orientation of the rendered mesh. This is available in shaders via the `mat4 view` uniform. The `Render` component specifies how the mesh is to be displayed; e.g. which primitive to use, indexing limites, etc. Note how we specify which `MeshHandle` to render!

```rust
io.create_entity()
    .add_component(Transform::identity());
    .add_component(Render::new(CUBE_HANDLE))
    .build();
```

> NOTE: All rendering data is processed client-side; e.g. UploadMesh should be sent in ClientState.

See the [cube example](https://github.com/ChatImproVR/chatimprovr/blob/main/example_plugins/cube/src/lib.rs)!

## Defining meshes
Meshes are defined using vertices and indices. The default shader uses `Vertex`'s `uvw` field to represent vertex color.
