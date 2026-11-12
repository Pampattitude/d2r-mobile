# Experimental Adreno 650 fragment-subgroup rewrite

The Adreno 650 reports subgroup operations only for compute shaders, while bundled fragment shaders contain translated `ShuffleXor` helpers. Runtime failures identified `ps_models_p.bin` (`5c1ca4e7c3e4657c`), `ps_alpha_threshold_p.bin` (`358197ac3db59cbc`), and a later module `b70c638ef69450c5`. A pack-wide scan found 182 fragment modules with subgroup instructions.

An opt-in experiment rewrites all 182 audited fragment subgroup modules to fragment derivatives, then removes the now-unreferenced helper functions with SPIRV-Tools. This deliberately relaxes the rewriter's control-flow and numeric bit-flow checks. It can change rendered values or produce artifacts; it is not a general Adreno 650 compatibility claim.

The experiment is disabled by default. To try it in a debug build, create the marker file `files/dm/gfx-adreno650-fragment-subgroups` in that app's private storage. It is honored only on Qualcomm Adreno 650 and applies to fragment modules containing subgroup operations. Remove the marker to disable it. The final rewritten module still goes through normal Vulkan capability validation; the stage check is not bypassed.

The CPU regression tests and offline SPIR-V validation establish that all 182 rewritten modules are structurally valid and contain no subgroup instructions after dead-function elimination. Only testing on an Adreno 650 can establish whether its driver accepts the modules and whether the resulting picture is usable.

On the connected Adreno 650, version 2.3.1 advanced past `5c1ca4e7c3e4657c` after capability 51 (`DerivativeControl`) was added to the Vulkan core allowlist, then failed on `358197ac3db59cbc`. Version 2.3.2 included that shader but exposed `b70c638ef69450c5`. Since the full pack audit confirmed all 182 fragment subgroup modules can be rewritten and validated offline, version 2.3.3 applies the opted-in workaround to all such modules. Device startup and visual correctness still require confirmation.
