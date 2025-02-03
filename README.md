# jacquard-gc
A conservative garbage collector for Rust.

(requires a nightly compiler)

## Features

### Zero-cost `Gc` smart pointers

A `Gc<T>` is a shared pointer to a GCed object. That's it.
Since it's just a pointer, it can `#[derive(Copy)]`,
without any extra costs when accessing the pointer.

This also allows you to use a `Gc<Mutex<T>>`
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

### DST support
- supports casting `Gc<T>` into `Gc<dyn Trait>`
- supports `unsize` casts (e.g: `Gc<[T; 10]>` into `Gc<[T]>`)

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
 - Collection phase is single threaded
   - maybe if a certain number of threads are waiting, one of them can wake up the collector thread, which then can like, enlist those other threads to help scan and mark?
   - or, just make the current GC thread parallel by spawning new threads if that can work, maybe idk
 - 

## Examples

### Passing data to a bunch of threads
<!-- something with GcMut to initialize and demotion to Gc to share to threads -->

### Simple linked list (without `unsafe` or refcounting)
<!-- just use the test code -->

### \[idk something with tokio and async\]
<!-- Gc<Mutex<T>> probably -->

