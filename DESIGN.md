# DESIGN.md — the tutorial suite

This document describes how the four courses are put together and why. It is
written for the next developer (or agent) who has to extend or teach them.

Raven itself is described in its own repository. This repository holds only the
course material; Raven is a development dependency.

---

## 1. Goals and non-goals

**Goals**

* teach Raven end to end, in four languages, in steps that always run;
* one application built four times, so the languages can be compared;
* every step checkable without a display or a GPU;
* a reader can start at step 01 with no GUI toolkit experience;
* packaging explained once per language.

**Non-goals**

* a general-purpose framework — it is a course;
* replacing Raven's library documentation;
* covering every Raven API — the courses use what the editor needs;
* a package repository or CI matrix;
* a single cross-language build system.

---

## 2. Shape

Four courses, each with **eight learning steps and one packaging step**, graded
by level:

| Level | Steps | Theme |
|---|---|---|
| 1 — Beginner | 01, 02 | window, widgets, callbacks, layout |
| 2 — Elementary | 03, 04 | controls and state, actions and shortcuts |
| 3 — Intermediate | 05, 06 | XML interfaces, menus |
| 4 — Advanced | 07, 08, 09 | text editor and files, capstone editor, packaging |

All four build the **same application**: a notes editor with menus, shortcuts,
file dialogs and an unsaved-changes guard.

| Course | Steps | Checks | Binding |
|---|---|---|---|
| C++ | 8 + packaging | 67 | the C API directly |
| Rust | 8 + packaging | 56 | `raven-gui` crate (git source) |
| Python | 8 + packaging | 55 | `bindings/python/raven.py` (ctypes) |
| TypeScript | 8 + packaging | 53 | `raven.ts`, written for this course (Deno FFI) |

---

## 3. Anatomy of a step

* one executable (`main.cpp` / `main.rs` / `main.py` / `main.ts`) and a README;
* the program runs windowed, or with `--test` on the null backend;
* `--test` drives the real widgets with synthetic input and asserts state;
* the README has: goal, what you learn, walkthrough, build, run, test,
  checkpoint, exercises, troubleshooting.

The shared harness differs per language because each has its own idiom:

| Language | Harness | Test entry |
|---|---|---|
| C++ | `common/tutorial.h` (inline helpers) | CTest calls each binary with `--test` |
| Rust | `src/harness.rs` copied per package | `tests/check.rs` spawns the binary; `cargo test` |
| Python | `tutorial.py`, imported by path insert | `check_all.py` runs each script |
| TypeScript | `tutorial.ts` | `check_all.ts`, `deno task test` |

---

## 4. How Raven is reached

Raven is a development dependency, never a copy:

| Course | Raven from | Binding from |
|---|---|---|
| C++ | installed CMake package (`raven::raven`) | the C headers in that package |
| Rust | installed `libraven.a`/`libraven.so`, or a checkout via `RAVEN_LIB_DIR` | `raven-gui` in the Raven repository, pinned by `rev` |
| Python | `RAVEN_LIBRARY` (`libraven.so`) | `PYTHONPATH` to `bindings/python` (or the wheel) |
| TypeScript | `RAVEN_LIBRARY` | `raven.ts` in this course (hand-written FFI) |

`Rust/.cargo/config.toml` sets the Linux `raven-dev` defaults (`/usr/lib`,
static). A real environment variable wins, so a checkout build needs no file
edit. Cargo cannot substitute an environment variable in a manifest; the Rust
binding is therefore a git dependency with a pinned revision, and switching to
a local checkout means changing one line in `Rust/Cargo.toml`.

---

## 5. Decisions

* **XML-first for the bindings.** Rust, Python and TypeScript bindings are
  action-centric (closures/functions per action name); building widgets in code
  would mean passing raw C callbacks everywhere. Rust, Python and TypeScript
  therefore describe the interface in XML from step 01 and bind actions; the
  C++ course starts with the widget tree in code (it is the API being taught)
  and moves to XML at step 05.
* **`--test` on the null backend.** A GUI is untestable until it can run
  headless; Raven's null backend plus `Rwindow_inject_*` makes every step a
  normal unit-testable program. A mock layer would test nothing real.
* **Packaging is a separate step, not a build feature.** Each `09-packaging`
  project builds the step-08 sources and installs/packs them (CPack: DEB + TGZ),
  which teaches that packaging is a build concern and keeps the application
  sources untouched.
* **The binding is not vendored.** The Rust course fetches `raven-gui` from the
  Raven repository; Python uses the module from the binding/wheel; TypeScript
  keeps its own small FFI file because that is the lesson. The C library always
  comes from the installation, never from this repository.
* **The same application four times.** The menu, shortcuts, file format, dirty
  flag, status texts and checks are the same, so a reader can compare languages
  line by line and an agent must change all four together.

---

## 6. Testing

* **Headless only.** `--test` starts Raven on the null backend, injects clicks,
  keys and text, and asserts values (labels, state, file contents), never
  pixels.
* **Every course has one entry point:** CTest, `cargo test --workspace`,
  `check_all.py`, `deno task test`.
* **The packaging steps are built too.** They are where a path or an install
  rule breaks; a course is not done until `09-packaging` configures, builds and
  packs.
* **Writes go to `/tmp`.** Save tests use temporary files and restore nothing.

---

## 7. File map

| Path | Contents |
|---|---|
| `C++/CMakeLists.txt` | the C++ course: eight executables and eight tests |
| `C++/common/tutorial.h` | headless helpers shared by the steps |
| `C++/0N-*/main.cpp` | the step |
| `C++/09-packaging/CMakeLists.txt` | install + CPack |
| `Rust/Cargo.toml` | workspace and the `raven-gui` git dependency |
| `Rust/.cargo/config.toml` | installed-library defaults and the override notes |
| `Rust/0N-*/src/main.rs`, `tests/check.rs` | the step and its check |
| `Rust/09-packaging/CMakeLists.txt` | cargo + install + CPack |
| `Python/tutorial.py` | headless helpers |
| `Python/check_all.py` | runs every step's `--test` |
| `Python/0N-*/main.py` | the step |
| `Python/09-packaging/CMakeLists.txt` | install + launcher + CPack |
| `TypeScript/raven.ts` | the Deno FFI binding |
| `TypeScript/tutorial.ts`, `check_all.ts` | helpers and the runner |
| `TypeScript/0N-*/main.ts` | the step |
| `TypeScript/09-packaging/CMakeLists.txt` | install, ship the library, CPack |

---

## 8. Limits and future work

* The courses are tested on Linux; Windows and macOS instructions exist but are
  not in a CI matrix.
* The Rust binding is pinned to a revision; following Raven needs a `rev` bump.
* The TypeScript binding covers what the editor needs, not the whole C API.
* No translations, no screenshots pipeline, no generated API reference.
