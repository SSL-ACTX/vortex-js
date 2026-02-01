<div align="center">

![VortexJS Banner](https://capsule-render.vercel.app/api?type=waving&color=0:00ffff,100:06b6d4&height=220&section=header&text=VortexJS&fontSize=80&fontColor=000000&animation=fadeIn&fontAlignY=35&desc=Next-Gen%20Stackless%20JavaScript%20Virtualization&descSize=20&descAlignY=55)

<!-- Badges -->
[![License: MIT](https://img.shields.io/badge/License-AGPL_3.0-blue.svg?style=for-the-badge&logo=open-source-initiative)](https://opensource.org/licenses/AGPL-3.0)
[![Node.js](https://img.shields.io/badge/Node.js-v18%2B-green.svg?style=for-the-badge&logo=node.js)](https://nodejs.org/)
[![Architecture](https://img.shields.io/badge/Architecture-SVM-magenta.svg?style=for-the-badge&logo=cpu)]()
[![Optimization](https://img.shields.io/badge/IR_Opt-Level_3-yellow.svg?style=for-the-badge)]()
[![Started](https://img.shields.io/badge/Started-Nov_24_2025-cyan.svg?style=for-the-badge&logo=calendar)]()
[![Status](https://img.shields.io/badge/Status-Beta-red.svg?style=for-the-badge&logo=clock)](https://github.com/SSL-ACTX/vortex-js)

[**Explore Internals**](#-architecture--internals) &middot; [**Report Bug**](https://github.com/SSL-ACTX/vortex-js/issues) &middot; [**Live Compiler**](https://vortexjs-server.onrender.com/)

</div>

---

> [!NOTE]
> **VortexJS is a Research-Inspired Virtualization Engine.**
> It is an educational study in compiler theory and software protection. It serves as a proof-of-concept for a source-to-source compiler that translates JavaScript into a custom bytecode instruction set, executed by a polymorphic, stackless virtual machine.

**VortexJS** is an advanced **JavaScript Virtualization Engine** and **Optimizing Compiler**. It transforms standard ECMAScript code into a linear **Finite State Machine (FSM)** running atop a custom **stackless virtual machine**.

By implementing a virtual instruction pointer, manual stack management, and a custom memory heap, VortexJS decouples code execution from the host environment's native call stack. This creates a secure, sandboxed execution environment that is mathematically complex to reverse engineer.

> [!WARNING]
> **SECURITY NOTICE**
> Unauthorized copies of this project have been found containing malicious
> modifications.
>
> This repository is the ONLY official source maintained by the author.
>
> Any software obtained elsewhere should be considered unsafe and removed.

---

## 📑 Table of Contents

- [Core Capabilities](#-core-capabilities)
- [Architecture & Internals](#-architecture--internals)
  - [The Stackless VM (SVM)](#the-stackless-vm-svm)
  - [Polymorphic Dispatchers](#polymorphic-dispatchers)
  - [Memory & Scope Virtualization](#memory--scope-virtualization)
  - [The Generator & Async Protocol](#the-generator--async-protocol)
- [Security Features](#-security-features)
  - [Hyperwave String Encryption](#hyperwave-string-encryption)
  - [Opaque Predicates](#opaque-predicates)
  - [Anti-Tamper Mechanisms](#anti-tamper-mechanisms)
- [Compiler Pipeline](#-compiler-pipeline)
  - [IR Optimization (Level 3)](#ir-optimization-level-3)
- [Installation](#-installation)
- [Usage & Configuration](#-usage--configuration)
- [Authors & Credits](#%EF%B8%8F-authors--credits)

---

## 🚀 Core Capabilities

*   **🌀 Control Flow Graph (CFG) Flattening:** Deconstructs complex AST nodes (`if`, `for`, `while`, `switch`, `try-catch`) into a flat, linearized instruction stream driven by a state register (`S`).
*   **⚡ Stackless Architecture:**
    *   **Infinite Recursion:** Implements a "Trampoline" execution model. Function calls push virtual frames to a heap-allocated stack (`VS`), preventing `RangeError: Maximum call stack size exceeded`.
    *   **Heap-Based Execution:** All local variables are mapped to a virtual memory array (`M`), decoupling logic from the native JS scope.
*   **🧵 Asynchronous & Generator Support:**
    *   **State Suspension:** The VM can serialize its entire state (`M`, `S`, `VS`) to pause execution for `await` or `yield` operations, resuming seamlessly when the Promise resolves.
    *   **Concurrency:** Supports `Promise.all`, `Promise.race`, and interleaved execution of virtual threads.
*   **📦 ES Module & Class Support:**
    *   **Import/Export Hoisting:** Automatically separates ESM syntax from virtualized logic to maintain bundler compatibility.
    *   **Deep Class Deconstruction:** Transforms ES6 classes into constructor functions, `Reflect.construct` calls, and `WeakMap`-based private field implementations.
*   **🛡️ Hybrid Execution Mode:**
    *   **Targeted Protection:** Use the `"use vortex";` directive to virtualize specific functions while leaving non-critical code native for maximum performance.

---

## 🏗 Architecture & Internals

VortexJS functions as a compiler. It parses source code into an **Intermediate Representation (IR)**, optimizes it, and generates a custom VM instruction set.

### The Stackless VM (SVM)

Unlike recursive interpreters, VortexJS runs inside a single, perpetual `while(true)` loop.

```javascript
// Simplified Runtime Model
const VM = async (EntryState, EntryArgs) => {
  let M = [...GlobalMemory]; // Virtual Heap
  let S = EntryState;        // Instruction Pointer
  let VS = [];               // Virtual Stack (Shadow Stack)

  while (true) {
    try {
      // Polymorphic Dispatcher (Switch, BST, or Chaos) decides next op
      switch (S) { 
        case CALL_OP:
          // Manual Stack Management (Trampoline)
          VS.push({ M, S: return_address });
          M = createNewMemoryFrame(args); // New Scope
          S = target_function_entry;      // Jump
          continue; 
        
        case RETURN_OP:
          // Stack Unwinding
          const frame = VS.pop();
          M = frame.M; // Restore previous scope
          S = frame.S; // Restore previous instruction pointer
          continue;
      }
    } catch (e) {
      // Manual Exception Handling (Bubble up virtual stack)
      handleVirtualException(e, M, VS);
    }
  }
};
```

### Polymorphic Dispatchers

The engine structures the main execution loop using one of four strategies to evade heuristic analysis and signature detection:

1.  **🔥 Chaos Dispatcher (Maximum Security):**
    *   **Horcrux State Variables:** The Instruction Pointer `S` is split into three interdependent variables (`K1`, `K2`, `K3`). The actual state is derived via `S = K1 ^ K2 ^ K3 ^ SALT`.
    *   **Graph Explosion:** Injects "Trampoline" states (useless hops) and "Alias" states (duplicate entry points) to artificially inflate the CFG.
    *   **Honey Pots:** Injects fake branches protected by mathematical predicates. If executed by a debugger forcing a path, they trigger infinite loops or memory corruption.

2.  **📦 Cluster Dispatcher:**
    *   **Hierarchical Bucketing:** Groups states into "Clusters" based on `MaskedID % BucketCount`.
    *   **Hybrid Routing:** Dynamically chooses between Switch statements, Binary Search Trees, or Linear scans for each bucket.

3.  **🌳 BST Dispatcher:**
    *   **Algorithmic Complexity:** Organizes state cases into a **Binary Search Tree** (O(log N)).
    *   **Code Scattering:** Scatters logic blocks non-linearly, making sequential reading impossible.

4.  **⚙️ Switch Dispatcher:**
    *   **Performance:** Uses a standard `switch(S)` statement. Fastest execution, ideal for performance-critical hot paths.

### Memory & Scope Virtualization

*   **Virtual Heap (`M`):** All local variables are converted to integer indices in a flat array (e.g., `var a = 1` becomes `M[4] = 1`).
*   **Global Proxy (`GM`):** External APIs (e.g., `document`, `console`) are identified during static analysis and preloaded into a Global Memory array, removing their string references from the bytecode.
*   **Closure Snapshots:** When a function creates a closure, VortexJS captures specific memory indices from the parent scope and passes them into the new virtual frame.

### The Generator & Async Protocol

VortexJS fully supports modern JS concurrency patterns by mapping `yield` and `await` to VM interrupts.
*   **AWAIT OpCode:** The VM suspends, attaches a `.then()` handler to the Promise, and returns. The handler re-invokes the VM with the resolved value when ready.
*   **YIELD OpCode:** The VM returns a specific token to the iterator protocol wrapper, preserving the stack `VS` for the next `.next()` call.

---

## 🔒 Security Features

### Hyperwave String Encryption
Strings are not merely encoded; they are projected into N-dimensional space.
*   **Geometric Transformation:** Data is mapped to coordinates (2D-5D).
*   **Wave Interference:** Procedurally generated sine waves distort the data points based on a specialized seed.
*   **Runtime Decryption:** A polymorphic `decoder` function reconstructs the string only at the moment of access.

### Opaque Predicates
To harden the Control Flow Graph against static analysis, the compiler injects conditions that evaluate to a known result at runtime but are mathematically complex to solve statically.
*   **Math Congruence:** `(a * b) % n === ((a % n) * (b % n)) % n` (Always True).
*   **Array Aliasing:** Creates two references to the same array, modifies one, and checks the other.
*   **VM State History:** Simulates a Linear Congruential Generator (LCG) to verify execution path integrity.
*   **Anti-Debug:** Measures execution time of tight loops (Timing Attack) to detect stepping/breakpoints.

### Anti-Tamper Mechanisms
*   **Honey Pots:** Fake code blocks that look legitimate but contain `while(true)` loops or memory corruption instructions. Reachable only if a reverse engineer forces a jump.
*   **Integrity Checks:** (Chaos Mode) If the Horcrux variables `K1/K2/K3` desynchronize from `S`, the VM resets or crashes.

---

## ⚙ Compiler Pipeline

The transformation process involves three major stages:

### 1. IR Generation (AST -> IR)
The source code is parsed into an AST, then lowered into a flat **Intermediate Representation (IR)**. High-level constructs (Classes, Loops) are broken down into primitive OpCodes (`ASSIGN`, `BINARY`, `GOTO`, `COND_JUMP`).

### 2. IR Optimization (Level 3)
The `IROptimizer` performs aggressive multi-pass optimization:
*   **Global Dead Store Elimination (DCE):** Removes assignments to registers that are never read.
*   **Constant Folding & Propagation:** Pre-calculates expressions like `1 + 2`.
*   **Expression Reassociation:** Simplifies math chains (`x + 1 + 2` → `x + 3`).
*   **Jump Threading:** Short-circuits jumps that point to other jumps.
*   **Tail Call Optimization (TCO):** Converts recursive calls into `GOTO` instructions.
*   **Superblock Merging:** Coalesces basic blocks to reduce dispatcher overhead.

### 3. Code Generation (IR -> VM Bytecode)
The finalized IR is mapped to the chosen Dispatcher (Switch/BST/Chaos) and emitted as a new JavaScript bundle containing the VM runtime.

---

## 📦 Installation

Requires **Node.js v18** or higher.

```bash
# Clone the repository
git clone https://github.com/SSL-ACTX/vortex-js.git

# Install dependencies
cd vortex-js
npm install

# Link command globally
npm link
```

---

## 💻 Usage & Configuration

**Try it online:** [VortexJS Live Compiler](https://vortexjs-server.onrender.com/)

CLI Usage:
```bash
vortex <inputFile> <outputFile> [flags]
```

### Command Flags

| Flag | Category | Description |
| :--- | :--- | :--- |
| `--min` | **Output** | Minify the final output using `esbuild`. |
| `--terser` | **Output** | Use `terser` for aggressive minification (slower, smaller). |
| `--no-post` | **Output** | Raw output mode (no minification/formatting) for debugging. |
| `--dispatcher <type>` | **Core** | Select strategy: `switch` (default), `bst`, `cluster`, `chaos`. |
| `--superblock <size>` | **Perf** | Max ops merged into a single block. Higher = faster, less granular graph. |
| `--opq` | **Security** | Enable **Opaque Predicates** (Control Flow Hardening). |
| `--opq-lvl <level>` | **Security** | `low`, `medium`, `high`. High includes Anti-Debug & LCG checks. |
| `--opq-prob <0-1>` | **Security** | Probability of injection per block (Default: 0.2). |
| `--randomize-ids` | **Security** | Randomize state IDs (integers) to prevent sequence analysis. |
| `--anti-debug` | **Security** | Inject timing-attack based debugger detection. |
| `--no-enc` | **Utility** | Disable string encryption (strings remain plain text). |
| `--watch` | **Utility** | Watch input file for changes and rebuild automatically. |
| `--run` | **Utility** | Execute the output file immediately after build. |

---

## ✍️ Authors & Credits

*   **Lead Engineer:** [Seuriin](https://github.com/SSL-ACTX)
*   **Concept & Architecture:**  [Seuriin](https://github.com/SSL-ACTX)

---

## ⚖ License

This project is licensed under the **AGPL-3.0 License**.

> [!CAUTION]
> **Disclaimer:** This tool is an educational research project demonstrating compiler theory and virtualization techniques. The author is not responsible for any misuse of this software.
