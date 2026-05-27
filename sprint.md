# Sprint 1 — v0.1.0 → v0.4.0: Vulkan stack for the Phase 1 cube

> First real wrapping work in this sub-repo, spanning four tagged releases that progressively unblock [zVoxRealms](https://github.com/SETA1609/zigVoxelWorlds) Phase 1 (window → clear screen → rotating cube). Roadmap: [`ROADMAP.md`](ROADMAP.md).
>
> **Sprint goal:** the engine renders a rotating cube using `vk_stack.{vk, volk, vma, shaderc}` + the per-OS surface creators.
>
> **Definition of done:** `v0.1.0`–`v0.4.0` tagged; the engine's `src/render/surface.zig` bridge resolves and produces a non-null surface; `nm` on a Linux build shows no Windows/Android surface symbols; CI green on `x86_64-linux-gnu` + `x86_64-windows-gnu`.

Each `[ ]` maps to one atomic sub-repo commit per [zVoxRealms commit rules](https://github.com/SETA1609/zigVoxelWorlds/blob/main/CONTRIBUTING.md).

## § A — v0.1.0: vk re-export (parent Sprint 1 § B.3)

- [ ] **V1.1** `build.zig.zon`: add [vulkan-zig](https://github.com/Snektron/vulkan-zig) as a pinned dependency.
  - Files: `build.zig.zon`
  - Commit: `chore(zon): add vulkan-zig dependency (pinned)`

- [ ] **V1.2** `build.zig`: drop the hello-world executable; expose a `vulkan_stack` module; wire vulkan-zig's `vk.xml` codegen.
  - Files: `build.zig`
  - Acceptance: `zig build` produces a static lib exporting `vulkan_stack`
  - Commit: `feat(build): expose vulkan_stack module + wire vulkan-zig codegen`

- [ ] **V1.3** `src/root.zig`: `pub const vk = @import("vulkan");` + stub `vma` / `volk` / `shaderc` and the per-OS surface creators as panic-on-call.
  - Files: `src/root.zig`, `src/surface.zig`
  - Acceptance: a consumer can reach `vk.Instance`, `vk.SurfaceKHR`
  - Commit: `feat(api): re-export vk; stub vma/volk/shaderc/surface (panic-on-call)`

- [ ] **V1.4** Tag `v0.1.0`; push. Parent engine wires this dep (parent `sprint.md` § B.4).

## § B — v0.2.0: loader + surface creators (parent § D.1)

- [ ] **V2.1** Vendor volk under `vendor/volk/` (submodule); `src/volk.zig` thin Zig wrapper loading `vkGetInstanceProcAddr` + instance/device tables. *(Skip if V0.2.0 review concludes vulkan-zig's own dispatch suffices — see ROADMAP note.)*
  - Files: `vendor/volk/`, `src/volk.zig`
  - Commit: `feat(volk): real loader wrapper`

- [ ] **V2.2** `src/surface.zig`: real `createX11Surface` + `createWin32Surface` calling `vkCreate{Xlib,Win32}SurfaceKHR`. Wayland/Android stay stubbed until v0.5.0.
  - Files: `src/surface.zig`
  - Acceptance: a valid `vk.Instance` + raw display/window pointers → non-null `vk.SurfaceKHR`; validation layer clean
  - Commit: `feat(surface): implement createX11Surface + createWin32Surface`

- [ ] **V2.3** Tag `v0.2.0`; push. Parent commit: `chore(deps): bump vulkan-adapter → v0.2.0 (loader + surface creators)`

## § C — v0.3.0: VMA (parent § E.1 — cube buffers)

- [ ] **V3.1** Vendor VMA under `vendor/VMA/` (submodule); `src/c/vma_bridge.{h,cpp}` — `extern "C"` bridge, every boundary fn `noexcept` and catching before crossing the C ABI (per `cpp-style.md`).
  - Files: `vendor/VMA/`, `src/c/vma_bridge.h`, `src/c/vma_bridge.cpp`
  - Commit: `feat(vma): extern C bridge over VulkanMemoryAllocator`

- [ ] **V3.2** `src/vma.zig`: idiomatic Zig wrapper — `createBuffer` / `createImage` / `destroyBuffer` + allocator lifecycle.
  - Files: `src/vma.zig`
  - Acceptance: allocate + free a vertex buffer; no validation-layer complaints
  - Commit: `feat(vma): idiomatic Zig wrapper — createBuffer/createImage`

- [ ] **V3.3** Tag `v0.3.0`; push.

## § D — v0.4.0: shaderc (parent § E.1 — shaders; deferrable)

> Optional for the cube: the parent sprint allows precompiled SPIR-V embedded in Zig source. Do this section only when you want runtime GLSL compilation.

- [ ] **V4.1** Vendor shaderc under `vendor/shaderc/` (submodule, Apache-2.0 over glslang BSD-3); `src/c/shaderc_bridge.{h,cpp}`.
  - Files: `vendor/shaderc/`, `src/c/shaderc_bridge.h`, `src/c/shaderc_bridge.cpp`
  - Commit: `feat(shaderc): extern C bridge over shaderc`

- [ ] **V4.2** `src/shaderc.zig`: `compile(allocator, source, stage)` → SPIR-V bytes.
  - Files: `src/shaderc.zig`
  - Acceptance: a trivial `.vert` compiles to valid SPIR-V
  - Commit: `feat(shaderc): idiomatic Zig wrapper — compile GLSL→SPIR-V`

- [ ] **V4.3** Tag `v0.4.0`; push.

## § E — Docs + CI (alongside the above)

- [ ] **V5.1** `README.md`: flip Status to "Phase 1 — wrapping in progress"; update the build section (static lib, not an executable).
  - Files: `README.md`
  - Commit: `docs(readme): Phase 1 status; library not executable`

- [ ] **V5.2** CI: build the adapter on `x86_64-linux-gnu` + `x86_64-windows-gnu`; `zig fmt --check` + `clang-format --dry-run -Werror` on `src/c`.
  - Files: `.github/workflows/build.yml`
  - Commit: `ci: build vulkan adapter + lint C bridges on linux + windows`

## What you write yourself vs. what AI helps with

Per [zVoxRealms `docs/guard.md`](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/guard.md) — this is a learning project:

- **You write all `.zig` / `.c` / `.cpp` wrapping code by hand** (the volk / VMA / shaderc bridges, the surface creators).
- **AI helps with:** reviewing your code, debugging compile errors you paste, drafting `build.zig.zon` / CI / `.clang-format` config, documentation, scaffolding empty files.
- **AI does not write:** the C bridges, the Zig wrappers, or the surface-creation calls.
