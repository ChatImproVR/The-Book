# Common fixes

## cannot find function `_print_str` in this scope
```
error[E0425]: cannot find function `_print_str` in this scope
  --> src/obj.rs:31:17
   |
31 |                 dbg!(pos, uvw);
   |                 ^^^^^^^^^^^^^^ not found in this scope
   |
   = note: this error originates in the macro `$crate::println` which comes from the expansion of the macro `dbg` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider importing one of these items
   |
1  | use cimvr_engine_interface::prelude::_print_str;
   |
1  | use crate::_print_str;
   |
```

### The fix:
Just add the prelude from the engine interface:
```rust
use cimvr_engine_interface::prelude::*;
```
