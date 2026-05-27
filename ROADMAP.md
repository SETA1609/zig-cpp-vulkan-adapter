# Roadmap — zig-cpp-vulkan-stack-adapter

> The versioned plan for this adapter, which bundles **vulkan-zig + volk + VMA + shaderc** version-pinned together in one sub-repo. Each version maps to a [zVoxRealms](https://github.com/SETA1609/zigVoxelWorlds) phase. Sprint-level breakdown: [`sprint.md`](sprint.md).
>
> **Why bundled:** VMA's headers embed specific Vulkan-1.x signatures, vulkan-zig's bindings come from a specific `vk.xml` snapshot, and shaderc emits SPIR-V for a specific Vulkan version — they must move together or you get cryptic runtime errors. One `build.zig.zon` enforces atomic version coherence. Full rationale: [`external-libs-catalog.md` § Note on the Vulkan-stack meta-package](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/external-libs-catalog.md).

## Bundled libs

| Lib | License | Surfaces as | Real at |
| --- | --- | --- | --- |
| [vulkan-zig](https://github.com/Snektron/vulkan-zig) (Snektron) | MIT | `pub const vk = @import("vulkan")` re-export — no C-ABI tax; typed enums, error sets, comptime dispatch | v0.1.0 |
| [volk](https://github.com/zeux/volk) | MIT | `vk_stack.volk` — loader / function-pointer table | v0.2.0 |
| [VMA](https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator) (AMD) | MIT | `vk_stack.vma` — GPU memory; `extern "C"` bridge → idiomatic Zig | v0.3.0 |
| [shaderc](https://github.com/google/shaderc) (Google, over glslang BSD-3) | Apache-2.0 | `vk_stack.shaderc` — GLSL→SPIR-V; `extern "C"` bridge → idiomatic Zig | v0.4.0 |

Plus per-OS **surface creators** (`createX11Surface`/`createWaylandSurface`/`createWin32Surface`/`createAndroidSurface`) — each takes raw OS primitives, **no import from the platform adapter** (per [platform spec Rule 2](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/specs/platform.md)).

## Version milestones

| Version | Scope | Unblocks (zVoxRealms) |
| --- | --- | --- |
| **v0.1.0** | `vk` re-export working; volk / VMA / shaderc + surface creators stubbed panic-on-call | Phase 1 / Sprint 1 § B — engine `@import("vulkan_stack").vk` compiles |
| **v0.2.0** | volk loader real + per-OS surface creators real (X11 + Win32) | Phase 1 § D — Vulkan instance + surface + clear screen |
| **v0.3.0** | VMA wrapper real — `createBuffer` / `createImage` / lifecycle | Phase 1 § E — cube vertex/index buffers |
| **v0.4.0** | shaderc wrapper real — `compile(glsl, stage)` → SPIR-V | Phase 1 § E — cube shaders (deferrable: cube can ship precompiled SPIR-V) |
| **v0.5.0** | Wayland + Android surface creators; full per-OS coverage | Phase 4+ / Android port |
| **v1.0.0** | Full stack stable; version-coherence pin documented; CI across targets; tree-shake verified (`nm` shows no off-target surface symbols) | Phase 13 (ship) |

Versions v0.1.0 and v0.2.0 are anchored by the parent [`sprint.md` § B.3 and § D.1](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/sprint.md). v0.3.0+ are this adapter's own continuation.

> **Note on volk vs. vulkan-zig:** vulkan-zig generates its own dispatch wrappers from `vk.xml`, which overlaps volk's loader role. volk stays in the lineup per the catalog, but whether it's still needed once vulkan-zig's dispatch is wired is worth revisiting at v0.2.0 — if redundant, drop it and reclaim a vendored dep.

## Deliberately NOT here

| Item | Where it lives | Why |
| --- | --- | --- |
| Windowing | sibling [zig-cpp-platform-stack-adapter](https://github.com/SETA1609/zig-cpp-platform-stack-adapter) | Orthogonal to Vulkan |
| SPIRV-Reflect | engine-side `@cImport` | Pure C, not Vulkan-version-coupled |
| SPIRV-Cross | deferred | Metal/D3D transpile, post-v1.0 |
| Frame graph / material pipeline | engine `src/render/` | Engine code, not a third-party lib |

## C++ boundary discipline

Every `extern "C"` bridge function (VMA, shaderc) is `noexcept` and catches all exceptions before they cross the C ABI. Follows [zVoxRealms `docs/cpp-style.md`](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/cpp-style.md).

## Cross-reference

- Catalog entry + bundling rationale: [`external-libs-catalog.md` § 3 Vulkan-stack](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/external-libs-catalog.md)
- Surface-creation contract: [`docs/specs/platform.md` § Rule 2](https://github.com/SETA1609/zigVoxelWorlds/blob/main/docs/specs/platform.md)
- Sibling adapter: [zig-cpp-platform-stack-adapter](https://github.com/SETA1609/zig-cpp-platform-stack-adapter)
- Sprint plan: [`sprint.md`](sprint.md)
