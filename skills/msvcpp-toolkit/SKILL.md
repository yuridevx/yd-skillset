---
name: msvcpp-toolkit
description: C++ language toolkit, MSVC/Windows-first — the language-specific half of the algo-rewrite and harden skills. Supplies, in named sections, toolchain detection (cl vs clang-cl, /arch, vcpkg triplets), benchmark harness order (Google Benchmark first) with Windows measurement hygiene and alloc-guard instrumentation, hot-path hidden-allocator list and error model, SIMD (highway-first, clang-cl for hot TUs) and concurrency (oneTBB/Taskflow, PPL is frozen legacy) facilities, the library-first shortlist with maintenance-health flags, fuzzing on Windows (clang-cl libFuzzer first, Jackalope for binary-only), and the per-defect-class hardening oracle table (MSVC ASan; UBSan via clang-cl; TSan/MSan/LSan need a WSL leg). PASSIVE reference — invoked by algo-rewrite and harden when the target language is C++; also usable directly when choosing C++ performance or hardening tooling on Windows. Trigger on "msvcpp-toolkit", "/msvcpp-toolkit", "msvc toolkit", or as the C++ toolkit for algo-rewrite/harden.
---

# msvcpp-toolkit

Language toolkit for C++, consumed by **algo-rewrite** and **harden**. **MSVC/Windows-first**: recommendations assume Visual Studio 2022/2026 with cl.exe and clang-cl available; pure clang/gcc-on-Linux projects can take the same first picks minus the MSVC caveats. Each `##` section below is part of the toolkit contract those skills reference by name; a new `<lang>-toolkit` (csharp-toolkit, rust-toolkit, …) must provide the same eight sections.

Recommendations are ordered: take the first that installs on the project's toolchain; ties go to what the project already depends on. Maintenance-health notes are as of mid-2026 — re-verify anything flagged before adopting.

## Toolchain

Detect and record in the ledger header: MSVC version (VS 2022 17.x / VS 2026 18.x — several libraries now gate on this), whether **clang-cl** is installed (it is the escape hatch for UBSan, libFuzzer DLL targets, and SIMD-heavy hot TUs — MSVC-ABI-compatible, per-TU mixing is routine, one linker per binary, never mix LTO across compilers), C++ standard, `/arch` level actually enabled (MSVC defines no `__SSE__`/`__AVX__`/`__FMA__` macros; without `/arch:AVX2` expect non-VEX/VEX mixing penalties), build system, CPU (cores, **P/E hybrid?**, caches), package manager.

Package manager: **vcpkg** (manifest mode + `builtin-baseline`; binary caching via NuGet/GitHub Packages — the old `x-gha` provider is removed) → Conan 2 → CPM/FetchContent. Triplet rules: `/MD`-family (`x64-windows`, `x64-windows-static-md`) for normal and **ASan** builds — never `/MT`+ASan via vcpkg; a **separate `/MT` static build only for clang-cl fuzzing** (its libFuzzer requires the static CRT).

`compile_commands.json` on Windows requires the **Ninja** generator — VS generators don't export it (matters for clang-tidy).

## Benchmarking

Harness: **Google Benchmark** (healthy, MSVC in CI; statistical repetitions, `DoNotOptimize`, `compare.py`, `RegisterMemoryManager`). Use `--benchmark_enable_random_interleaving --benchmark_repetitions=9` to cut Windows variance; keep CRT flavor consistent and set `BENCHMARK_STATIC_DEFINE` correctly or expect link errors. nanobench is **dormant** (no release since 2023) — acceptable as a frozen single-header tool, not for anything long-lived. Catch2 `BENCHMARK` only if Catch2 is already the test framework.

**Windows measurement hygiene** (numbers are noise without this): High Performance power plan; pin the bench thread to one **P-core** via `SetThreadAffinityMask` (hard affinity is the only reliable isolation on hybrid Intel; CPUSETs are soft hints); `SetPriorityClass(HIGH_PRIORITY_CLASS)` (not REALTIME); `QueryPerformanceCounter` for timing; beware Win11 EcoQoS down-clocking low-priority processes; add the bench binary to Defender exclusions. Google Benchmark's CPU-scaling warning is weaker on Windows — absence of the warning guarantees nothing.

Instrumentation: generate `bench/algo_bench.h` with `ALGO_BENCH_SCOPE(name)`, `ALGO_BENCH_COUNT(name, n)`, `ALGO_BENCH_ALLOC_GUARD(name)`, gated on `ALGO_BENCH=1`; without the flag they expand to **nothing** — no branch, no atomic, no string — so they may live in product code permanently.

Allocation tracking: hand-rolled global `operator new`/`operator delete` counting hook with thread-local counters (~30 lines) — still the norm, no established library exists; wire it into the alloc guard and, with Google Benchmark, into `RegisterMemoryManager` so allocs/bytes land in the JSON output.

Macro profiling stack, in order of reach-for: VS Profiler (zero setup) → **Tracy** (free, healthy, frame/zone instrumentation; bus-factor-1) or Superluminal (commercial) → Intel VTune (deepest PMU) → WPA/ETW (system-level interference).

## Hot-path rules

Hidden allocators to avoid on hot paths: `std::function` beyond SBO, `std::string` temporaries, `std::vector`/`std::deque` growth, `unordered_map` rehash, `shared_ptr` control blocks, iostreams, `std::regex`, `throw`/`catch`, coroutine frames that fail HALO.

Error model: errors by value — `std::expected`/`std::optional`/status codes; exceptions are setup/teardown territory. Layout is fully under your control: contiguous over node-based, SoA for SIMD, `alignas` padding against false sharing. Debug-vs-release trap: `_ITERATOR_DEBUG_LEVEL` is ABI-breaking (all TUs must agree) and makes debug-build hot-path numbers meaningless — benchmark release only.

## SIMD / data-parallel

**Do not trust cl.exe auto-vectorization for hot loops** — it trails LLVM (no real gather/scatter, FP reductions need `/fp:fast`, conservative aliasing); `/Qvec-report:2` tells you what failed but cryptically. The reliable patterns, in order:

1. **google/highway** — runtime dispatch (SSE4/AVX2/AVX-512/NEON) from one compile, MSVC in CI, battle-tested (Chromium, NumPy). On MSVC build with `/arch:AVX2` + `/Gv`; the project itself treats clang-cl as the happier Windows path.
2. **Compile hot TUs with clang-cl** inside the otherwise-MSVC project — the established pattern (Chrome/Firefox ship this way); simdjson-class code runs ~2× faster than under cl.
3. **xsimd** — nicer fixed-ISA API; needs explicit `/arch:*`; **ARM64/NEON broken under cl.exe** (clang-cl required there).

Avoid: EVE (MSVC-broken, no Windows CI), `std::simd` (C++26 on paper, **not shipped in MSVC STL** — revisit ~2027+).

## Concurrency

**oneTBB** (healthy under UXL Foundation, MSVC CI) for parallel loops/reductions/concurrent containers. **Taskflow** (healthy; bus-factor concentrated) when the problem is a dependency DAG. MSVC's C++17 parallel STL (built on the Windows Thread Pool) is a fine drop-in for `sort(par, …)`-class work — note it deliberately keeps some algorithms serial. Raw Windows Thread Pool API as the zero-dependency baseline. **PPL/ConcRT is frozen legacy** — fine to keep, wrong to adopt. Lock-free queues: moodycamel/concurrentqueue (battle-tested but bus-factor-1, slow maintenance — treat as frozen) or boost.lockfree. stdexec/P2300: broken under cl.exe, not production — mention, don't adopt.

## Library shortlist

| Domain | First pick | Health / MSVC notes |
| --- | --- | --- |
| Hash map | `boost::unordered_flat_map` | Healthy, org-backed, excellent MSVC. Fallback `ankerl::unordered_dense` (healthy, single-maintainer; contiguous, iteration-heavy) |
| Hash function | rapidhash | Healthy; uses `_umul128` on MSVC x64 properly; fastest SMHasher3-passing hash. Fallback XXH3 (slow cadence, canonical) |
| JSON (read-heavy) | simdjson | Healthy, VS 2026 + clang-cl in CI; **MSVC codegen ~40–100% slower than clang-cl — build hot JSON TUs with clang-cl** |
| JSON (serialize/deserialize structs) | Glaze | Hyperactive but **bus-factor-1 and MSVC-bleeding-edge**: floor is VS 2026 (14.50), VS 2022 effectively dropped, `/Zc:preprocessor` mandatory, each toolset rev re-rolls the ICE dice. On VS 2022 pin an old version or skip. Cold convenience paths: nlohmann |
| Formatting | fmt | Healthy (v12.x), excellent MSVC; `std::format` fine on fully-C++23 toolchains |
| Global allocator | mimalloc | Healthy, Microsoft-hosted (but solo-maintained); v3.3+ is the recommended line, v2.3 for max soak. **Prefer explicit `mi_*` or `mimalloc-new-delete.h` override; the Windows redirect DLL is closed-source and fragile against AV/EDR.** jemalloc archived 2025 — do not adopt |
| Compile-time const maps | frozen | Dormant/maintenance-mode — stable, fine as a frozen dependency |

abseil: only if already in-tree (boost/ankerl beat it now). folly: cherry-pick ideas, never adopt as a dependency.

## Fuzzing

Order for MSVC-built code:

1. **clang-cl `-fsanitize=address,fuzzer`** — the capable path: supports `fuzzer-no-link` (so DLL targets work). Hard constraints: **requires ASan, `/MT` static CRT, no `/INCREMENTAL`/`/DEBUG`**.
2. **MSVC `/fsanitize=fuzzer`** (pass `/fsanitize=address` separately — comma syntax rejected) — EXE/static-lib targets only (**no DLL fuzzing**, no `fuzzer-no-link`); accepts all CRT flavors; x64-centric, experimental-but-shipping.
3. Binary-only targets: **Jackalope** (active, Project Zero) → WinAFL (older, needs DynamoRIO). Kernel/hard-to-harness: wtf (snapshot fuzzer, bus-factor-1).

Structure-aware inputs: **FuzzedDataProvider** (single header) — still standard. libFuzzer is maintenance-only upstream; **Google FuzzTest is Linux/Bazel-only — not an option on MSVC**. AFL++ has no native Windows support.

## Hardening oracles

Defect-class refinement for C++: all four classes fully apply (manual memory, UB, data races) — nothing is out of scope by construction.

| Class | Oracle on Windows/MSVC |
| --- | --- |
| Boundary | MSVC ASan (`/fsanitize=address` — mature x64/x86; all CRT flavors, but adds an ASan DLL dependency even for `/MT` since 17.7), or assertion on the corrupted value |
| Lifetime/resource | **ASan catches UAF/double-free but NOT leaks — there is no LSan on Windows.** Leaks: CRT debug heap (`_CrtSetDbgFlag` + `_CrtMemCheckpoint`/`Difference`; not usable simultaneously with ASan) in debug; UMDH snapshot diffs for release/long-running; VS heap profiler interactively; ASan+LSan under WSL for portable code |
| Garbage data | UBSan **via clang-cl only** (cl.exe has none) — use `-fsanitize=undefined -fsanitize-trap=undefined` or the minimal runtime to dodge chronic link breakage; MSan is Linux-only (WSL leg); `/RTCsu` and `-ftrivial-auto-var-init=pattern` (clang-cl) as hunting aids |
| Concurrency | **No TSan on Windows — cl or clang-cl, permanently.** The accepted leg is TSan under WSL2/Linux CI (caveat: exercises pthread paths, not SRWLOCK/IOCP). Intel Inspector is discontinued (EOL 2026); Application Verifier catches lock misuse, not races. Windows-only code with no WSL leg → concurrency survivors end `UNPROVEN`, never fixed on suspicion |

MSVC ASan compatibility notes: incompatible with `/RTC*`, incremental linking, `/ZI` (use `/Zi`), PGO, coroutines; **fine with `/O2 /GL`**. `ASAN_OPTIONS=continue_on_error=1` for sweep runs; `stack-use-after-return` needs `/fsanitize-address-use-after-return` plus the env var.

Cheap extra oracles for test builds: **`_MSVC_STL_HARDENING=1`** (17.14+, release-safe OOB checks on vector/span/string_view/optional — fastfail) and **`_MSVC_STL_DESTRUCTOR_TOMBSTONES=1`** (turns UAF into deterministic crashes); ASan container annotations are default-on (all TUs must agree or `LNK2038`); `_ITERATOR_DEBUG_LEVEL=2` in debug; gflags/PageHeap + Application Verifier for heap corruption. Candidate *generators* (hits still need a failing test to become defects): clang-tidy `bugprone-*`, `cert-*`, `concurrency-*`, `clang-analyzer-*` (via Ninja + compile_commands.json) **and** MSVC `/analyze` with `/analyze:external-` — they barely overlap, run both. Hardening flag floor for shipping builds: `/sdl /GS /guard:cf /DYNAMICBASE /HIGHENTROPYVA` (`/Qspectre` only on trust-boundary code — benchmark it).
