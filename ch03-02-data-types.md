## Data Types

Rust me har value ka ek *data type* hota hai, jo Rust ko batata hai ki kaunsa type ka data use ho raha hai aur Rust kaise uske sath kaam kare. Hum do data type subsets dekhenge: scalar aur compound.

Yaad rahe ki Rust ek *statically typed* language hai, iska matlab hai ki compiler ko compile time par saare variables ke types pata hone chahiye. Compiler aksar infer kar leta hai ki hum kaunsa type use karna chahte hain value aur use ke tarike se. Lekin jab multiple types possible ho, jaise ki humne Chapter 2 me “Comparing the Guess to the Secret Number” section me `String` ko numeric type me `parse` se convert kiya, tab hume type annotation add karna padta hai, jaise:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Agar hum yahan type annotation nahi dete, Rust ye error dikhata hai, iska matlab compiler ko pata nahi ki kaunsa type use karna hai:

```text
error[E0282]: type annotations needed
 --> src/main.rs:2:9
  |
2 |     let guess = "42".parse().expect("Not a number!");
  |         ^^^^^
  |         |
  |         cannot infer type for `_`
  |         consider giving `guess` a type
```

Aap dusre data types ke liye alag type annotations bhi dekhenge.

### Scalar Types

*Scalar* type ek single value ko represent karta hai. Rust me char, Boolean, floating-point aur integers ke 4 primary scalar types hote hain. Ye types aapko dusre programming languages me bhi milenge. Ab dekhte hain Rust me ye kaise kaam karte hain.

#### Integer Types

*Integer* ek number hota hai jisme fractional component nahi hota. Chapter 2 me humne `u32` integer type use kiya tha. Ye type indicate karta hai ki value ek unsigned integer hai (signed integers `i` se start hote hain, `u` ki jagah) aur 32 bits space leti hai. Table 3-1 me Rust ke built-in integer types diye hain. Signed aur Unsigned columns ke har variant (jaise `i16`) integer value declare karne ke liye use ho sakte hain.

| Length | Signed  | Unsigned |
| ------ | ------- | -------- |
| 8-bit  | `i8`    | `u8`     |
| 16-bit | `i16`   | `u16`    |
| 32-bit | `i32`   | `u32`    |
| 64-bit | `i64`   | `u64`    |
| arch   | `isize` | `usize`  |

Signed aur unsigned ka matlab hai ki number negative ho sakta hai ya sirf positive hoga. Signed numbers two’s complement representation me store hote hain. Signed variant numbers -(2<sup>n - 1</sup>) se 2<sup>n - 1</sup> - 1 ke range me store karte hain. Unsigned variant 0 se 2<sup>n</sup> - 1 ke range me store karta hai. `isize` aur `usize` architecture ke hisaab se size lete hain (32-bit ya 64-bit).

Integer literals ko aap alag forms me likh sakte hain aur type suffix aur visual separator `_` bhi use kar sakte hain.

| Number literals  | Example       |
| ---------------- | ------------- |
| Decimal          | `98_222`      |
| Hex              | `0xff`        |
| Octal            | `0o77`        |
| Binary           | `0b1111_0000` |
| Byte (`u8` only) | `b'A'`        |

Agar unsure ho kaunsa type use kare, Rust ka default `i32` generally sahi choice hai.

#### Floating-Point Types

Floating-point numbers decimal points ke sath numbers hote hain. Rust ke floating-point types `f32` aur `f64` hain. Default `f64` hai kyunki modern CPUs me f32 ke speed ke barabar hai aur jyada precision deta hai.

```rust
fn main() {
    let x = 2.0; // f64
    let y: f32 = 3.0; // f32
}
```

Floating-point numbers IEEE-754 standard ke according represent hote hain.

#### Numeric Operations

Rust basic mathematical operations support karta hai: addition, subtraction, multiplication, division, remainder.

```rust
fn main() {
    let sum = 5 + 10;
    let difference = 95.5 - 4.3;
    let product = 4 * 30;
    let quotient = 56.7 / 32.2;
    let remainder = 43 % 5;
}
```

#### Boolean Type

Boolean type me sirf do values ho sakti hain: `true` aur `false`. Boolean type Rust me `bool` se define hota hai.

```rust
fn main() {
    let t = true;
    let f: bool = false;
}
```

Boolean values ko conditionals jaise `if` me use kiya jata hai.

#### Character Type

Rust ke `char` type letters aur Unicode characters ko represent karta hai. Single quotes use hote hain.

```rust
fn main() {
    let c = 'z';
    let z = 'ℤ';
    let heart_eyed_cat = '😻';
}
```

`char` Unicode Scalar Value represent karta hai, jo ASCII se bahut jyada characters cover karta hai.

### Compound Types

*Compound types* multiple values ko ek type me group karte hain. Rust ke primitive compound types tuples aur arrays hain.

#### Tuple Type

Tuple alag-alag types ke values ko ek group me store karta hai.

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    let (x, y, z) = tup;
    println!("The value of y is: {}", y);
}
```

Tuple elements ko direct index se access bhi kar sakte hain:

```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);
    let five_hundred = x.0;
    let six_point_four = x.1;
    let one = x.2;
}
```

#### Array Type

Array me sabhi elements same type ke hote hain aur fixed length ke hote hain.

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
    let first = a[0];
    let second = a[1];
}
```

Agar array index out of bounds try karte hain, program *panic* karta hai:

```rust,ignore
fn main() {
    let a = [1, 2, 3, 4, 5];
    let index = 10;
    let element = a[index];
    println!("The value of element is: {}", element);
}
```

Rust runtime safety principles ke liye index check karta hai aur invalid memory access se bachata hai.
