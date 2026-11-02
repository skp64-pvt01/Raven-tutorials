# Raven Tutorials

Four graded courses that take a reader from an empty `main()` to a small text
editor — the shape most desktop tools start from — using
[Raven](https://github.com/skp64-pvt01/Raven):

| Course | Steps | Language | Binding |
|---|---|---|---|
| [C++](C++/) | 8 + packaging | C++17 | the C API directly |
| [Rust](Rust/) | 8 + packaging | Rust | the `raven-gui` crate (git source) |
| [Python](Python/) | 8 + packaging | Python 3.8+ | `bindings/python/raven.py` (ctypes) |
| [TypeScript](TypeScript/) | 8 + packaging | Deno | `raven.ts` (Deno FFI) |

All four build the **same application** — a notes editor with menus, shortcuts,
file dialogs, a dirty flag and drag and drop — so a reader can compare the
languages side by side.

Raven is a **development dependency**, not vendored: the courses use an
installed Raven (the `raven-dev` package, or `cmake --install` of a checkout),
the binding from the Raven repository, and the shared library from the same
installation. There is no copy of Raven in this repository.

---

## Before you start

* **Raven installed** — `raven-dev` and the CMake package, or a checkout built
  and installed:

  ```bash
  cmake -S /path/to/Raven -B /path/to/Raven/build -DRAVEN_BUILD_SHARED=ON
  cmake --build /path/to/Raven/build -j"$(nproc)"
  cmake --install /path/to/Raven/build --prefix /usr/local
  ```

* **CMake 3.20+** and a C++17 compiler for the C++ course.
* **Rust 1.70+** for the Rust course.
* **Python 3.8+** for the Python course.
* **Deno** for the TypeScript course.
* A Vulkan driver only for a real window. Every step has `--test`, which uses
  Raven's null backend and needs neither a display nor a GPU.

---

## Build and test

From the root of this repository.

### C++

```bash
cmake -S C++ -B /tmp/raven-cpp
cmake --build /tmp/raven-cpp -j"$(nproc)"
ctest --test-dir /tmp/raven-cpp --output-on-failure
```

If Raven is not in `/usr` or `/usr/local`, add
`-DCMAKE_PREFIX_PATH=/your/prefix`.

### Rust

```bash
cd Rust
cargo test --workspace
```

`Rust/.cargo/config.toml` defaults to the installed `raven-dev` package on
Linux (`/usr/lib`, static). For a checkout build:

```bash
RAVEN_LIB_DIR=/path/to/Raven/build RAVEN_STATIC=0 \
RUSTFLAGS="-C link-arg=-Wl,-rpath,/path/to/Raven/build" \
cargo test --workspace
```

The `raven-gui` binding is fetched from the Raven repository, pinned to the
revision the course was written against; see [Rust/README.md](Rust/README.md)
to follow `devel` or use a local checkout.

### Python

```bash
cd Python
PYTHONPATH=/path/to/Raven/bindings/python \
RAVEN_LIBRARY=/path/to/Raven/build/libraven.so \
python3 check_all.py
```

On Windows the `raven-gui` wheel installs the module and library together, so
neither variable is needed.

### TypeScript

```bash
cd TypeScript
RAVEN_LIBRARY=/path/to/Raven/build/libraven.so deno task check
RAVEN_LIBRARY=/path/to/Raven/build/libraven.so deno task test
```

Each step also runs on its own: `deno task 05` (or `deno run --allow-ffi ...`),
or the equivalent `cargo run -p ...` / `cmake` / `python3`.

---

## Packaging

Every course ends with a `09-packaging` CMake project that builds or installs
the step-08 application and produces a DEB/TGZ with CPack. They are the only
steps that install files; the application sources stay untouched.

```bash
# C++
cmake -S C++/09-packaging -B /tmp/notes-cpp -DCMAKE_INSTALL_PREFIX=/usr
cmake --build /tmp/notes-cpp && (cd /tmp/notes-cpp && cpack -G DEB)

# Rust
cmake -S Rust/09-packaging -B /tmp/notes-rust -DCMAKE_INSTALL_PREFIX=/usr
cmake --build /tmp/notes-rust && (cd /tmp/notes-rust && cpack -G DEB)

# Python
cmake -S Python/09-packaging -B /tmp/notes-py -DCMAKE_INSTALL_PREFIX=/usr
(cd /tmp/notes-py && cpack -G DEB)

# TypeScript (the package also ships libraven.so)
RAVEN_LIBRARY=/path/to/Raven/build/libraven.so \
cmake -S TypeScript/09-packaging -B /tmp/notes-ts -DCMAKE_INSTALL_PREFIX=/usr
(cd /tmp/notes-ts && cpack -G DEB)
```

Each step's README explains the CPack variables and how to inspect the
package.

---

## Layout

```
Raven-tutorials/
├── C++/            one CMake project, one executable per step
├── Rust/           one cargo workspace, one package per step
├── Python/         one script per step, tutorial.py, check_all.py
├── TypeScript/     one script per step, raven.ts, tutorial.ts, check_all.ts
├── README.md       this file
├── AGENTS.md       build/test hand-off and the traps
├── DESIGN.md       how the suite is put together
└── LICENSE         Apache-2.0
```

---

## License

Apache-2.0; see [`LICENSE`](LICENSE) and [`NOTICE`](NOTICE). Raven is Apache-2.0
as well.
