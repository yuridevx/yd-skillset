---
name: msvcpp-toolkit
description: C++ language toolkit, MSVC/Windows-first — the language-specific half of the algo-rewrite and harden skills. Supplies, in named sections, toolchain detection (cl vs clang-cl, /arch, vcpkg triplets), benchmark harness order (Google Benchmark first) with Windows measurement hygiene and alloc-guard instrumentation, hot-path hidden-allocator list and error model, SIMD (highway-first, clang-cl for hot TUs) and concurrency (oneTBB/Taskflow, PPL is frozen legacy) facilities, the library-first shortlist with maintenance-health flags, boundary enforcement on Windows (bounds-checked STL hardening, checked views, asserted preconditions, ASan-backed test builds — no fuzzing engines), and the per-defect-class hardening oracle table (MSVC ASan; UBSan via clang-cl; TSan/MSan/LSan need a WSL leg). PASSIVE reference — invoked by algo-rewrite and harden when the target language is C++; also usable directly when choosing C++ performance or hardening tooling on Windows. Trigger on "msvcpp-toolkit", "/msvcpp-toolkit", "msvc toolkit", or as the C++ toolkit for algo-rewrite/harden.
---

# msvcpp-toolkit

Language toolkit for C++, consumed by **algo-rewrite** and **harden**. **MSVC/Windows-first**: recommendations assume Visual Studio 2022/2026 with cl.exe and clang-cl available; pure clang/gcc-on-Linux projects can take the same first picks minus the MSVC caveats. Each `##` section below is part of the toolkit contract those skills reference by name; a new `<lang>-toolkit` (csharp-toolkit, rust-toolkit, …) must provide the same eight sections.

Recommendations are ordered: take the first that installs on the project's toolchain; ties go to what the project already depends on. Maintenance-health notes are as of July 2026, verified against release pages — re-verify anything flagged before adopting.

## Toolchain

Detect and record in the ledger header: MSVC version (VS 2022 17.x / VS 2026 18.x — several libraries now gate on this; VS 2026 toolsets are serviced only **9 months each**, 14.51 is the default from 18.6, and side-by-side toolsets 14.30–14.43 are installable — bleeding-edge libraries care which rev you're on), whether **clang-cl** is installed (it is the escape hatch for UBSan and SIMD-heavy hot TUs — MSVC-ABI-compatible, per-TU mixing is routine, one linker per binary, never mix LTO across compilers), C++ standard, `/arch` level actually enabled (MSVC defines no `__SSE__`/`__AVX__`/`__FMA__` macros; without `/arch:AVX2` expect non-VEX/VEX mixing penalties; `/arch:AVX10.2` is new in VS 2026 — 256-bit default vector length, `/vlen=512` to widen; clang-cl gets AVX10 parity in LLVM 22), build system, CPU (cores, **P/E hybrid?**, caches), package manager.

Package manager: **vcpkg** (manifest mode + `builtin-baseline`; binary caching via NuGet/GitHub Packages — the old `x-gha` provider is removed; `x-azcopy` is the experimental Azure Blob provider) → Conan 2 (healthy, ~monthly releases) → CPM/FetchContent. Triplet rules: `/MD`-family (`x64-windows`, `x64-windows-static-md`) for normal and **ASan** builds — never debug-CRT+ASan via vcpkg.

`compile_commands.json` on Windows requires the **Ninja** generator — VS generators don't export it, still true in CMake 4.x (matters for clang-tidy).

## Benchmarking

Harness: **Google Benchmark** (healthy, v1.9.5 Jan 2026; MSVC in CI including VS 2026 runners; statistical repetitions, `DoNotOptimize`, `compare.py`, `RegisterMemoryManager`). Use `--benchmark_enable_random_interleaving --benchmark_repetitions=9` to cut Windows variance; keep CRT flavor consistent and set `BENCHMARK_STATIC_DEFINE` correctly or expect link errors. nanobench is **dormant** (no release since 2023, last commit late 2024) — acceptable as a frozen single-header tool, not for anything long-lived. Catch2 `BENCHMARK` only if Catch2 is already the test framework.

**Windows measurement hygiene** (numbers are noise without this): High Performance power plan; pin the bench thread to one **P-core** via `SetThreadAffinityMask` (hard affinity is the only reliable isolation on hybrid Intel; CPUSETs are soft hints); `SetPriorityClass(HIGH_PRIORITY_CLASS)` (not REALTIME); `QueryPerformanceCounter` for timing; beware Win11 EcoQoS down-clocking low-priority processes; add the bench binary to Defender exclusions. New noise source since June 2026: the **KB5094126 "Low Latency Profile"** (24H2/25H2, staged rollout) boosts CPU clocks for 1–3 s on foreground actions — short benches may catch or miss the boost window nondeterministically; pin affinity and run long enough to amortize. Google Benchmark's CPU-scaling warning is weaker on Windows — absence of the warning guarantees nothing.

Instrumentation: generate `bench/algo_bench.h` with `ALGO_BENCH_SCOPE(name)`, `ALGO_BENCH_COUNT(name, n)`, `ALGO_BENCH_ALLOC_GUARD(name)`, gated on `ALGO_BENCH=1`; without the flag they expand to **nothing** — no branch, no atomic, no string — so they may live in product code permanently.

Allocation tracking: hand-rolled global `operator new`/`operator delete` counting hook with thread-local counters (~30 lines) — still the norm, no established library exists; wire it into the alloc guard and, with Google Benchmark, into `RegisterMemoryManager` so allocs/bytes land in the JSON output.

Macro profiling stack, in order of reach-for: VS Profiler (zero setup) → **Tracy** (free, active — v0.13.x; frame/zone instrumentation; bus-factor-1) or Superluminal (commercial) → Intel VTune (deepest PMU; 2026 versions need ≥11th-gen Intel for µarch metrics and dropped VS2019 integration) → WPA/ETW (system-level interference). Optick is abandoned — do not adopt.

## Hot-path rules

Hidden allocators to avoid on hot paths: `std::function` beyond SBO, `std::string` temporaries, `std::vector`/`std::deque` growth, `unordered_map` rehash, `shared_ptr` control blocks, iostreams, `std::regex`, `throw`/`catch`, coroutine frames that fail HALO.

Error model: errors by value — `std::expected`/`std::optional`/status codes; exceptions are setup/teardown territory. Layout is fully under your control: contiguous over node-based, SoA for SIMD, `alignas` padding against false sharing. Debug-vs-release trap: `_ITERATOR_DEBUG_LEVEL` is ABI-breaking (all TUs must agree) and makes debug-build hot-path numbers meaningless — benchmark release only.

## SIMD / data-parallel

**Do not trust cl.exe auto-vectorization for hot loops** — it trails LLVM (no real gather/scatter, FP reductions need `/fp:fast`, conservative aliasing); a new SLP vectorizer is landing through the 2025–26 previews and 14.51 shipped NEON-vectorized STL algorithms on ARM64, so it is improving — but the verdict stands; `/Qvec-report:2` tells you what failed but cryptically. The reliable patterns, in order:

1. **google/highway** — runtime dispatch (SSE4/AVX2/AVX-512/NEON) from one compile, battle-tested (Chromium, NumPy); MSVC is tested per the README though the public CI is Linux-only. On MSVC build with `/arch:AVX2` + `/Gv`; Windows ARM64 under cl.exe is unsupported (clang-cl there); the AVX10.2 target needs GCC 16/Clang 22 — no MSVC path. The project itself treats clang-cl as the happier Windows path.
2. **Compile hot TUs with clang-cl** inside the otherwise-MSVC project — the established pattern (Chrome/Firefox ship this way); simdjson-class code runs ~2× faster than under cl.
3. **xsimd** — nicer fixed-ISA API; needs explicit `/arch:*`; **ARM64/NEON broken under cl.exe** (issue still open mid-2026 — clang-cl required there).

Avoid: EVE (still no MSVC support, no Windows CI), `std::simd` (C++26 — **not started in MSVC STL**; partial in libstdc++ since GCC 16.1, usable for a WSL leg via GSI-HPC/simd; revisit MSVC ~2027+).

## Concurrency

**oneTBB** (healthy under UXL Foundation, v2023.0; MSVC CI including VS 2026 and Windows-ARM) for parallel loops/reductions/concurrent containers. **Taskflow** (active, v4.x; bus-factor concentrated in one author) when the problem is a dependency DAG. MSVC's C++17 parallel STL (built on the Windows Thread Pool) is a fine drop-in for `sort(par, …)`-class work — note it deliberately keeps some algorithms serial. Raw Windows Thread Pool API as the zero-dependency baseline; **BS::thread_pool** when you want a simple pool without adopting oneTBB. **PPL/ConcRT is frozen legacy** (docs untouched since 2018; no official deprecation banner) — fine to keep, wrong to adopt. Lock-free queues: moodycamel/concurrentqueue (battle-tested; bus-factor-1, slow but alive — v1.0.5 Apr 2026), boost.lockfree (back under active development — `mpsc_weak_queue`/`bounded_ticket_queue` landing), or max0x7ba/atomic_queue. stdexec/P2300: **now builds under cl ≥14.43 with Windows CI** — but open MSVC codegen bugs remain and no standard library ships `std::execution`; mention, don't adopt yet.

## Library shortlist

| Domain | First pick | Health / MSVC notes |
| --- | --- | --- |
| Hash map | `boost::unordered_flat_map` | Healthy (Boost 1.91, deliberate maintenance-stable), excellent MSVC. Fallback `ankerl::unordered_dense` (active, single-maintainer; contiguous, iteration-heavy) |
| Hash function | rapidhash (V3) | Healthy, now vendored by Chromium; uses `_umul128` on MSVC x64 properly; fastest SMHasher3-passing hash. Fallback XXH3 (slow cadence, canonical) |
| JSON (read-heavy) | simdjson | Healthy, VS 2026 CI; **MSVC codegen ~40–100% slower than clang-cl — build hot JSON TUs with clang-cl**. Pin carefully (v4.6.2 was a breaking patch) |
| JSON (serialize/deserialize structs) | Glaze | Hyperactive but **bus-factor-1 and MSVC-bleeding-edge**: floor is VS 2026 (14.50), VS 2022 effectively dropped, `/Zc:preprocessor` mandatory; ICE churn is stabilizing on VS 2026 but each toolset rev is still a gamble. On VS 2022: **reflect-cpp** (corporate-backed, C++20, MSVC-friendly) or pin an old Glaze. Cold convenience paths: nlohmann |
| Formatting | fmt | Healthy (v12.2), excellent MSVC; `std::format` fine on fully-C++23 toolchains. Float→string hot paths: **zmij** (fmt author; already in Glaze, slated to replace Dragonbox in fmt) |
| Global allocator | mimalloc | Healthy, Microsoft-hosted (but solo-maintained); v3.3.x is the recommended line, v2.3.x for max soak. **Prefer explicit `mi_*` or `mimalloc-new-delete.h` override; the Windows redirect DLL is closed-source and fragile against AV/EDR.** jemalloc revived under Meta (5.3.1, Apr 2026) — cadence unproven, still not Windows-first. snmalloc (Microsoft, active, header-only) is the research-grade alternative |
| Compile-time const maps | frozen | Dormant/maintenance-mode — stable, fine as a frozen dependency. No MSVC-ready successor (qlibs/mph is gcc/clang-only) |

abseil: only if already in-tree (boost/ankerl beat it now). folly: cherry-pick ideas, never adopt as a dependency (vcpkg port unstable on Windows).

## Boundary enforcement

Enforce first, detect second — order for MSVC-built code:

1. **Bounds-checked types at the boundary itself:** `std::span`/`gsl::span` with `.at()` (never raw `[]`/pointer arithmetic) for any buffer crossing a trust boundary. **`_MSVC_STL_HARDENING=1`** (17.14+, release-safe, fastfail; Microsoft plans default-on post-17.x) turns `vector`/`span`/`string_view`/`optional` OOB access into a guaranteed crash — cheap enough to ship, not just to test with. Pair with **`_MSVC_STL_DESTRUCTOR_TOMBSTONES=1`** to make UAF deterministic too.
2. **Assert the boundary's own invariant, not the symptom:** `_ASSERTE`/`assert` on every documented size/index/capacity precondition, compiled into hardened and test builds. Per test-practice's Boundary enforcement section, a red test must prove the assertion fires before it's trusted — an assert added without that proof is unverified.
3. **Sanitizer-backed builds for CI/test:** MSVC ASan (`/fsanitize=address`) catches what the checked types above miss (raw pointer arithmetic, buffer copies via legacy APIs) — full detail in Hardening oracles below.
4. **Shipping-build floor underneath all three:** `/sdl /GS /guard:cf` (Hardening oracles) — necessary, not sufficient; enforcement belongs at the boundary, not only the binary.

Validate every length/count/size field crossing a trust boundary (parser input, network data, file formats) against the buffer's actual capacity before use — the BVA class map (test-practice TDD) names exactly which fields need this.

## Hardening oracles

Defect-class refinement for C++: all four classes fully apply (manual memory, UB, data races) — nothing is out of scope by construction.

| Class | Oracle on Windows/MSVC |
| --- | --- |
| Boundary | MSVC ASan (`/fsanitize=address` — mature x64/x86, ARM64 in preview since VS 2026; all CRT flavors, but adds an ASan DLL dependency even for `/MT` since 17.7), or assertion on the corrupted value |
| Lifetime/resource | **ASan catches UAF/double-free but NOT leaks — there is no LSan on Windows (still true in VS 2026).** Leaks: CRT debug heap (`_CrtSetDbgFlag` + `_CrtMemCheckpoint`/`Difference`; not usable simultaneously with ASan) in debug; UMDH snapshot diffs for release/long-running; VS heap profiler interactively; ASan+LSan under WSL for portable code |
| Garbage data | UBSan **via clang-cl only** (cl.exe has none — still an open feature request) — use `-fsanitize=undefined -fsanitize-trap=undefined` or the minimal runtime to dodge chronic link breakage; MSan is Linux-only (WSL leg); `/RTCsu` and `-ftrivial-auto-var-init=pattern` (clang-cl) as hunting aids |
| Concurrency | **No TSan on Windows — cl or clang-cl, permanently.** The accepted leg is TSan under WSL2/Linux CI (caveat: exercises pthread paths, not SRWLOCK/IOCP). Intel Inspector is discontinued (support access ended Jan 2026); Application Verifier catches lock misuse, not races. Windows-only code with no WSL leg → concurrency survivors end `UNPROVEN`, never fixed on suspicion |

MSVC ASan compatibility notes: incompatible with `/RTC*`, incremental linking, `/ZI` (use `/Zi`), PGO, coroutines; **fine with `/O2 /GL`**. `ASAN_OPTIONS=continue_on_error=1` for sweep runs; `stack-use-after-return` needs `/fsanitize-address-use-after-return` plus the env var.

Cheap extra oracles for test builds: **`_MSVC_STL_HARDENING=1`** (17.14+, release-safe OOB checks on vector/span/string_view/optional — fastfail; Microsoft plans default-on post-17.x) and **`_MSVC_STL_DESTRUCTOR_TOMBSTONES=1`** (turns UAF into deterministic crashes); ASan container annotations are default-on (all TUs must agree or `LNK2038`); `_ITERATOR_DEBUG_LEVEL=2` in debug; gflags/PageHeap + Application Verifier for heap corruption. Candidate *generators* (hits still need a failing test to become defects): clang-tidy `bugprone-*`, `cert-*`, `concurrency-*`, `clang-analyzer-*` (via Ninja + compile_commands.json) **and** MSVC `/analyze` with `/analyze:external-` — they barely overlap, run both. Hardening flag floor for shipping builds: `/sdl /GS /guard:cf /DYNAMICBASE /HIGHENTROPYVA /CETCOMPAT` (`/guard:xfg` as the stronger CFI on trust-boundary/hardened builds; `/Qspectre` only on trust-boundary code — benchmark it).
