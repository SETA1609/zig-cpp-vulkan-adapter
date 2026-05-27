# Vision — zig-cpp-vulkan-stack-adapter

> One Zig dependency that delivers the **whole Vulkan stack, version-coherent** — vulkan-zig + volk + VMA + shaderc moving together — surfaced as idiomatic Zig, never a raw C ABI.

## The north star

A consumer adds **one** `build.zig.zon` entry and gets typed Vulkan bindings, a GPU memory allocator, a loader, a shader compiler, and per-OS surface creators — all pinned to versions known to agree. The bundling exists to make that agreement **atomic**: bump the sub-repo, and all four move as one. No more "VMA expects a Vulkan signature vulkan-zig didn't generate."

## The Vulkan destination for a staged migration

This adapter is the **target** a renderer migrates *to*. A reference C++ host with an OpenGL renderer can adopt this stack incrementally — building Vulkan paths against it while the GL renderer keeps shipping (the windowing floor stays constant via the sibling [platform-stack adapter](https://github.com/SETA1609/zig-cpp-platform-stack-adapter), which serves both GL and Vulkan). When the Vulkan path reaches parity, the host flips over. This adapter's job is to make that destination as low-friction as a single dependency.

## Standalone — no windowing dependency

Surface creators take **raw OS primitives**, not a windowing type. So this adapter is usable with any window source — the platform-stack adapter, SDL directly, raw X11, or none at all (headless / offscreen). Enforced by the `nm` check in [`validation-apps.md`](validation-apps.md): a headless binary shows zero `SDL_*` symbols.

## Non-vision

- Windowing / input — sibling [platform-stack adapter](https://github.com/SETA1609/zig-cpp-platform-stack-adapter).
- Frame graph, render passes, materials — engine code (`src/render/`).
- Metal / D3D backends — deferred (SPIRV-Cross, post-v1.0).

See [`mission.md`](mission.md) for the concrete commitments and [`ROADMAP.md`](ROADMAP.md) for the version sequence.
