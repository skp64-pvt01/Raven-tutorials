# AGENTS.md — Raven Tutorials

Four graded courses — C++, Rust, Python and TypeScript — that build the same
small text editor on [Raven](https://github.com/skp64-pvt01/Raven). Each course
has eight learning steps and a packaging step.

Raven is a **development dependency**, not vendored: the courses use an installed
Raven, its binding from the Raven repository, and its shared library. This
repository contains only the course material.

---

## What is here

| Path | What it is |
|---|---|
| `C++/` | one CMake project; one executable per step; `common/tutorial.h`; `09-packaging` |
| `Rust/` | one cargo workspace; one package per step; `src/harness.rs`; `09-packaging` |
| `Python/` | one script per step; `tutorial.py`, `check_all.py`; `09-packaging` |
| `TypeScript/` | one script per step; `raven.ts`, `tutorial.ts`, `check_all.ts`; `09-packaging` |
| `README.md` | the overview and the commands for all four courses |
| `DESIGN.md` | how the suite is put together and why |

Every step has a `README.md` with goal, walkthrough, build, run, test,
checkpoint, exercises and troubleshooting.

## Prerequisites

* Raven installed: `raven-dev` (CMake package + library), or a checkout built
  and installed with `cmake --install`.
* CMake 3.20+ and a C++17 compiler; Rust 1.70+; Python 3.8+; Deno.
* A Vulkan driver only for a real window; `--test` uses the null backend.

`Rust/.cargo/config.toml` defaults to the Linux `raven-dev` package
(`RAVEN_LIB_DIR=/usr/lib`, `RAVEN_STATIC=1`). Override the environment for a
checkout build; a real environment variable wins over the file.

## Build and test

```bash
# C++ - 8 executables, 8 CTest entries
cmake -S C++ -B /tmp/raven-cpp
cmake --build /tmp/raven-cpp -j"$(nproc)"
ctest --test-dir /tmp/raven-cpp --output-on-failure

# Rust - 8 packages, each tests/check.rs spawns the binary
cd Rust && cargo test --workspace

# Python - check_all.py runs every step with --test
cd Python
PYTHONPATH=/path/to/Raven/bindings/python \
RAVEN_LIBRARY=/path/to/Raven/build/libraven.so \
python3 check_all.py

# TypeScript - type-check, then every step with --test
cd TypeScript
RAVEN_LIBRARY=/path/to/Raven/build/libraven.so deno task check
RAVEN_LIBRARY=/path/to/Raven/build/libraven.so deno task test
```

The packaging steps are separate CMake projects:

```bash
cmake -S <course>/09-packaging -B /tmp/notes -DCMAKE_INSTALL_PREFIX=/usr
cmake --build /tmp/notes          # C++ and Rust
(cd /tmp/notes && cpack -G DEB)
```

## Rules

1. **The same application in every course.** A behaviour change — a menu item,
   a shortcut, a check, a new step — belongs in all four languages.
2. **Every step is a normal program.** It runs windowed, and with `--test` on the
   null backend; no test framework, no mock layer.
3. **A step README is part of the step.** Goal, walkthrough, build, run, test,
   checkpoint, exercises, troubleshooting; update it with the code.
4. **Packaging stays a separate step.** `09-packaging` builds or installs the
   step-08 sources; do not add packaging to the application step.
5. **No copy of Raven.** Do not vendor `src/`, `include/`, `bindings/` or the
   library. Use the installed package, the binding from Raven, and environment
   variables.
6. **Tests write only to `/tmp`.** File-dialog and save checks use temporary
   paths; never touch the course files.
7. **Do not commit build outputs.** `build/`, `target/`, `__pycache__/`, `.deb`
   and `.tar.gz` are ignored.

## Pitfalls (each one cost a debugging session)

| Symptom | Cause |
|---|---|
| Rust: `unresolved import raven` | The dependency was renamed. Keep `raven-gui.workspace = true` with the workspace entry as `raven-gui = { git = ... }`; Cargo then exposes the library target name `raven`. |
| Rust: hundreds of `rust-lld: undefined symbol` | The binding picked the static `libraven.a` but X11 and friends were not linked. Use `RAVEN_STATIC=1` (the default config does) or build a shared library. |
| Rust: `cannot load libraven.so.0` | Cargo overwrites `LD_LIBRARY_PATH` for the programs it starts. Add an rpath when using a shared checkout build, or use the installed package. |
| Rust: cannot fetch the binding | The git dependency needs the pinned Raven revision (or network). Change `rev` to `branch = "devel"`, or use `path = "/path/to/Raven/bindings/rust"`. |
| Python: `No module named 'raven'` | `PYTHONPATH` must include `Raven/bindings/python` (or install `raven-gui`). |
| Python: `the Raven library was not found` | `RAVEN_LIBRARY` must point at the shared `libraven.so`; the `/usr/lib` static package is not loadable with ctypes. |
| Deno: `Requires ffi access` | Run with `--allow-ffi`; the `deno.json` tasks already pass it. |
| Deno: library not found | `RAVEN_LIBRARY` must point at `libraven.so` (the binding checks it, then the program folder, then the system). |
| C++: `find_package(raven)` fails | Install the `raven-dev` package or `cmake --install` a checkout; then set `CMAKE_PREFIX_PATH` if it is not in `/usr` or `/usr/local`. |
| A step passes alone but not in the course | Relative paths to `main.cpp` / `notes.xml`; each harness finds its own folder, so run through the course build (`ctest`, workspace, `check_all`). |

## Adding a step

1. Create the folder for every course (same number and name).
2. Add the source, the XML if the step has one, and the README.
3. Wire it in: `C++/CMakeLists.txt` (`raven_tutorial(number-name)`), the Rust
   workspace members, `Python/check_all.py` and the TypeScript step list /
   `deno.json` if it is not picked up by the glob.
4. Add checks to the step's `--test` path that assert state, never pixels.
5. Run all four test commands above.

## Definition of done

* Clean builds, **zero warnings**.
* Every step's `--test` passes in all four courses.
* Packaging steps still configure, build and pack.
* Step README / course README updated together.
* Nothing built or temporary is staged.
