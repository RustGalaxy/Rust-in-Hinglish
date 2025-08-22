## Vah Sabhi Jagah Jahan Patterns ka Istemal Ho Sakta Hai

Patterns Rust mein kai jagahon par aate hain, aur aap unka bahut istemal kar rahe hain, shayad bina ehsaas kiye\! Yah section un sabhi jagahon par charcha karta hai jahan patterns valid hain.

-----

### `match` Arms

Jaisa ki Chapter 6 mein charcha ki gayi hai, hum `match` expressions ke arms mein patterns ka istemal karte hain. Formally, `match` expressions is tarah se define kiye jaate hain:

```
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

Udaharan ke liye, yahan Listing 6-5 se `match` expression hai jo `x` variable mein ek `Option<i32>` value par match karta hai:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Is `match` expression mein patterns har arrow ke baayin or `None` aur `Some(i)` hain.

`match` expressions ke liye ek avashyakta (requirement) yah hai ki ve **exhaustive** (sampoorn) hone chahiye, is arth mein ki `match` expression mein value ke liye sabhi sambhavnaon ko cover kiya jana chahiye. Ek catch-all pattern, jaise ki ek variable naam ya `_` wildcard, aakhri arm mein rakhkar yah sunishchit kiya ja sakta hai.

-----

### `let` Statements

Is chapter se pehle, humne keval `match` aur `if let` ke saath patterns ka istemal karne par spasht roop se charcha ki thi, lekin vastav mein, humne `let` statements sahit anya jagahon par bhi patterns ka istemal kiya hai. Udaharan ke liye, is seedhe `let` ke saath variable assignment par vichaar karein:

```rust
let x = 5;
```

Har baar jab aapne is tarah ka `let` statement istemal kiya hai, aapne patterns ka istemal kiya hai\! Adhik formally, ek `let` statement is tarah dikhta hai:

```
let PATTERN = EXPRESSION;
```

`let x = 5;` jaise statements mein, variable naam (`x`) pattern ka ek bahut hi saral roop hai. Iska matlab hai "jo bhi yahan match hota hai use variable `x` se bind karo."

`let` ke pattern-matching pehlu ko aur spasht roop se dekhne ke liye, Listing 19-1 par vichaar karein, jo ek tuple ko **destructure** karne ke liye `let` ke saath ek pattern ka istemal karta hai.

```rust
// Filename: src/main.rs
fn main() {
    let (x, y, z) = (1, 2, 3);
    println!("x = {x}, y = {y}, z = {z}");
}
```

*Listing 19-1: Ek tuple ko destructure karne aur ek saath teen variables banane ke liye ek pattern ka istemal karna*

Yahan, Rust value `(1, 2, 3)` ko pattern `(x, y, z)` se compare karta hai aur dekhta hai ki value pattern se match karti hai, isliye Rust `1` ko `x` se, `2` ko `y` se, aur `3` ko `z` se bind karta hai.

Agar pattern mein elements ki sankhya tuple mein elements ki sankhya se match nahi karti hai, to humein ek compiler error milega.

-----

### Conditional `if let` Expressions

`if let` expressions `match` ka ek chhota roop hain jo keval ek case ko match karta hai. Isteemaal ke anusaar, `if let` mein ek `else` block bhi ho sakta hai. Listing 19-3 dikhata hai ki `if let`, `else if`, aur `else if let` expressions ko mix karna bhi sambhav hai.

```rust
// Filename: src/main.rs
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

*Listing 19-3: `if let`, `else if`, `else if let`, aur `else` ko mix karna*

`if let` ka ek nuksaan yah hai ki compiler exhaustiveness ki jaanch nahi karta, jabki `match` expressions mein vah karta hai.

-----

### `while let` Conditional Loops

`if let` ke samaan, `while let` conditional loop ek `while` loop ko tab tak chalne deta hai jab tak ek pattern match karta rehta hai. Listing 19-4 mein ek `while let` loop hai jo ek channel se messages prapt karta hai jab tak ki `recv()` `Ok` return karta hai.

```rust
// Filename: src/main.rs
# use std::sync::mpsc::{channel, Sender, Receiver};
# use std::thread;
# use std::time::Duration;
fn main() {
    let (tx, rx): (Sender<i32>, Receiver<i32>) = channel();
    thread::spawn(move || {
        tx.send(1).unwrap();
        tx.send(2).unwrap();
        tx.send(3).unwrap();
    });

    while let Ok(value) = rx.recv() {
        println!("{value}");
    }
}
```

*Listing 19-4: `rx.recv()` jab tak `Ok` return karta hai, tab tak values print karne ke liye `while let` loop ka istemal karna*

-----

### `for` Loops

Ek `for` loop mein, `for` keyword ke theek baad aane wali value ek pattern hai. Udaharan ke liye, `for x in y` mein, `x` pattern hai. Listing 19-5 `for` loop mein ek tuple ko destructure karne ke liye ek pattern ka istemal karna dikhata hai.

```rust
// Filename: src/main.rs
fn main() {
    let v = vec!['a', 'b', 'c'];

    for (index, value) in v.iter().enumerate() {
        println!("{} is at index {}", value, index);
    }
}
```

*Listing 19-5: Ek tuple ko destructure karne ke liye `for` loop mein ek pattern ka istemal karna*

Output hoga:

```console
a is at index 0
b is at index 1
c is at index 2
```

Hum `enumerate` method ka istemal karke ek iterator se `(index, value)` ka tuple prapt karte hain, jise `for` loop seedhe destructure kar leta hai.

-----

### Function Parameters

Function parameters bhi patterns ho sakte hain.

```rust
// Filename: src/main.rs
fn foo(x: i32) {
    // code goes here
}
fn main() { foo(5); }
```

*Listing 19-6: Ek function signature parameters mein patterns ka istemal karta hai*

`x` wala hissa ek pattern hai\! Jaisa ki humne `let` ke saath kiya, hum ek function ke arguments mein ek tuple ko pattern se match kar sakte hain.

```rust
// Filename: src/main.rs
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

*Listing 19-7: Parameters ke saath ek function jo ek tuple ko destructure karta hai*

Yah code `Current location: (3, 5)` print karta hai. `&(3, 5)` value `&(x, y)` pattern se match karti hai, isliye `x` ki value `3` aur `y` ki value `5` hai.

Ab tak, aapne patterns ka istemal karne ke kai tareeke dekhe hain, lekin patterns har jagah jahan hum unka istemal kar sakte hain, ek jaise kaam nahi karte. Kuch jagahon par, patterns ko irrefutable hona chahiye; anya paristhitiyon mein, ve refutable ho sakte hain. Hum in do concepts par aage charcha karenge.