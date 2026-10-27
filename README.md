# TopSpin 2K25 - Emulation & Reverse Engineering Progress

This repository tracks my technical progress, patches, and shader recompiler investigations for running **TopSpin 2K25** (Hangar 13 / 2K Czech *Hammer Engine*) on PC via open-source PlayStation emulation (`shadPS4` for PS4 `CUSA32748` and custom `KytyPS5` source builds for PS5 `PPSA06899`).

## Repository Contents

* `topspin2k25_shader_fixes.patch`: Local patches for `shadPS4` covering FMask storage writes, PC-relative `ReadConst` inline tables, and graph-based unreachable block pruning.
* `kytyps5_topspin2k25_fixes.patch`: Patches for `KytyPS5` implementing a `D16 -> D24S8` depth promotion texture cache copy path and 1080p display resolution overrides.
* `fs_0x000000006b13f853.asl.txt`: Abstract Shader Language (ASL) dump of the fragment shader currently under investigation.

---

## Technical Breakdown (shadPS4 Patches)

1. **FMask Storage Writes (`resource_patching_pass.cpp`):**
   * Bypasses strict assertions on FMask write operations by invalidating unused instructions or replacing them with a zero immediate (`inst.Invalidate()` / `ReplaceUsesWith(0)`), allowing complex pipeline compilation to proceed.

2. **PC-Relative Inline Tables in `ReadConst` (`flatten_extended_userdata_pass.cpp` & `resource_discover_pass.cpp`):**
   * Excludes non-SRT inline constant table reads (such as `GetPcLo`-based addresses) from the extended userdata flattening pass and sharp source discovery to prevent translation aborts.

3. **Graph-Based Unreachable Block Pruning (`control_flow_graph.cpp`):**
   * Replaced the standard single-pass check (`num_predecessors == 0`) with a graph reachability traversal (`std::unordered_set` BFS/DFS worklist) from the entry block (`0x000`). This ensures orphaned sub-blocks properly decrement child `num_predecessors` without corrupting the SPIR-V generator.

---

## Current Status & Next Steps

* **KytyPS5:** Boots successfully and reaches in-game matches at Wimbledon, but requires memory optimizations (`KYTY_FAST_GC`) and resolution constraints (`KYTY_OUTPUT_RES_1080`) to handle 6GB VRAM limits gracefully.
* **shadPS4:** Initializes `sceVideoOut` in 1080p, loads all Hammer Engine `.prx` modules, compiles over 30 pipelines, and reaches the `Display2DThin_8 tiler` while streaming Indian Wells audio. Current investigations focus on resolving PC-relative `S_SETPC_B64` jump tables in fragment shaders (like `0x6b13f853`) to prevent block truncation and `EnsureResident` synchronization waits.

## Contributions & Collaboration
Feedback, suggestions, or pull requests from developers interested in shader recompiler architectures and orbis kernel emulation are highly welcome!
# TopSpin2K25-Emulation-Progress