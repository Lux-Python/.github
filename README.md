<p align="center">
  <img src="assets/logo.png" alt="Lux Logo" width="220" />
</p>

<h1 align="center">Lux Systems Engineering</h1>

<p align="center">
  <strong>Next-generation developer infrastructure, SAT resolvers, and hermetic runtimes for Python.</strong>
</p>

<p align="center">
  <a href="https://github.com/Lux-Python/Lux"><img src="https://img.shields.io/badge/flagship-lux-blueviolet.svg?style=flat-square" alt="Flagship" /></a>
  <a href="https://www.rust-lang.org"><img src="https://img.shields.io/badge/built_with-100%25_Safe_Rust-orange.svg?style=flat-square&logo=rust" alt="Rust" /></a>
  <a href="https://github.com/Lux-Python/Lux/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT_OR_Apache--2.0-blue.svg?style=flat-square" alt="License" /></a>
</p>

<p align="center">
  <a href="https://github.com/Lux-Python/Lux">Repository</a> •
  <a href="https://github.com/Lux-Python/Lux#quickstart">Quickstart</a> •
  <a href="https://github.com/Lux-Python/Lux#features">Feature Guides</a> •
  <a href="https://github.com/Lux-Python/Lux#performance--architecture">Architecture</a> •
  <a href="https://github.com/Lux-Python/Lux/releases">Releases</a>
</p>

---

## About Us

**Lux Systems Engineering** designs and builds high-performance, deterministic developer toolchains. We believe that developer tools should be instantaneous, zero-overhead, memory-safe, and visually clean.

Our flagship project is [**Lux (`lux`)**](https://github.com/Lux-Python/Lux): an all-in-one Python package and project manager, SAT dependency solver, hermetic runtime orchestrator, and binary ABI bridge written in 100% safe Rust (`#![deny(unsafe_code)]`).

---

## Why Lux?

<table>
  <tr>
    <td width="50%">
      <h3>Fast SAT Resolution</h3>
      <p>A specialized PubGrub SAT solver resolves package dependencies quickly and deterministically directly in memory.</p>
    </td>
    <td width="50%">
      <h3>Binary ABI Inspection (<code>lux doctor</code>)</h3>
      <p>In-process binary inspection parses PE (<code>.pyd</code>), ELF (<code>.so</code>), and Mach-O extensions to catch missing shared libraries and ABI mismatches before execution.</p>
    </td>
  </tr>
  <tr>
    <td width="50%">
      <h3>Content-Addressable Storage (CAS)</h3>
      <p>Global artifact deduplication (<code>~/.lux/cache/cas</code>) hardlinks files into virtual environments with cryptographic SHA-256 validation.</p>
    </td>
    <td width="50%">
      <h3>Python Runtime Management</h3>
      <p>Manage, pin (<code>.python-version</code>), and provision Python runtimes without requiring root permissions.</p>
    </td>
  </tr>
  <tr>
    <td width="50%">
      <h3>Drop-in Pip Interface</h3>
      <p>Seamless replacement for <code>pip</code> with full support for <code>lux pip install -r requirements.txt</code> and <code>-e &lt;path&gt;</code> editable installs.</p>
    </td>
    <td width="50%">
      <h3>Single-File Scripts & Workspaces</h3>
      <p>Execute standalone scripts with PEP 723 inline dependencies (<code>lux run script.py</code>) and coordinate monorepos (<code>[tool.lux.workspace]</code>).</p>
    </td>
  </tr>
</table>

---

## Quickstart

Install Lux with a single command:

#### Windows (PowerShell)
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://raw.githubusercontent.com/Lux-Python/Lux/main/install.ps1 | iex"
```

#### macOS and Linux
```bash
curl -LsSf https://raw.githubusercontent.com/Lux-Python/Lux/main/install.sh | sh
```

Initialize and run a project in seconds:

```bash
# Initialize project with pyproject.toml and .venv
lux init my-app
cd my-app

# Or create a standalone virtual environment anywhere
lux venv

# Add dependencies with instant PubGrub SAT resolution
lux add fastapi uvicorn "pydantic>=2.0"

# Run your application in the virtual environment
lux run uvicorn main:app --reload
```

---

## Technical Deep Dives

<details>
<summary><strong>PubGrub-CDCL SAT Solver Architecture</strong></summary>

<br/>

Lux uses a Conflict-Driven Clause Learning (CDCL) dependency solver designed for package version intervals:
- **Unit Propagation:** Propagates positive and negative package assignments to minimize decisions.
- **Incompatibility Graph:** Derives conflict clauses upon encountering unsatisfiable requirement constraints.
- **Non-Chronological Backjumping:** Backjumps directly to the decision level responsible for the conflict.
- **Pure Rust In-Process Execution:** Executes directly in-memory without spawning external resolver processes.

</details>

<details>
<summary><strong>Native Binary ABI Inspection (<code>lux doctor</code>)</strong></summary>

<br/>

Compiled C/C++ extensions frequently fail at runtime due to missing dynamic link libraries (e.g. `MSVCP140.dll`, `libgomp.so`, `libc++.dylib`).

`lux doctor` provides native, zero-copy binary inspection powered by the `object` crate:
- Inspects PE/COFF, ELF, and Mach-O import headers directly from disk.
- Validates that all imported symbols and shared libraries resolve against the active virtual sysroot (`.venv/sysroot`) and system library paths.
- Provides immediate diagnostics with actionable resolution paths before running Python.

</details>

<details>
<summary><strong>Content-Addressable Storage (CAS) & Zero-Copy Linking</strong></summary>

<br/>

Lux uses a global CAS directory (`~/.lux/cache/cas`):
- Wheel archives are downloaded and decompressed directly into content-addressed directories indexed by SHA-256.
- Virtual environment installation uses atomic filesystem hardlinks with copy fallback.
- Disk usage across multiple projects sharing packages is deduplicated.

</details>

---

## Core Repositories & Workspace Crates

Our flagship codebase is hosted at [**`Lux-Python/Lux`**](https://github.com/Lux-Python/Lux):

```
lux
├── crates/lux_core       # PEP 440, PEP 508, PEP 621, PEP 723, lux.lock, .python-version
├── crates/lux_resolver   # PubGrub-CDCL SAT dependency solver
├── crates/lux_cache      # CAS store, HTTP/2 range reader, Python runtime manager
├── crates/lux_sysroot    # Binary ABI inspector (PE/ELF/Mach-O) & PEP 517 builder
└── crates/lux_cli        # Minimalist CLI toolchain & pip interface
```

---

## Engineering Invariants

1. **100% Safe Rust:** `#![deny(unsafe_code)]` enforced across all workspace crates.
2. **Zero Clippy Warnings:** Maintained under `-D clippy::all -D clippy::pedantic -D clippy::nursery`.
3. **No Marketing Fluff or Emojis:** Strict Cargo/uv design language with right-aligned verbs, microsecond timestamps, and pure box-drawing characters.
4. **Hermetic Determinism:** Cryptographic integrity hashes (SHA-256) on all locked packages.

---

<p align="center">
  <sub>© 2026 Lux Systems Engineering • Open Source under MIT OR Apache-2.0</sub>
</p>
