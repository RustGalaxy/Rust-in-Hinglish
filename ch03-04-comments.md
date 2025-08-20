## Comments

Rust ke code ko aasan banane ke liye, **programmers** apne code mein notes chhodte hain, jise hum **comments** kehte hain. Compiler in comments ko ignore kar deta hai, lekin code padhne wale logon ke liye yeh bahut useful hote hain.

### Single-line Comments

Rust mein, comments do (//) slashes se shuru hote hain aur line ke aakhir tak rehte hain. Jaise:

```rust
// hello, world
```

Agar aapka comment ek se zyada line ka hai, to aapko har line par `//` lagana padega:

```rust
// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.
```

Aap comments ko code wali line ke aakhir mein bhi likh sakte hain:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let lucky_number = 7; // I’m feeling lucky today
}
```

Lekin, jyadatar log comments ko code se pehle ek alag line mein likhna pasand karte hain, jisse code aur bhi saaf dikhe:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    // I’m feeling lucky today
    let lucky_number = 7;
}
```

-----

### Documentation Comments

Rust mein ek aur tarah ke comments hote hain, jise hum **documentation comments** kehte hain. Inke baare mein hum **Chapter 14** mein detail mein discuss karenge.
