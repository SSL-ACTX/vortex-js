# Contributing to VortexJS

First off, thanks for taking the time to contribute! 🚀

VortexJS is a complex **source-to-source compiler** and **stackless virtualization engine**. Because of the sensitive nature of the logic (handling ASTs, IR, and custom memory management), we ask that you read this guide before making changes.

## 🛠 Development Setup

### Prerequisites
*   **Node.js** v18.0.0 or higher
*   **NPM** or **Yarn**

### Installation
1.  **Fork** the repository on GitHub.
2.  **Clone** your fork locally:
    ```bash
    git clone https://github.com/your-username/vortex-js.git
    cd vortex-js
    ```
3.  **Install dependencies**:
    ```bash
    npm install
    ```
4.  **Link the CLI** (Optional, for easy testing):
    ```bash
    npm link
    ```

---

## 🏗 Project Architecture

To contribute effectively, you must understand the compilation pipeline. VortexJS does not simply "minify" code; it recompiles it.

### The Pipeline (`src/transformer.js`)
1.  **Parsing:** Babel parses raw JS into an AST.
2.  **Analysis:** `StringCollector` and `ScopeAnalysis` map variables to virtual memory indices (`M[0]`, `M[1]`, etc.).
3.  **IR Generation (`src/ir-gen/`):** The AST is flattened into **Intermediate Representation (IR)** states.
    *   *Control Flow:* `if/else`, `loops` are converted to `GOTO` and `COND_JUMP`.
    *   *Expressions:* Complex expressions are broken into atomic `ASSIGN` or `BINARY` ops.
4.  **Optimization (`src/ir-optimizer.js`):** The IR passes through Level-3 optimizations (DCE, Constant Folding, Superblock merging).
5.  **Code Generation (`src/ast-gen/`):** The optimized IR is wrapped in a Dispatcher (Switch/BST/Chaos) and emitted as a new AST.

### Directory Structure
*   `src/ir-gen/`: Logic for converting JS Statements/Expressions to IR.
*   `src/ir-optimizer.js`: The optimization pass. **Touch with caution.**
*   `src/dispatcher/`: Strategies for the VM loop (Switch, Chaos, BST).
*   `src/obfuscation/`: Opaque predicates and string encryption logic.
*   `src/string-collector/`, `src/string-concealer/` : String collection and encryption logic.

---

## 🧪 Testing & Debugging

Because VortexJS generates a Virtual Machine, standard unit tests are often insufficient. You must verify that the **output code** runs correctly.

### Recommended Workflow
1.  Create a test file (e.g., `test/input.js`) with complex logic (recursion, classes, async).
2.  Run the compiler in **Raw Mode** (no minification, no encryption):
    ```bash
    node index.js test/input.js test/output.js --no-post --no-enc
    ```
3.  Inspect `test/output.js`. Ensure the IR states make sense.
4.  Run the output to verify behavior matches the input:
    ```bash
    node test/output.js
    ```

### Debugging the VM
If the VM crashes or hangs:
1.  Use the **Switch Dispatcher** (`--dispatcher switch`) for easier readability.
2.  Add `console.log(S)` inside the generated `while(true)` loop to trace the state.

---

## 📐 Coding Standards

*   **Linting (Optional):** Ensure code passes ESLint rules (if configured).
*   **Modern JS:** Use ES6+ syntax.
*   **Immutability:** When modifying the AST, prefer immutable transformations where possible to prevent side effects in the `path` traversal.

### IR Rules
If you add a new Feature (e.g., support for `with` statements):
1.  **Add OpCode:** Define the new OpCode in `ir-generator.js`.
2.  **Implement Handler:** Update `base-dispatcher.js` to handle the execution logic.
3.  **Update Optimizer:** Ensure `ir-optimizer.js` knows how to handle (or ignore) this new OpCode.

---

## 🔏 Pull Request Process

1.  **Branch:** Create a feature branch from `main` (e.g., `feat/add-generator-support` or `fix/memory-leak`).
2.  **Commit:** Use **Conventional Commits** for your messages.
    *   `feat: ...` for new features.
    *   `fix: ...` for bug fixes.
    *   `perf: ...` for IR optimization improvements.
    *   `docs: ...` for documentation.
3.  **Verify:** Ensure `test/output.js` runs successfully for basic inputs.
4.  **Open PR:** Describe your changes and the reasoning behind them.

---

## 📜 License

By contributing, you agree that your contributions will be licensed under the MIT License defined in the root of this repository.
