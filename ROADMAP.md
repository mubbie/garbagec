# garbagec — Learning Roadmap

A garbage collection library for C, built incrementally as a learning project.

## Project Setup

### Structure

```
garbagec/
├── CMakeLists.txt
├── README.md
├── ROADMAP.md
├── include/
│   └── garbagec/
│       ├── garbagec.h         # Public API (shared interface)
│       ├── refcount.h         # Reference counting API
│       └── marksweep.h        # Mark-and-sweep API
├── src/
│   ├── refcount.c
│   └── marksweep.c
├── tests/
│   ├── test_refcount.c
│   └── test_marksweep.c
└── examples/
    ├── refcount_demo.c
    └── marksweep_demo.c
```

### Build

Using CMake:

```bash
mkdir build && cd build
cmake ..
make
```

---

## Phase 1: Foundation

**Goal:** Get the project scaffolding in place and understand the basics.

- [ ] Set up the repo, CMakeLists.txt, and directory structure
- [ ] Implement a simple memory allocator wrapper (`rc_alloc`, `rc_free`) that wraps `malloc`/`free` and tracks total bytes allocated/freed
- [ ] Add basic logging/stats so you can see allocations and frees happening
- [ ] Write a small test harness (or use a lightweight C test framework)

**What you'll learn:** How `malloc`/`free` work under the hood, how to track allocations, and how to structure a C library.

---

## Phase 2: Reference Counting

**Goal:** Implement a working reference-counted memory manager.

### 2a — Basic reference counting

- [ ] Define an object header struct that stores the reference count alongside the user's data
- [ ] Implement `rc_alloc(size)` — allocates an object with refcount initialized to 1
- [ ] Implement `rc_retain(obj)` — increments the refcount
- [ ] Implement `rc_release(obj)` — decrements the refcount; frees the object when it hits 0
- [ ] Write tests: allocate, retain, release, verify memory is freed at the right time

### 2b — Poking at the edges

- [ ] Demonstrate the circular reference problem: create two objects that point to each other, show that they leak
- [ ] Add a destructor/cleanup callback so objects can release their children when freed
- [ ] Track total live objects and bytes to confirm leaks vs. clean collection

### 2c — Cycle detection (stretch goal)

- [ ] Implement a simple cycle detector (trial deletion or tracing-based) to break reference cycles
- [ ] Write tests with circular structures and verify they're collected

**What you'll learn:** How reference counting works in practice, why circular references are a problem, and the overhead of maintaining counts on every pointer operation.

---

## Phase 3: Mark-and-Sweep

**Goal:** Implement a tracing garbage collector.

### 3a — Object model and root set

- [ ] Define a GC-managed object header: mark bit, size, pointer to next object (for the heap list), and a way to enumerate an object's references
- [ ] Maintain a linked list (or array) of all allocated objects so the sweep phase can iterate the heap
- [ ] Define a root set: a simple array/stack of pointers the GC treats as roots
- [ ] Implement `gc_alloc(size)` and `gc_add_root(obj)` / `gc_remove_root(obj)`

### 3b — Mark phase

- [ ] Implement the mark phase: starting from roots, recursively (or iteratively with a worklist) traverse object references and set the mark bit
- [ ] Decide how objects describe their references — options:
  - A callback per object type (like a `trace` function pointer)
  - A fixed layout descriptor (offsets of pointer fields)
  - Conservative scanning (treat every word as a potential pointer)
- [ ] Write tests: create a reachable object graph, run mark, verify correct objects are marked

### 3c — Sweep phase

- [ ] Walk the heap list; free any object that isn't marked
- [ ] Clear mark bits on surviving objects
- [ ] Write tests: create reachable and unreachable objects, run collect, verify unreachable ones are freed and reachable ones survive

### 3d — Putting it together

- [ ] Implement `gc_collect()` that runs mark + sweep
- [ ] Add a trigger: collect automatically when total allocations exceed a threshold
- [ ] Test with circular references — verify they're collected (unlike reference counting)
- [ ] Add stats: objects scanned, objects freed, time spent in GC

**What you'll learn:** How tracing works, the challenge of identifying roots and references in C, stop-the-world pauses, and why mark-and-sweep handles cycles for free.

---

## Phase 4: Refinement and Exploration (Stretch Goals)

Pick and choose based on what interests you.

### Compacting / copying collector
- [ ] Implement a semi-space collector: divide memory into two halves, copy live objects from one to the other
- [ ] Observe how this eliminates fragmentation and enables bump-pointer allocation

### Generational collection
- [ ] Split the heap into young and old generations
- [ ] Collect the young generation frequently, promote survivors to old
- [ ] Implement a simple write barrier to track old-to-young references

### Performance benchmarks
- [ ] Write allocation-heavy benchmarks (e.g., building and tearing down large linked lists or trees)
- [ ] Compare reference counting vs. mark-and-sweep on throughput, pause times, and peak memory usage
- [ ] Profile with `valgrind` or `perf`

### Conservative stack scanning
- [ ] Scan the C stack for values that look like heap pointers (the Boehm GC approach)
- [ ] Explore the tradeoffs: no manual root registration, but potential for false positives

---

## Resources

- [Baby's First Garbage Collector](https://journal.stuffwithstuff.com/2013/12/08/babys-first-garbage-collector/) — Bob Nystrom's walkthrough of a tiny mark-and-sweep GC in C
- [The Garbage Collection Handbook](https://gchandbook.org/) — the comprehensive reference on GC algorithms
- [Boehm-Demers-Weiser GC](https://www.hboehm.info/gc/) — production conservative GC for C/C++, great for studying design decisions
- [Crafting Interpreters, Ch. 26](https://craftinginterpreters.com/garbage-collection.html) — implementing a GC for a language VM, very practical
- [Writing a Simple Garbage Collector in C](https://maplant.com/2020-04-25-Writing-a-Simple-Garbage-Collector-in-C.html) — another hands-on walkthrough
