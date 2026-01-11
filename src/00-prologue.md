# PROLOGUE: FIRST LAUNCH

## Why Rust

Imagine a world where only one programming language exists. One tool for everything: from a blinking LED on a microcontroller to a distributed cluster processing millions of requests per second. From a system kernel to a web interface in the browser. From a network protocol parser to a game engine.

This world is not a utopia. This is Rust.

(And no, it's not a cult. Although, honestly, sometimes it looks like one.)

Rust solves a fundamental problem in programming: how to get full control over hardware without shooting yourself in the foot with every release. Three mechanisms make this possible:

**Borrow checker** – a static analyzer that proves memory safety. If the program compiles – an entire class of bugs becomes physically impossible.

**Zero-cost abstractions** – high-level constructs without runtime overhead. You don't pay for what you don't use.

**Fearless concurrency** – a type system that eliminates data races at compile time.

A skeptic might say: "Sounds like marketing, but does it actually work in practice?"

It works. Three of the largest tech companies publish metrics on Rust adoption – and the results speak for themselves.

### Google Android

Google has been integrating Rust into Android since 2019. By 2025, the platform contains about 5 million lines of Rust code. Results:

- **Memory vulnerability density**: approximately 0.2 per million lines of code. In all that time, one potential vulnerability was discovered – and it was fixed before release.
- **Change rollbacks**: 4 times less frequent.
- **Code review time**: 25% less.
- **Number of code revisions**: 20% less.

In 2025, more lines of Rust were added to Android than any other systems language. The share of memory vulnerabilities dropped below 20% for the first time in the platform's history.

Jeff Vander Stoep from Google: *"The safer path is now also the faster one."*

### Microsoft Azure

Mark Russinovich, CTO of Microsoft Azure, issued a directive several years ago: "Speaking of languages, it's time to halt starting any new projects in C/C++ and use Rust for those scenarios where a non-GC language is required. For the sake of security and reliability. The industry should declare those languages as deprecated". Rewriting results:

**DirectWrite Core** (Windows font parser): 152,000 lines of Rust. Two developers, six months. Text rendering performance improved by approximately 5-15% – without special optimizations, just from the rewrite.

**Azure Data Explorer**: 350,000 lines of Rust. Processes petabytes of data daily, billions of queries.

**Hyperlight**: micro-VM startup in 0.9 milliseconds.

**OpenVMM**, **Caliptra**, **Azure Integrated HSM** – critical security infrastructure, entirely in Rust.

This is no longer just an experiment. For the past couple of years, not only Windows but also all the low-level software around it has been rusting. In 2023, the first Windows kernel syscall written in Rust appeared. And in the Windows 11 24H2 release, a full-fledged Rust driver win32kbase_rs.sys runs in the graphics subsystem for managing memory regions in GDI – intercepting an entire class of LPE vulnerabilities. The print stack is protected by Rust parsers – PrintNightmare is closed at the architectural level. But most importantly: Microsoft released the Windows Driver Kit with official Rust support. This means any antivirus, device driver, or chipset developer can write kernel code in a memory-safe language.

Microsoft is also developing tools for automatic legacy code migration:
- **Eurydice** – a transpiler for verified cryptographic code that preserves formal verification
- **GraphRAG** – using LLMs with hierarchical retrieval for automatic code translation

Russinovich: *"The faster we can accelerate migration, the better – we’re 100% behind Rust."*

### Adobe

Adobe manages a codebase of 78 million lines of legacy code, a massive zoo of languages and scripts. A complete rewrite is impossible – the company chose an incremental transition strategy.

Sean Parent, Senior Principal Scientist: *"We're within about 15% performance of the C++ implementation. In some cases, the Rust implementation actually wins."*

The main public project is **C2PA** (Coalition for Content Provenance and Authenticity): a library for content authenticity verification and combating deepfakes. Entirely in Rust, open source.

According to Adobe's internal data: developers reach productivity in Rust in about 2 months. Two-thirds of them feel confident enough to work on real products.

David Sankel, Principal Scientist at Adobe: *"Putting the best tools in the hands of our engineers means creating better products for our customers."*

---

## Who This Book Is For

Let's clarify right away: **this is not an introduction to Computer Science.**

This book doesn't teach programming from scratch. If terms like "function argument," "scope," or "iteration" raise questions – you'd better start with simpler introductory courses and come back here later.

This material is designed for an engineer who:

1.  **Already has development experience.** It doesn't matter what language your past projects were written in. What matters is that you already know how to turn algorithms into code and have faced real-world tasks.
2.  **Understands memory layout.** At least intuitively: how data on the stack differs from data on the heap and why it matters.
3.  **Isn't afraid of the terminal.** Working with Rust tools happens primarily on the command line.

Rust is primarily a systems language. It removes the layer of "magic" and forces you to consider what is usually swept under the rug. If you're ready for this – welcome.

---

## How to Read This Book

The book's structure is unusual. It consists of a **Core** and **Specialized Appendices**.

### Core (Parts I–VIII)
This is a direct path from syntax to production. We'll write what 90% of Rust developers write: CLI utilities, web services, database work, and distributed systems. This is the mandatory program.

### Appendices F and G (Hardcore Mode)
This is the "rabbit hole" for those who find just shuffling JSONs boring.
*   **Appendix F**: Evolutionary algorithms and life simulations.
*   **Appendix G**: An engineering model of consciousness (Global Workspace Theory).

These chapters use Rust for modeling complex systems. They are optional. **You can become an excellent Rust developer without ever opening them.** But if you're interested in how to create a virtual population or model an attention mechanism – look for links like `→ Appendix F.2` at the end of core chapters.

### Quality Filters
At the end of each part, there's a "Quality Filter" checklist. These aren't just topic lists, but specific skills. For example: *"Draw a memory diagram for `&mut T` in 30 seconds."* Don't move forward until you've checked all the boxes.

This structure was chosen deliberately. This book's goal is to guide you through all the necessary stages as efficiently as possible. There won't be theory for theory's sake. Every chapter is a project. Every project is a step to the next level of complexity.

But before writing code, you need to set up your environment. The Rust ecosystem is famous for its convenience, and you'll soon see this for yourself.

---

## Installing Tools

### rustup – Rust Version Manager

The official way to install Rust is through rustup.

On Linux/macOS:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

On Windows:

Install Visual Studio 2022 Community. During installation, select "Desktop development with C++". Then download and run [rustup-init.exe](https://rustup.rs).

After installation, restart your terminal and verify:

```bash
rustc --version
cargo --version
```
If both commands respond – Rust is installed.
rustup manages toolchains. The stable version is your main tool. For some advanced features, you'll need nightly:

```bash
rustup toolchain install nightly
rustup default stable
```

Switching between versions:

```bash
rustup override set nightly  # for current directory
rustup default nightly       # globally
```

### Cargo – The Heart of the Ecosystem

Cargo is simultaneously:
- Dependency manager
- Build system
- Test runner
- Documentation generator
- Package publisher

One tool instead of a zoo. One command – one task.

Key commands:

```bash
cargo new project_name       # create new project
cargo build                  # build (debug)
cargo build --release        # build (release, with optimizations)
cargo run                    # build and run
cargo test                   # run tests
cargo check                  # check without generating binary (fast)
cargo doc --open             # generate documentation and open
cargo clippy                 # static analyzer (linter)
cargo fmt                    # code formatting
```

### IDE: Zed

**Zed** – an editor written in Rust, for Rust development (and more). Fast, minimalist, with built-in language support out of the box.

Installation: [zed.dev](https://zed.dev)

Zed automatically detects Rust projects and activates rust-analyzer. No additional configuration required – open a folder with `Cargo.toml` and start working.

Useful hotkeys:

| Action | macOS | Linux |
|--------|-------|-------|
| Go to definition | `Cmd+Click` or `F12` | `Ctrl+Click` or `F12` |
| Find all references | `Shift+F12` | `Shift+F12` |
| Rename symbol | `F2` | `F2` |
| Show actions (fix) | `Cmd+.` | `Ctrl+.` |
| Command palette | `Cmd+Shift+P` | `Ctrl+Shift+P` |

Settings in `~/.config/zed/settings.json`:

```json
{
  "lsp": {
    "rust-analyzer": {
      "initialization_options": {
        "check": {
          "command": "clippy"
        }
      }
    }
  }
}
```

**Alternatives:**

- **VS Code + rust-analyzer** – popular choice, extension from marketplace
- **RustRover** – full IDE from JetBrains with debugger and profiler
- **Neovim + rust-analyzer** – for those who live in the terminal

rust-analyzer (used by all of the above) provides:
- Type-aware autocompletion
- Inline type and lifetime hints
- Go to definition/implementation
- Rename refactoring
- Expand macro (see what a macro expands to)

### Additional Tools

```bash
rustup component add clippy rustfmt    # linter and formatter (already installed with default profile)
cargo install cargo-watch              # automatic rebuild on changes
cargo install cargo-expand             # view expanded macros
```

`cargo watch` – must-have for development:

```bash
cargo watch -x check           # check on every save
cargo watch -x test            # tests on every save
cargo watch -x "run -- args"   # run with arguments
```

---

## First Project: Hello World

```bash
cargo new hello_rust
cd hello_rust
```

Project structure:

```
hello_rust/
├── Cargo.toml
└── src/
    └── main.rs
```

`Cargo.toml` – project manifest:

```toml
[package]
name = "hello_rust"
version = "0.1.0"
edition = "2024"

[dependencies]
```

`edition = "2024"` – language version. Rust releases new editions every three years (2015, 2018, 2021, 2024). Editions allow introducing changes without losing backward compatibility: old code compiles with the old edition, new code – with the new one. Crates of different editions are fully compatible with each other.

`src/main.rs`:

```rust
fn main() {
    println!("Hello, Rust!");
}
```

Run:

```bash
cargo run
```
Output:

```
   Compiling hello v0.1.0 (.../hello_rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.50s
     Running `target/debug/hello`
Hello, world!
```

`println!` – a macro, not a function. The exclamation mark indicates a macro. Macros in Rust are a powerful metaprogramming tool. `println!` takes a format string with placeholders:

```rust
fn main() {
    let name = "world";
    let answer = 42;
    println!("Hello, {}! Answer: {}", name, answer);
    println!("Debug: {:?}", (name, answer));
    println!("Pretty debug:\n{:#?}", vec![1, 2, 3]);
}
```

`{}` – Display formatting (for users)
`{:?}` – Debug formatting (for developers)
`{:#?}` – Debug with pretty-print

---

## Syntax Capsule: Basic Syntax

### Variables

```rust
let x = 5;           // immutable variable
let mut y = 10;      // mutable variable
y = 15;              // ok
// x = 6;            // compilation error
```

By default, variables are **immutable**. These aren't constants – they're variables that cannot be changed after initialization. For a mutable variable, you need `mut`.

Why? Immutability by default makes code safer and clearer. If a variable doesn't change – its value can be traced without reading all the code.

### Types

Rust is statically typed, but the compiler often infers types automatically:

```rust
let x = 5;              // i32 (default integer)
let y = 3.14;           // f64 (floating point)
let flag = true;        // bool
let ch = 'A';           // char (Unicode character)
let text = "hello";     // &str (string slice)
```

Explicit type annotation:

```rust
let x: i32 = 5;
let y: f64 = 3.14;
```

Main numeric types:
- `i8`, `i16`, `i32`, `i64`, `i128` – signed integers
- `u8`, `u16`, `u32`, `u64`, `u128` – unsigned integers
- `f32`, `f64` – floating point numbers
- `usize`, `isize` – architecture-dependent (32/64 bit)

### Functions

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b    // no semicolon = return value
}

fn greet(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let result = add(5, 3);
    greet("Rust");
}
```

**Key points:**
- Parameter types are required
- `->` indicates return type
- The last expression without `;` is the return value
- `return` can be used for early exit

### Strings

Two main types:

```rust
let s1: &str = "hello";                  // string slice
let s2: String = String::from("hello");  // owned string
```

`&str` – a reference to string data (immutable, often in program code).
`String` – an owned type, can grow and change.

Details in Chapter 5. For now, use `&str` for literals and `String::from()` when you need an owned string.

---

## What's Next?

You've set up your environment and run your first program. Now you have everything you need to start writing real code.

Rust requires investment upfront. The compiler will reject code that at first glance seems to work. This isn't a bug, it's a feature. And when you internalize this, you'll get:

- "If it compiles – it works" – not a joke, but daily reality
- Fearless refactoring – the compiler will find all the places that need to change
- Multithreading without sleepless nights – the type system eliminates data races
- Performance without sacrifices – you get both speed and safety

In Chapter 1, we'll write the first real project and get acquainted with control flow: conditions, loops, and collections.

---
