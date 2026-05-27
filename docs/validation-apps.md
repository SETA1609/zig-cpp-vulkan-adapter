# Validation apps — zig-cpp-vulkan-stack-adapter

> Small standalone apps that consume this adapter via `build.zig.zon` **exactly the way the engine will** — to validate the public Zig API (`vk` re-export, volk, VMA, shaderc, the per-OS surface creators) *before* the engine exists. Each is a throwaway project in its own directory/repo, not part of the engine.
>
> **Complementary to the reference-C++-host track** ([zVoxRealms `external-libs-catalog.md` § 5.5](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/external-libs-catalog.md)). That track drops the adapter into a reference C++ host to validate the *C ABI + build wiring* against real workloads. A C++ host uses its own renderer, so it **cannot** exercise *this adapter's* idiomatic Zig wrappers (`vk_stack.vma`, `vk_stack.shaderc`) the way the engine will — these mini-apps are the only thing that does.
>
> Roadmap + version gates: [`ROADMAP.md`](ROADMAP.md). Sprint: [`sprint.md`](sprint.md).

## Completion checklist

Mark `[x]` only when the app **builds and runs correctly** — not merely compiles. `[~]` = in progress.

- [ ] **Headless triangle → PPM** — offscreen render, no window, dump raw pixels · *vulkan v0.3.0 + v0.4.0 (no surface, no platform)*
- [ ] **`nm` decoupling check** — headless-triangle binary shows **zero `SDL_*` symbols** · *vulkan v0.3.0+*
- [ ] **Reactive clear-color** — swapchain clear/present, color from input (paired with platform-stack) · *vulkan v0.2.0 + platform v0.6.0*
- [ ] **Snake** — VMA quad buffer + ortho projection + draw loop (paired) · *vulkan v0.3.0 + platform v0.6.0*
- [ ] **Breakout** — many quads → instancing/batching throughput through VMA (paired) · *vulkan v0.3.0 + platform v0.6.0*
- [ ] **Conway's Life** — fullscreen grid via a real fragment/compute shader (paired) · *vulkan v0.4.0 + platform v0.6.0*

## The ladder — what each app validates

| App | Needs | Validates (for this adapter) | Paired? |
| --- | --- | --- | --- |
| **Headless triangle → PPM** | vulkan v0.3.0 + v0.4.0 | Proves the adapter is **fully standalone**: instance (no surface) → device → offscreen framebuffer → VMA buffer → shaderc shader → readback → PPM. No windowing dep at all. | no |
| **Reactive clear-color** | + platform v0.6.0 | `vk` instance + the per-OS surface creator (via the engine bridge) → swapchain → per-frame clear → present; swapchain-recreate on resize. **The key proof the decoupled pairing works end to end.** | yes |
| **Snake** | + platform v0.6.0 | VMA vertex/index buffer (one quad), push-constant for per-cell color/position, ortho projection, the full present loop. (precompiled SPIR-V is fine — defer shaderc) | yes |
| **Breakout** | + platform v0.6.0 | Many quads at once → instancing / batching throughput through VMA. Catches allocation-churn or descriptor issues a single quad won't. | yes |
| **Conway's Life** | + platform v0.6.0 | A genuine fragment/compute shader + large dynamic buffer churn. **The best shaderc stress test** (needs vulkan v0.4.0). | yes |

> The cube is intentionally absent — a spinning textured cube is the engine's own Phase 1 milestone ([parent `sprint.md` § E](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/sprint.md)). Standalone tests stay 2D so they don't pre-build the engine's first deliverable.

## Required decoupling check (`nm`)

The architecture rests on this adapter dragging **no windowing**. After building the **Headless triangle**:

```sh
nm <headless-triangle-binary> | grep -i 'SDL_\|x11\|wayland\|glfw'   # must print NOTHING
```

A non-empty result means a windowing symbol leaked across the boundary — fix it now, while it's a ~200-line app.

## Discipline

Per [zVoxRealms `docs/guard.md`](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/guard.md): **you write these apps by hand** (learning project). They live outside the engine tree and depend on this adapter (and, for paired apps, the [platform-stack adapter](https://github.com/SETA1609/zig-cpp-platform-stack-adapter)) via pinned `build.zig.zon` entries — the same consumption pattern the engine uses. Every `extern "C"` bridge crossed here (VMA, shaderc) must stay `noexcept` per [`docs/cpp-style.md`](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/cpp-style.md).
