# jacquard-gc
A conservative garbage collector for Rust.

(requires a nightly compiler)

## Features

### Zero-cost `Gc` smart pointers

A `Gc<T>` is a shared pointer to a GCed object. That's it.
Since it's just a pointer, it can `#[derive(Copy)]`,
and have no hidden costs when accessing the pointer.

This also allows you to use `Gc<Mutex<T>>`
where you might otherwise use an `Arc`,
removing the cost of reference counting.

### `GcMut` (a garbage-collected `Box`)

`GcMut` smart pointer, for exclusive access to a GCed object

### Finalization without orphan rule issues
 - external types which don't implement `Finalize` are still usable
 - (obviously they wont get finalized if they manage a resource though)
 - Topologically ordered finalization guarantee

### `Allocator` API
- uses the unified allocator API
- allocator object is zero-sized (optimized out of containers that use it)

## Limitations
 - stops the world (i.e: not *that* concurrent)
   - im pretty sure this is the only way to do it with a zero-cost `Gc` pointer
 - large RAM preallocation (lots of GCs have this tho)
 - conservative GC
   - stuff might not get freed if pointers to the value havent been overwritten yet
   - integers that look like pointers (or more likely, were created *from* pointers) to the GCed memory will prevent collection
   - requires scanning *all* of the process's memory on every GC sweep, adds latency
 - GCed field projection is probably unsound
 - Shared `Gc` pointers don't allow exclusive access, even if you don't copy it

## Examples

### Simple linked list (without `unsafe` or refcounting)

### Passing data to a bunch of threads

### \[idk something with tokio and async\]

