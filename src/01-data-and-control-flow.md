# CHAPTER 1. DATA AND CONTROL FLOW

In the prologue, we set up our environment and ran Hello World. Now it's time to write something useful.

We'll start with a simple task: a password generator. Along the way, we'll get acquainted with the `Option` and `Result` types — the two pillars of error handling in Rust. Then we'll study control flow: conditionals, loops, pattern matching. And we'll transform a simple generator into an interactive application.

---

## First Project: Password Generator

Create the project:

```bash
cargo new passgen
cd passgen
```

### Dependencies

For random number generation, we'll use the `rand` crate. Add the dependency to `Cargo.toml`:

```toml
[dependencies]
rand = "0.9.2"
```

Or via command:

```bash
cargo add rand
```

### Code

`src/main.rs`:

```rust
use rand::Rng;

fn generate_password(length: usize) -> String {
    const CHARSET: &[u8] = b"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*";

    let mut rng = rand::rng();

    (0..length)
        .map(|_| {
            let idx = rng.random_range(0..CHARSET.len());
            CHARSET[idx] as char
        })
        .collect()
}

fn main() {
    let password = generate_password(16);
    println!("Generated password: {}", password);
}
```

Run:

```bash
cargo run
```

Output:

```
Generated password: n*EbG1KXg2CAiIBQ
```

### Code Breakdown

**`use rand::Rng;`**
Importing the `Rng` trait from the `rand` crate. A trait is a set of methods that a type can implement. `Rng` provides methods for generating random numbers. More on traits in Chapter 3.


**`const CHARSET: &[u8]`**
A constant — a byte slice containing valid password characters.
`b"..."` — a byte string literal, where each element has type `u8`.


**`let mut rng = rand::rng();`**
Creating a random number generator.
`mut` is required because the generator maintains internal state and changes with each call. This is an important point: even "reading" a random number is a state change of the generator.


**`(0..length)`**
A half-open range from `0` to `length` (upper bound excluded). Used to specify the number of characters to generate.


**`.map(|_| { ... })`**
Applying a closure to each element of the range.
`|_|` — a closure with an ignored argument, since the range index value isn't used. More on closures and iterators in Chapter 5.


**`rng.random_range(0..CHARSET.len())`**
Generating a uniformly distributed random index within the length of the `CHARSET` array.


**`CHARSET[idx] as char`**
Getting the byte at the index and casting it to `char`.
This is correct in this case since `CHARSET` contains only ASCII characters.


**`.collect()`**
Collecting the sequence of `char` into a collection.
The `String` type is inferred by the compiler from the function's return type.

---

## Option and Result

The program above works, but what if the user passes length 0? Or requests reading a non-existent file? In most languages, such situations are handled through exceptions or returning null. Rust takes a different path (algebraic types).

### Option — Something or Nothing

In Rust, a variable cannot be "just empty". To express the absence of a value, the `Option<T>` wrapper is used:

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 1 {
        Some(String::from("Lenie"))
    } else {
        None
    }
}

fn main() {
    let user = find_user(1);
    
    match user {
        Some(name) => println!("Found: {}", name),
        None => println!("Not found"),
    }
}
```

`Option<T>` has two variants:
- `Some(value)` — value is present
- `None` — value is absent

The compiler **forces** you to handle both cases. You can't forget to check for null.

### Result — Success or Failure

Unlike `Option` (value exists or not), `Result<T, E>` tells us **why** the operation failed:

```rust
use std::fs;

fn read_config() -> Result<String, std::io::Error> {
    fs::read_to_string("config.txt")
}

fn main() {
    match read_config() {
        Ok(content) => println!("Configuration: {}", content),
        Err(e) => println!("Error: {}", e),
    }
}
```

`Result<T, E>`:
- `Ok(value)` — operation succeeded
- `Err(error)` — an error occurred

### The ? Operator

Syntactic sugar for propagating errors:

```rust
use std::fs;
use std::io;

fn read_and_process() -> Result<(), io::Error> {
    let content = fs::read_to_string("data.txt")?;
    println!("Read {} bytes", content.len());
    Ok(())
}
```

`?` works like this:
- If `Ok(value)` — extracts `value`
- If `Err(e)` — immediately returns `Err(e)` from the function

Only works in functions returning `Result` or `Option`.

Compare:

```rust
// Without the ? operator
fn read_file() -> Result<String, io::Error> {
    let content = match fs::read_to_string("file.txt") {
        Ok(c) => c,
        Err(e) => return Err(e),
    };
    Ok(content)
}

// With the ? operator
fn read_file() -> Result<String, io::Error> {
    let content = fs::read_to_string("file.txt")?;
    Ok(content)
}
```

### unwrap() and expect() — Extraction with Panic

Sometimes you need to extract a value knowing that an error is impossible (or its presence means a critical bug):

```rust
let config = std::fs::read_to_string("config.txt").unwrap();
```

**`unwrap()`** works like this:
- If `Ok(value)` → returns `value`
- If `Err(e)` → the program **panics** (terminates abnormally)

**`expect(message)`** — same thing, but with an explanation:

```rust
let config = std::fs::read_to_string("config.txt")
    .expect("File config.txt must exist");
```

On panic it will print:
```
thread 'main' panicked at 'File config.txt must exist: ...'
```

**When to use:**

✓ **Prototypes and examples** — when error handling doesn't matter:
```rust
let number = "42".parse::<i32>().unwrap();
```

✓ **True invariants** — when an error means a critical bug in logic:
```rust
let mutex = Arc::new(Mutex::new(data));
let guard = mutex.lock().unwrap();  // If fail — deadlock, this is a critical bug
```

✗ **Production with external data** — files, network, user input:
```rust
// BAD: file might not exist
let config = std::fs::read_to_string("user_data.json").unwrap();

// GOOD: error handling
let config = std::fs::read_to_string("user_data.json")?;
```

---

> **Real-life story: the unwrap() that took down the internet**
>
> On November 18, 2025, Cloudflare experienced a major outage. X, ChatGPT, Canva, and thousands of other services went down. The cause: one `unwrap()` in the Rust code of the FL2 proxy server.
>
> The Bot Management system used a configuration file with a limit of 200 features (usually there were about 60). After a permissions change in the database, the file suddenly grew — and exceeded the limit. The code checked the size, got an error... and called `unwrap()`:
>
> ```
> thread fl2_worker_thread panicked: called Result::unwrap() on an Err value
> ```
>
> The panic spread across Cloudflare's global network. Approximately six hours to full recovery.
>
> Moral: `unwrap()` is not "I'm sure there won't be an error". It's "if an error happens, let the whole world burn". In prototypes — acceptable. In production — only for true invariants: situations where an error means a bug in the program's logic, not a problem with external data.
>
> (To their credit, Cloudflare published a full postmortem with source code. Transparency worthy of respect.)

---

### Improving the Generator: Parameterization

Let's add the ability to specify password length via command line argument:

```rust
use rand::Rng;
use std::env;

fn generate_password(length: usize) -> String {
    const CHARSET: &[u8] = b"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*";

    let mut rng = rand::rng();

    (0..length)
        .map(|_| {
            let idx = rng.random_range(0..CHARSET.len());
            CHARSET[idx] as char
        })
        .collect()
}

fn parse_length() -> Result<usize, String> {
    let args: Vec<String> = env::args().collect();

    if args.len() < 2 {
        return Ok(16);
    }

    args[1]
        .parse::<usize>()
        .map_err(|_| format!("Invalid length: {}", args[1]))
}

fn main() {
    match parse_length() {
        Ok(length) => {
            let password = generate_password(length);
            println!("Generated password: {}", password);
        }
        Err(e) => {
            eprintln!("Error: {}", e);
            std::process::exit(1);
        }
    }
}
```

Run:

```bash
cargo run          # length 16 by default
cargo run -- 24    # length 24
```

**What's new:**

**`env::args()`**
Returns an iterator of command line arguments.

**`.collect()`**
Collects the iterator into a `Vec<String>`.

**`.parse::<usize>()`**
Parsing a string to a number. Returns `Result<usize, ParseIntError>`.

**`.map_err(|_| ...)`**
Transforming the error type. `ParseIntError` → `String`.

**`eprintln!`**
Printing to stderr (standard error stream).

**`std::process::exit(1)`**
Terminating the program with return code 1 (error).

---

## Syntax Capsule

The password generator works, but it's linear: start → execute → finish. Real programs make decisions, repeat actions, store collections of data. Let's teach our program to think.

### Conditional Expressions: if/else

Basic form:

```rust
fn main() {
    let number = 7;
    
    if number < 5 {
        println!("Number is less than 5");
    } else if number < 10 {
        println!("Number is less than 10");
    } else {
        println!("Number is 10 or greater");
    }
}
```

**Key features:**

1. Condition **doesn't require parentheses**
2. Condition must be of type `bool` (not `0` or `1`)
3. `if` is an expression, can return a value:

```rust
let number = 5;
let result = if number > 0 {
    "positive"
} else {
    "non-positive"
};
println!("{}", result);
```

Both branches must return the same type:

```rust
// Compilation error:
let x = if true { 5 } else { "hello" };

// Correct:
let x = if true { 5 } else { 0 };
```

### Pattern Matching: match

`match` — a powerful tool for working with enumerations and patterns:

```rust
fn describe_number(n: i32) {
    match n {
        0 => println!("zero"),
        1 | 2 => println!("one or two"),
        3..=5 => println!("three to five"),
        _ => println!("something else"),
    }
}
```

**Key points:**

1. **Exhaustive matching** — all variants must be covered
2. `_` — universal pattern (catch-all)
3. `|` — alternative (or)
4. `..=` — inclusive range

`match` is also an expression:

```rust
let result = match value {
    0 => "zero",
    1 => "one",
    _ => "many",
};
```

Tuple destructuring:

```rust
let point = (3, 5);

match point {
    (0, 0) => println!("Origin"),
    (x, 0) => println!("On X axis: {}", x),
    (0, y) => println!("On Y axis: {}", y),
    (x, y) => println!("Point ({}, {})", x, y),
}
```

Working with `Option` and `Result`:

```rust
fn divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None
    } else {
        Some(a / b)
    }
}

fn main() {
    let result = divide(10.0, 2.0);
    
    match result {
        Some(value) => println!("Result: {}", value),
        None => println!("Division by zero"),
    }
}
```

Guard expressions:

```rust
let number = Some(4);

match number {
    Some(x) if x < 5 => println!("Less than five: {}", x),
    Some(x) => println!("Five or greater: {}", x),
    None => println!("Nothing"),
}
```

### Loops: loop, while, for

#### loop — Infinite Loop

```rust
let mut count = 0;

loop {
    count += 1;
    println!("{}", count);
    
    if count == 5 {
        break;
    }
}
```

`loop` can return a value through `break`:

```rust
let mut counter = 0;

let result = loop {
    counter += 1;
    
    if counter == 10 {
        break counter * 2;
    }
};

println!("Result: {}", result);  // 20
```

Labels for nested loops:

```rust
'outer: loop {
    println!("Outer loop");
    
    loop {
        println!("Inner loop");
        break 'outer;  // exit from outer loop
    }
}
```

#### while — Conditional Loop

```rust
let mut number = 3;

while number != 0 {
    println!("{}!", number);
    number -= 1;
}

println!("Liftoff!");
```

#### for — Collection Iteration

```rust
let a = [10, 20, 30, 40, 50];

for element in a {
    println!("value: {}", element);
}
```

Ranges:

```rust
// 1..5 -> 1, 2, 3, 4 (without 5)
for n in 1..5 {
    println!("{}", n);
}

// 1..=5 -> 1, 2, 3, 4, 5 (including 5)
for n in 1..=5 {
    println!("{}", n);
}
```

With indices:

```rust
let names = ["Alice", "Bob", "Carol"];

for (index, name) in names.iter().enumerate() {
    println!("{}: {}", index, name);
}
```

Reverse order:

```rust
for n in (1..5).rev() {
    println!("{}", n);
}
```

### Arrays — Fixed Size

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
let first = a[0];
let second = a[1];

// All elements the same:
let zeros = [0; 100];  // [0, 0, 0, ..., 0]
```

Arrays:
- Fixed size (part of the type)
- Data on the stack
- Fast index access

### Vec — Dynamic Array

```rust
let mut v: Vec<i32> = Vec::new();

v.push(1);
v.push(2);
v.push(3);

println!("{:?}", v);  // [1, 2, 3]
```

The `vec!` macro:

```rust
let v = vec![1, 2, 3, 4, 5];
```

Element access:

```rust
let v = vec![1, 2, 3, 4, 5];

// Method 1: panic on out of bounds
let third = v[2];

// Method 2: safe access
match v.get(2) {
    Some(value) => println!("Third element: {}", value),
    None => println!("No third element"),
}
```

Iteration:

```rust
let v = vec![10, 20, 30];

for value in &v {
    println!("{}", value);
}

// Modifying elements:
let mut v = vec![10, 20, 30];

for value in &mut v {
    *value += 50;
}

println!("{:?}", v);  // [60, 70, 80]
```

Useful methods:

```rust
let mut v = vec![1, 2, 3];

v.push(4);              // add to end
let last = v.pop();     // extract from end (Option<T>)
let len = v.len();      // length
let is_empty = v.is_empty();  // is empty

v.clear();              // clear
```

---

## Project: Interactive Password Generator

Let's transform the generator into an interactive application with a simple menu.

**Requirements:**
- Menu: Generate / Change length / Exit
- Loop until explicit exit

```bash
cargo new passgen_interactive
cd passgen_interactive
cargo add rand
```

### Implementation

`src/main.rs`:

```rust
use rand::Rng;
use std::io::{self, Write};

fn generate_password(
    length: usize,
    use_upper: bool,
    use_lower: bool,
    use_digits: bool,
    use_special: bool,
) -> Option<String> {
    let mut charset = Vec::new();

    if use_upper {
        charset.extend_from_slice(b"ABCDEFGHIJKLMNOPQRSTUVWXYZ");
    }
    if use_lower {
        charset.extend_from_slice(b"abcdefghijklmnopqrstuvwxyz");
    }
    if use_digits {
        charset.extend_from_slice(b"0123456789");
    }
    if use_special {
        charset.extend_from_slice(b"!@#$%^&*");
    }

    if charset.is_empty() {
        return None;
    }

    let mut rng = rand::rng();

    let password: String = (0..length)
        .map(|_| {
            let idx = rng.random_range(0..charset.len());
            charset[idx] as char
        })
        .collect();

    Some(password)
}

fn read_line() -> String {
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read input");
    input.trim().to_string()
}

fn print_menu() {
    println!("\n=== PASSWORD GENERATOR ===");
    println!("1. Generate password");
    println!("2. Change length");
    println!("3. Exit");
    print!("Choice: ");
    io::stdout().flush().unwrap();
}

fn main() {
    // Settings as separate variables
    let mut length = 16;
    let use_upper = true;
    let use_lower = true;
    let use_digits = true;
    let use_special = true;

    loop {
        print_menu();
        let choice = read_line();

        match choice.as_str() {
            "1" => {
                match generate_password(length, use_upper, use_lower, use_digits, use_special) {
                    Some(password) => {
                        println!("\nPassword: {}", password);
                        println!("Length: {} characters", password.len());
                    }
                    None => {
                        println!("Error: no available characters");
                    }
                }
            }
            "2" => {
                print!("New length (4-128): ");
                io::stdout().flush().unwrap();
                let input = read_line();

                match input.parse::<usize>() {
                    Ok(new_len) if new_len >= 4 && new_len <= 128 => {
                        length = new_len;
                        println!("Length changed to {}", length);
                    }
                    Ok(_) => {
                        println!("Length must be between 4 and 128");
                    }
                    Err(_) => {
                        println!("Invalid number");
                    }
                }
            }
            "3" => {
                println!("¡Adiós!");
                break;
            }
            _ => {
                println!("Invalid choice");
            }
        }
    }
}
```

### Code Breakdown

#### Parameter Passing

```rust
fn generate_password(
    length: usize,
    use_upper: bool,
    use_lower: bool,
    use_digits: bool,
    use_special: bool,
) -> Option<String> {
```

Five parameters — this is inconvenient. Every call requires passing all of them:

```rust
generate_password(length, use_upper, use_lower, use_digits, use_special)
```

Looks cumbersome? It is. In Chapter 3, we'll learn to pack related data into structures.

#### Building charset:

```rust
let mut charset = Vec::new();

if use_upper {
    charset.extend_from_slice(b"ABCDEFGHIJKLMNOPQRSTUVWXYZ");
}
```

We create an empty vector and add characters depending on flags. `extend_from_slice` adds all elements of a slice to the end of the vector.

#### Reading input:

```rust
fn read_line() -> String {
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read input");
    input.trim().to_string()
}
```

`read_line` adds a newline character — `trim()` removes it.

#### Main loop:

```rust
loop {
    print_menu();
    let choice = read_line();

    match choice.as_str() {
        "1" => { /* generation */ }
        "2" => { /* change length */ }
        "3" => break,
        _ => println!("Invalid choice"),
    }
}
```

An infinite loop, interrupted when "3" is selected. `match` handles all input variants. The `_` pattern catches everything that didn't match previous variants.

#### Guard in match:

```rust
match input.parse::<usize>() {
    Ok(new_len) if new_len >= 4 && new_len <= 128 => {
        length = new_len;
    }
    Ok(_) => {
        println!("Length must be between 4 and 128");
    }
    Err(_) => {
        println!("Invalid number");
    }
}
```

`if new_len >= 4 && new_len <= 128` — this is a guard, an additional condition for the pattern. It allows splitting `Ok` into two cases: valid and invalid value.

---

## Project: Wordcount Utility

Let's write a full CLI utility, analogous to the Unix `wc` (word count) command.

**Functionality:**
- Count lines, words, characters, bytes
- Process multiple files
- Flags for selecting count type
- Total line when processing multiple files

```bash
cargo new wordcount
cd wordcount
```

### Implementation

`src/main.rs`:

```rust
use std::env;
use std::fs;

fn count_stats(content: &str) -> (usize, usize, usize, usize) {
    let lines = content.lines().count();
    let words = content.split_whitespace().count();
    let chars = content.chars().count();
    let bytes = content.len();

    (lines, words, chars, bytes)
}

fn print_stats(
    lines: usize,
    words: usize,
    chars: usize,
    bytes: usize,
    show_lines: bool,
    show_words: bool,
    show_chars: bool,
    show_bytes: bool,
    filename: &str,
) {
    if show_lines {
        print!("{:>8}", lines);
    }
    if show_words {
        print!("{:>8}", words);
    }
    if show_chars {
        print!("{:>8}", chars);
    }
    if show_bytes {
        print!("{:>8}", bytes);
    }
    if !filename.is_empty() {
        print!(" {}", filename);
    }
    println!();
}

fn main() {
    let args: Vec<String> = env::args().skip(1).collect();

    // Flags as separate variables
    let mut show_lines = false;
    let mut show_words = false;
    let mut show_chars = false;
    let mut show_bytes = false;
    let mut files: Vec<String> = Vec::new();

    // Argument parsing
    for arg in &args {
        if arg.starts_with('-') && arg.len() > 1 {
            for c in arg[1..].chars() {
                if c == 'l' {
                    show_lines = true;
                } else if c == 'w' {
                    show_words = true;
                } else if c == 'c' {
                    show_bytes = true;
                } else if c == 'm' {
                    show_chars = true;
                } else {
                    eprintln!("wordcount: unknown flag: -{}", c);
                }
            }
        } else {
            files.push(arg.clone());
        }
    }

    // If no flags specified — enable defaults
    if !show_lines && !show_words && !show_chars && !show_bytes {
        show_lines = true;
        show_words = true;
        show_bytes = true;
    }

    // Process files
    let mut total_lines = 0;
    let mut total_words = 0;
    let mut total_chars = 0;
    let mut total_bytes = 0;
    let mut file_count = 0;

    for path in &files {
        match fs::read_to_string(path) {
            Ok(content) => {
                let (lines, words, chars, bytes) = count_stats(&content);

                print_stats(
                    lines, words, chars, bytes,
                    show_lines, show_words, show_chars, show_bytes,
                    path,
                );

                total_lines += lines;
                total_words += words;
                total_chars += chars;
                total_bytes += bytes;
                file_count += 1;
            }
            Err(e) => {
                eprintln!("wordcount: {}: {}", path, e);
            }
        }
    }

    // Total line
    if file_count > 1 {
        print_stats(
            total_lines, total_words, total_chars, total_bytes,
            show_lines, show_words, show_chars, show_bytes,
            "total",
        );
    }

    // If no files specified
    if files.is_empty() {
        eprintln!("Usage: wordcount [-lwcm] <files...>");
        eprintln!("Flags: -l (lines), -w (words), -c (bytes), -m (characters)");
    }
}
```

### Code Breakdown

#### Counting Statistics

```rust
fn count_stats(content: &str) -> (usize, usize, usize, usize) {
    let lines = content.lines().count();
    let words = content.split_whitespace().count();
    let chars = content.chars().count();
    let bytes = content.len();

    (lines, words, chars, bytes)
}
```

The function returns a tuple of four values. This is a temporary solution — in Chapter 3, we'll learn to create structures for such cases.


`.lines()` — iterator over lines.

`.split_whitespace()` — iterator over words (delimiter — any whitespace).

`.chars()` — iterator over Unicode characters.

`.len()` — number of bytes (not characters!).

#### Argument Parsing:

```rust
for arg in &args {
    if arg.starts_with('-') && arg.len() > 1 {
        for c in arg[1..].chars() {
            if c == 'l' {
                show_lines = true;
            } else if c == 'w' {
                show_words = true;
            }
            // ...
        }
    } else {
        files.push(arg.clone());
    }
}
```

`arg[1..]` — string slice, all characters except the first (removing the hyphen).
`.chars()` — character iterator, allows processing `-lwc` as three separate flags.


#### Error Handling:

```rust
match fs::read_to_string(path) {
    Ok(content) => {
        // processing
    }
    Err(e) => {
        eprintln!("wordcount: {}: {}", path, e);
    }
}
```

On file read error, we output a message to stderr but continue processing the remaining files. This is standard behavior for Unix utilities.

### Testing

Create test files:

```bash
echo "Hello world" > test1.txt
echo -e "Line one\nLine two\nLine three" > test2.txt
```

Run:

```bash
cargo run -- test1.txt
cargo run -- -l test1.txt              # lines only
cargo run -- -wl test1.txt             # words and lines
cargo run -- test1.txt test2.txt       # multiple files
```

### Unit Tests

Add a test module at the end of the file:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_empty_string() {
        let (lines, words, chars, bytes) = count_stats("");
        assert_eq!(lines, 0);
        assert_eq!(words, 0);
        assert_eq!(bytes, 0);
    }

    #[test]
    fn test_single_line() {
        let (lines, words, _, bytes) = count_stats("hello world");
        assert_eq!(lines, 1);
        assert_eq!(words, 2);
        assert_eq!(bytes, 11);
    }

    #[test]
    fn test_multiple_lines() {
        let (lines, words, _, _) = count_stats("line one\nline two\nline three");
        assert_eq!(lines, 3);
        assert_eq!(words, 6);
    }

    #[test]
    fn test_unicode() {
        let (_, words, chars, bytes) = count_stats("Hello world");
        assert_eq!(words, 2);
        assert_eq!(chars, 11);
        assert_eq!(bytes, 11);
    }

    #[test]
    fn test_whitespace_handling() {
        let (_, words, _, _) = count_stats("  many    spaces   ");
        assert_eq!(words, 2);
    }
}
```

Run tests:

```bash
cargo test
```
Now let's break down the magic incantations we just wrote:

1.  **`#[cfg(test)]`**
    This is a conditional compilation attribute. It tells Rust: "The entire `tests` module should only be compiled and included in the binary **when** I run `cargo test`". In a normal build (`cargo build`), this code simply disappears and won't take up space.

2.  **`mod tests { ... }`**
    In Rust, tests are conventionally written in the same file as the code, but isolated in a separate module. This allows testing private functions without polluting the main namespace.

3.  **`use super::*;`**
    Since tests live in a nested module (`tests`), they don't see functions from the parent module (`main`). This line imports everything (`*`) from the parent module (`super`) into the test scope.

4.  **`#[test]`**
    Marks a function as a test. `cargo test` looks for exactly these markers.

5.  **`assert_eq!(left, right)`**
    An assertion macro. Checks that the left argument equals the right. If not — panics and outputs both values, which counts as a test failure.

---

## Exercises

We wrote prototypes of two programs (MVPs). Your task is to turn them into full-fledged tools that are convenient to use. Choose your difficulty level.

### Level 1: Intern

**Task 1.1: Help for Wordcount**

Add handling for the `-h` or `--help` flag. If the user runs `wordcount --help`, the program should print usage instructions and exit.

**Task 1.2: Safe Input in PassGen**

In the interactive generator, we use `.expect()` when reading input. What happens if stdin closes? Replace `expect` with `Result` handling — on error, print a message and terminate the program gracefully.

### Level 2: Developer

**Task 2.1: Password Strength Assessment**

Add a function to `PassGen` that assesses the generated password:
- < 8 characters → "Weak"
- ≥ 8 characters, letters only → "Medium"
- ≥ 8 characters, letters + digits → "Good"
- ≥ 12 characters, letters + digits + special characters → "Strong"

Display the assessment after generation.

**Task 2.2: Finding the Longest Word**

Add a `-M` (max) flag to `wordcount`. When present, the program should not count statistics but find and display the longest word in the file.

*Hint:* use a variable `let mut longest = "";` and a loop over `content.split_whitespace()`.

### Level 3: Engineer

**Task 3.1: Clipboard**

Make `PassGen` offer to copy the password to clipboard.

*Challenge:* Rust doesn't work with the clipboard out of the box. Go to [crates.io](https://crates.io), find a suitable crate (e.g., `arboard` or `cli-clipboard`), add it via `cargo add`, and figure out the documentation.

**Task 3.2: Reading from stdin**

The real `wc` utility reads from stdin if no files are specified:

```bash
echo "Hello Rust" | wc
```

Implement the same behavior. If the file list is empty, read data from `std::io::stdin()`.

*Hint:* `io::stdin().read_to_string(&mut buffer)` works similarly to reading a file.

---

## Bonus: Weasel Program

In 1986, Richard Dawkins published "The Blind Watchmaker" — a response to the argument about the impossibility of complex structures arising by chance. Creationists argued: the probability of a functional protein or meaningful text appearing randomly is so small that it requires an intelligent designer.

Dawkins proposed an elegant demonstration. Take a phrase from Shakespeare:

```
METHINKS IT IS LIKE A WEASEL
```

The probability of typing this string randomly is about 1 in 10^40. Even if every atom in the Universe generated a million strings per second since the Big Bang, we wouldn't have enough time.

But evolution works differently. It doesn't roll the dice anew each time — it **accumulates** successful changes.

Dawkins's algorithm:
1. Start with a random string of the same length
2. Create several copies with small mutations
3. Select the copy most similar to the target
4. Repeat until the string matches the target

Result: instead of 10^40 attempts, a few hundred generations are sufficient. Cumulative selection turns the astronomically improbable into the inevitable.

The Weasel Program is an ideal playground for practicing with this chapter's constructs: `loop` for generations, `Vec` for population, `if` for counting matches, loops for mutations and selection.

`→ Appendix F.1: Weasel Program — complete implementation with analysis`

---

## ✓ Quality Filter

You've mastered this chapter if you can:

- [ ] Explain the difference between `Option` and `Result`
- [ ] Use the `?` operator for error propagation
- [ ] Explain when `unwrap()` is appropriate and when it's not
- [ ] Understand why a random number generator requires `mut`
- [ ] Write an `if`/`else` expression that returns a value
- [ ] Write a `match` for `Option` without looking
- [ ] Use guard expressions in `match`
- [ ] Explain the difference between `loop`, `while`, and `for`
- [ ] Write a `for` loop with indices (`enumerate`)
- [ ] Use `break` with a label to exit a nested loop
- [ ] Create a `Vec`, add elements, iterate over it
- [ ] Handle command line arguments via `env::args()`
- [ ] Read a file with error handling
- [ ] Write a test with `#[test]` and `assert_eq!`

**If even one checkbox isn't checked — go back to the corresponding section.**

---

## What's Next?

You've written an application with a text menu and a classic system utility. You've learned to handle errors, control execution flow, work with collections.

But the code turned out cumbersome. The `generate_password` function takes five parameters. `print_stats` — nine. This is inconvenient and unreliable: it's easy to mix up the argument order, hard to add new ones.

**Chapter 2** will show how Rust manages memory without a garbage collector. Why are some data copied while others are moved? What is ownership and borrowing?

**Chapter 3** will give you a tool for fighting parameter chaos: structures, enumerations, and traits. We'll return to these projects and make the code clean.

---
