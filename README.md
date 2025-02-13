# Formal Verification of Asterinas OSTD with Verus

The `vostd` project provides a formally-verified version of [OSTD](https://asterinas.github.io/book/ostd/index.html), the (unofficial) standard library for OS development in safe Rust. OSTD encapsulates low-level hardware interactions—which requires using `unsafe` Rust—into a small yet powerful set of high-level, safe abstractions. These abstractions enable the creation of complex, general-purpose OSes lik [Asterinas](https://github.com/asterinas/asterinas) entirely in safe Rust.

By design, OSTD guarantees _soundness_: no undefined behavior is possible, regardless of how its API is used in safe Rust. The goal of the `vostd` project is to bolster confidence in this soundness through formal verification, leveraging the [Verus](https://github.com/verus-lang/verus) verification tool.

This work is ongoing. Our current focus is on verifying OSTD’s _memory management subsystem_, a core component that is directly related to kernel memory safety. As we continue, we aim to extend formal verification to additional parts of OSTD to further ensure its reliability and correctness.

## Building the Proof Development

#### Install Rust

If you have not installed Rust yet, follow the [official instructions](https://www.rust-lang.org/tools/install).

#### Install LLVM

##### On Windows

Download the [LLVM binary installer](https://github.com/llvm/llvm-project/releases). Then create an environment variable named `LLVM_OBJDUMP` and set it to point to `llvm-objdump.exe`.

##### On Debian/Ubuntu

Run

```
sudo apt update
sudo apt install build-essential unzip pkg-config libssl-dev llvm
```

##### On Mac OS

Obtain the `binutils` package if you don't already have it. It comes bundled with `xcode`, but you can also obtain it through an external source such as homebrew:
```
brew install binutils
```

Make sure that the `llvm-objdump` binary is in your path, e.g. if installed via `xcode`:
```
export PATH=$PATH:/Library/Developer/CommandLineTools/usr/bin/
```

Or via homebrew:
```
export PATH=$PATH:/usr/local/opt/binutils/bin/
```

#### Build Verus

The ``tools/verus`` directory contains the Verus repository at tag  ``release/rolling/0.2024.10.25.601e1e7``, which is guaranteed to be compatible with the rest of the development.

For VsCode users, you can build verus with the verus-analyzer:
```bash
cargo xtask bootstrap
```
otherwise, you can build it with:
```bash
cargo xtask bootstrap --no-vscode-extension
```

#### Build Verification Targets 

Then, run ``make`` to build the common libraries and both verification targets.
Make simply runs:

```bash
cargo xtask compile --targets vstd_extra
cargo xtask compile --targets aster_common
cargo xtask verify --targets fvt10-pt-cursor-navigation
cargo xtask verify --targets fvt11-pt-cursor-guards
```

After the first build, you may directly build one of the specific targets
with ``make fvt10`` or ``make fvt11``

## The verification targets

Currently, this repository contains two verification targets. 

Target ``fvt10-pt-cursor-navigation``
verifies the behavior of the cursor methods ``push_level``, ``pop_level``, and ``move_forward``.
The specification for these functions are defined in ``src/page_table/cursor/model.rs``, along with
the ``ConcreteCursor`` type that contains an abstract instance of a page table, represented as a tree,
as well as the path into that tree that the cursor currently points to, and the subtree found at the
end of the path. The functions themselves, and their verification, are found in ``src/page_table/cursor/mod.rs``.

Target ``fvt11-pt-guards`` extends the previous target's specification with a system of locks. Take a look at
``src/page_table/cursor/mod.rs`` and note that the proofs are much more complex, requiring multiple assertions and lemmas.

We will release the code for more verification targets.