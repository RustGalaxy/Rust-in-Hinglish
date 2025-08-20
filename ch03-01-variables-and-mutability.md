## Variables and Mutability

Jaise ki Chapter 2 me bataya gaya, by default variables immutable hote hain. Ye Rust ka ek tarika hai aapko encourage karne ka ki aap apna code is tarah likhein jo Rust ke safety aur easy concurrency ka faida utha sake. Lekin, aap fir bhi apne variables ko mutable bana sakte hain. Chaliye dekhte hain kaise aur kyu Rust aapko immutability prefer karne ke liye encourage karta hai aur kab kab aap mutable use karna chahenge.

Jab ek variable immutable hota hai, ek baar value bind ho jaye to aap us value ko change nahi kar sakte. Isko illustrate karne ke liye, naya project banaiye *variables* naam se aapke *projects* directory me: `cargo new --bin variables`.

Phir, apne naye *variables* directory me, *src/main.rs* open karke code replace karein is code se (jo abhi compile nahi hoga):

```rust,ignore
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

Program run karne ke liye `cargo run` use karein. Aapko error milega:

```text
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         - first assignment to `x`
3 |     println!("The value of x is: {}", x);
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
```

Ye example dikhata hai kaise compiler aapke program me errors dhundta hai. Compiler errors frustrating lag sakte hain, lekin iska matlab ye nahi hai ki aap ache programmer nahi hain! Experienced Rustaceans bhi compiler errors se guzar te hain.

Error message batata hai ki problem ye hai ki aap `cannot assign twice to immutable variable x`, kyunki aapne immutable `x` me second value assign karne ki koshish ki.

Rust me compiler guarantee karta hai ki agar aapne ek value ko immutable declare kiya hai, to wo value kabhi change nahi hogi. Isse aapko code read/write karte waqt value ke change hone ka dhyan rakhne ki zarurat nahi hoti. Aapka code easy to reason hota hai.

### Mutability

Lekin mutability kabhi kabhi useful hoti hai. Variables by default immutable hote hain; jaise Chapter 2 me kiya, aap `mut` add karke variable ko mutable bana sakte hain. `mut` use karne se aap value change kar sakte hain aur code padhne walon ko signal milta hai ki ye variable change hoga.

Example:

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

Run karne par output:

```text
The value of x is: 5
The value of x is: 6
```

### Differences Between Variables and Constants

Immutable variable yaad dilata hai ek aur concept: *constants*. Constants bhi ek name se bind hote hain aur change nahi hote, lekin kuch differences hain:

* Constants me `mut` use nahi hota. Ye hamesha immutable hote hain.
* Constants declare karte hain `const` keyword se aur type annotation mandatory hai.
* Constants kisi bhi scope me declare ho sakte hain (global bhi), useful jab value ko multiple parts me access karna ho.
* Constants me sirf constant expression assign hota hai, function call ka result nahi.

Example:

```rust
const MAX_POINTS: u32 = 100_000;
```

### Shadowing

Jaise guessing game me dekha, aap same name ka naya variable declare kar sakte hain jo purane ko shadow karega. Ye Rust me *shadowing* kehlata hai. Example:

```rust
fn main() {
    let x = 5;
    let x = x + 1;
    let x = x * 2;
    println!("The value of x is: {}", x);
}
```

Output:

```text
The value of x is: 12
```

Shadowing `mut` se different hai, kyunki ye new variable create karta hai aur type bhi change kar sakte hain:

```rust
let spaces = "   ";
let spaces = spaces.len();
```

Agar `mut` use karte, to error aata:

```rust,ignore
let mut spaces = "   ";
spaces = spaces.len();
```

Error:

```text
error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected &str, found usize
```

Ab humne variables ke bare me cover kar liya, agla step hai data types explore karna.
