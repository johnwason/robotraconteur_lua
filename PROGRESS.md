# Progress

Read `CLAUDE.md` first. Keep this file current: it is how the next session, possibly on another
machine, knows where things stand. Confirm what it says against `git log` and `scripts/verify.sh`.

## Status

**Last verified state:** nothing built yet. The repository contains only the plan.

| Task | Status | Notes |
|---|---|---|
| M0.1 Repo skeleton, license | not started | |
| M0.2 `bootstrap.sh` | not started | |
| M0.3 `verify.sh` | not started | |
| M0.4 Trivial module, error guard | not started | |
| M0.5 Sanitizer builds and teardown stress | not started | |
| M1.1 Matrix userdata | not started | |
| M1.2 Linear algebra | not started | |
| M1.3 Spatial | not started | |
| M1.4 In-place forms | not started | |
| M2.1 Dispatch queue, `rr.step` | not started | |
| M2.2 `host_sim` harness | not started | |
| M2.3 `rr.array`, marshalling | not started | |
| M2.4 Connect, properties, functions, objrefs | not started | |
| M3 Client streaming and async | not started | |
| M4 Server core | not started | |
| M5 Remaining parity | not started | |
| M6 CoppeliaSim slice | human gate | Install and licensing needed |
| M7 Tesseract integration | human gate | Confirm demand first |

Status values: `not started`, `in progress`, `done` (acceptance criteria verified), `blocked`,
`human gate`.

## Next

M0.1, then continue in order.

## In progress

None.

## Blocked / needs a human

None yet.

## Decisions and plan changes

Record the date, the evidence, and what changed.

- 2026-11-08: D8 changed from one node per process to multiple independent nodes. The project owner
  confirmed the single node is only a MATLAB convenience. The core supports it directly (constructor
  plus `Init()`), and RR's own C++ tests run several nodes per process.
- 2026-11-08: D1 and D5 changed. SWIG is now used for fixed-signature wrappers only, with new `.i`
  files; `LuaBridge3` and `sol2` are rejected. Evidence is under "Measurements". Added the podman rule,
  `Containerfile`, and M0.5 sanitizer builds.
- 2026-11-08: RR data now passes through SWIG as raw stack references (`rl::Arg`, `rl::Ret`), following
  the Python binding's raw `PyObject*` approach at the project owner's direction. `%native` is limited
  to true varargs functions.

## Measurements

From the original feasibility study, on x86_64 Ubuntu with Eigen 3.4 and stripped `-O2` builds:

- Lua 5.4 shared library: 265 KB.
- Eigen kernels with no Lua wrappers: 39 KB for basic and spatial operations, 264 KB for
  decompositions, 285 KB combined.
- Python: `libpython3.12` 9 MB, untrimmed standard library 55 MB, NumPy 36 MB.

Binding comparison, 2026-11-08: the same small C++ class wrapped with SWIG 4.2 and `LuaBridge3` master,
on x86_64 in `ubuntu:24.04`, with Lua 5.4.7 built as C and LuaJIT 2.1. This covered one class and one
error path, not the real RR surface.

| | SWIG 4.2 | `LuaBridge3` |
|---|---|---|
| RSS after 50,000 C++ throws with a 4 KB argument, Lua 5.4 | 4.5 → 4.5 MB | 4.6 → 422 MB |
| Same, LuaJIT | 4.6 → 4.8 MB | 4.4 → 4.6 MB |
| Stripped module size, LuaJIT build | 53 KB | 81 KB |
| Stripped module size, Lua 5.4 build | not captured | 77 KB |
| Time per method call | 73–78 ns | 62–74 ns |

The `LuaBridge3` leak was the same with and without `enableExceptions`. It is about 8.3 KB per throw,
consistent with the 4 KB argument and 4 KB exception message not being destroyed (inferred, not traced).

Raw pass-through test, 2026-11-08, same environment, using the `rl::Arg` and `rl::Ret` typemaps: tables,
functions and `nil` passed through intact; multiple returns worked; wrong argument counts raised Lua
errors. RSS stayed flat over 50,000 throws (Lua 5.4: 4.3 → 4.6 MB; LuaJIT: 4.5 → 4.9 MB). 66–69 ns per
call. 54 KB stripped.

Maintenance as of 2026-11-08: `sol2` last commit March 2025; `LuaBridge3` last commit October 2026.

## Ideas

Out of scope for now. Do not act on these without a plan change.

- Transparent `__index` proxy written in pure Lua on top of the explicit API.
- LuaJIT FFI paths for very fast array access.

## Session log

Newest first. Each entry: date, machine-independent summary, what was verified and how.

- 2026-11-08: Plan created from a feasibility discussion. No code written, nothing run.
