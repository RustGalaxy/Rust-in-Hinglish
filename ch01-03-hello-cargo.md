## Hello, Cargo!

Cargo Rust ka build system aur package manager hai. Zyadatar Rustaceans Cargo use karte hain apne Rust projects manage karne ke liye kyunki ye kaafi tasks handle karta hai: code build karna, libraries download karna jo code depend karta hai, aur un libraries ko build karna. (Libraries jise code need karta hai, unhe hum *dependencies* kehte hain.)

Simple programs jaise Hello, world! me dependencies nahi hoti. Agar humne Hello, world! Cargo se banaya hota, to bas build part use hota. Jaise jaise programs complex hote hain, dependencies add karna easy ho jata hai agar project Cargo se start ho.

Kyuki zyadatar Rust projects Cargo use karte hain, is book me bhi assume kiya gaya hai ki aap Cargo use kar rahe ho. Agar official installer se Rust install kiya hai, Cargo automatically installed hoga. Check karne ke liye:

```text
$ cargo --version
```

Agar version number dikh raha hai, matlab installed hai. Error aaye jaise `command not found`, to installation method ki documentation check karo.

### Creating a Project with Cargo

Ab naya project create karte hain Cargo se. Apne *projects* directory me jao aur command run karo:

```text
$ cargo new hello_cargo --bin
$ cd hello_cargo
```

* Ye ek binary executable *hello\_cargo* banata hai.
* `--bin` ka matlab executable banega, library nahi.
* Cargo files ussi name ki directory me create karta hai.

Directory me jaake files check karo:

* *Cargo.toml* file
* *src* directory with *main.rs*
* Git repository with *.gitignore* file

> Note: Git common version control system hai. `--vcs` flag se alag VCS ya none choose kar sakte ho. `cargo new --help` run karo options ke liye.

*Cargo.toml* file example:

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
```

* `[package]` section heading hai.
* Name, version, author details yaha set hote hain.
* `[dependencies]` section me project ki dependencies list hoti hain. Rust me code packages ko *crates* kehte hain.

*src/main.rs* me:

```rust
fn main() {
    println!("Hello, world!");
}
```

* Cargo ne Hello, world! program generate kiya.
* Code *src* directory me hai, aur top level me *Cargo.toml*.
* Cargo source files *src* me expect karta hai, top-level directory me config aur README files hote hain.

### Building and Running a Cargo Project

*hello\_cargo* directory me build karo:

```text
$ cargo build
```

* Executable create hoga *target/debug/hello\_cargo* (Windows: *.exe*).
* Run:

```text
$ ./target/debug/hello_cargo # Windows: .\target\debug\hello_cargo.exe
Hello, world!
```

* `cargo build` se top level pe *Cargo.lock* file bhi generate hoti hai, dependencies ke versions track karne ke liye.

Compile aur run ek saath:

```text
$ cargo run
Hello, world!
```

* Agar files change nahi hui, Cargo bas binary run karega. Change kiya ho to rebuild karega.

Fast check ke liye:

```text
$ cargo check
```

* Sirf compile check karta hai, executable generate nahi karta.
* `cargo check` fast hota hai, `cargo build` se.

Recap Cargo:

* `cargo build` aur `cargo check` se project build hota hai.
* `cargo run` se build aur run ek saath.
* Build result *target/debug* me store hota hai.
* Commands OS independent hain.

### Building for Release

Release ke liye:

```text
$ cargo build --release
```

* Executable *target/release* me create hota hai, optimized.
* Development fast rebuild ke liye debug profile use hota hai, release fast code ke liye release profile.

### Cargo as Convention

Simple projects me `rustc` kaafi hai, lekin complex projects me Cargo ka value dikhata hai. Multiple crates wale projects me build coordination easy hota hai.

Existing projects me:

```text
$ git clone someurl.com/someproject
$ cd someproject
$ cargo build
```

More info: [Cargo Documentation](https://doc.rust-lang.org/cargo/)

## Summary

Is chapter me aapne seekha:

* `rustup` se latest stable Rust install karna
* Rust version update karna
* Local documentation access karna
* `rustc` se Hello, world! program run karna
* Cargo ke conventions ke sath new project create aur run karna

Next, Chapter 2 me guessing game program banayenge ya Chapter 3 me Rust programming concepts samjhenge.
