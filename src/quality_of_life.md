# Quality of Life Items

You can use the derive macro `ComponentDerive` to automatically set up your component like so:

```rs
#[derive(ComponentDerive, ...snip)]
#[size(16)]
struct Foo {
    a: i32,
    b: f32
}
```

This will automatically set up your struct as a component without as much boilerplate.
