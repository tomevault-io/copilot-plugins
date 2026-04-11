## bebopescrow

> This document outlines the principles and practices of a highly effective Soroban smart contract developer. Adherence to these rules ensures the creation of secure, efficient, and maintainable contracts on the Stellar network.

# 10x Rust Soroban Smart Contract Developer Rule Set

This document outlines the principles and practices of a highly effective Soroban smart contract developer. Adherence to these rules ensures the creation of secure, efficient, and maintainable contracts on the Stellar network.

---

### 1. Security is Paramount

*   **Think Adversarially:** Always code as if you are under attack. Assume all external calls and inputs are potentially malicious.
*   **Checks-Effects-Interactions:** Strictly follow this pattern to prevent reentrancy attacks. Perform all validation checks first, then update state, and only then interact with other contracts.
*   **No Panics:** A panic in a contract is a critical failure. Use `Result` and proper error handling for all fallible operations. Never let an arithmetic overflow go unhandled.
*   **Authentication & Authorization:** Every sensitive function must be protected with robust authentication and authorization checks. Use Soroban's built-in crypto primitives and identity verification.
*   **Audit Rigorously:** Before deploying to mainnet, subject your code to at least one independent security audit. Treat all findings, even low-severity ones, with seriousness.

### 2. Code Quality & Craftsmanship

*   **Clarity Over Cleverness:** Write code that is easy to read and understand. The next developer to read your code (which might be you in six months) will thank you.
*   **Idiomatic Rust & Soroban:** Leverage Rust's type system, ownership model, and standard library features. Use Soroban-specific types (`Symbol`, `Vec`, `Map`, etc.) and host functions as intended.
*   **Modularity:** Keep contracts and their modules small and focused on a single responsibility.
*   **Documentation is Non-Negotiable:**
    *   Add `#[doc]` comments to all public functions, structs, and enums.
    *   Explain the *why* behind complex logic, not just the *what*.
    *   Provide a high-level description of the contract's purpose and architecture in the `lib.rs` file.

### 3. Rigorous & Comprehensive Testing

*   **Test Every Path:** Aim for 100% line and branch coverage in your unit tests. Every public function must have a corresponding suite of tests.
*   **Property-Based Testing:** Use tools like `proptest` to generate a wide range of inputs and test for invariants and edge cases that you might not think of manually.
*   **Integration Testing:** Write tests that simulate real-world scenarios and interactions between your contract and others. Use the Soroban test environment to its full potential.
*   **Test Snapshots:** Use snapshot testing to easily track and verify complex data structures returned by your contract.

### 4. Gas & Performance Optimization

*   **Profile Before Optimizing:** Use the Soroban profiler to identify actual gas bottlenecks. Do not guess.
*   **Minimize State:** Every read from and write to storage costs gas. Design your contract to keep state minimal and perform as much computation off-chain as possible.
*   **Efficient Data Structures:** Choose the right data structure for the job. Understand the performance characteristics of `Vec`, `Map`, and other collections in the Soroban environment.
*   **Batch Operations:** When possible, design functions to perform batch operations to reduce the overhead of multiple separate calls.

### 5. Master the Toolchain

*   **Soroban CLI Proficiency:** Be an expert with the `soroban-cli`. Know how to build, deploy, inspect, and invoke contracts efficiently from the command line.
*   **Cargo Ecosystem:** Leverage `cargo` for dependency management, running tests, and integrating tools like `rustfmt` for formatting and `clippy` for linting.
*   **Stay Updated:** The Soroban and Stellar ecosystems are constantly evolving. Regularly update your toolchains and dependencies, and stay informed about new features and best practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/christopherkarani)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/christopherkarani)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
