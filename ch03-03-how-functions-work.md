## Functions

Rust code mein **functions** har jagah hote hain. Aapne pehle hi ek sabse important function dekha hai: `main` function, jo bahut se programs ka entry point hota hai. Aapne `fn` keyword bhi dekha hai, jo aapko naye functions declare karne deta hai.

Rust code mein function aur variable ke naam ke liye conventional style **snake case** use kiya jaata hai. Snake case mein, sabhi letters lowercase hote hain aur words ko **underscores** se alag kiya jaata hai. Yahaan ek program hai jisme ek example function definition hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

Rust mein function definitions `fn` se shuru hote hain aur function name ke baad ek set of parentheses hote hain. Curly brackets compiler ko batate hain ki function body kahan shuru aur kahan khatam hoti hai.

Hum kisi bhi function ko jo humne define kiya hai, uske naam ke baad parentheses lagakar call kar sakte hain. Kyunki `another_function` program mein define kiya gaya hai, isse `main` function ke andar se call kiya jaa sakta hai. Note karein ki humne `another_function` ko `main` function ke **baad** define kiya hai source code mein; hum isse pehle bhi define kar sakte the. Rust ko farak nahi padta ki aap apne functions kahan define karte hain, bas woh kahin na kahin define hone chahiye.

Aaiye, functions ko aur explore karne ke liye ek naya binary project **functions** naam se shuru karte hain. `another_function` example ko `src/main.rs` mein rakhein aur isse run karein. Aapko neeche diya gaya output dekhna chahiye:

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28 secs
     Running `target/debug/functions`
Hello, world!
Another function.
```

Lines us order mein execute hoti hain jisme woh `main` function mein appear hoti hain. Pehle, “Hello, world\!” message print hota hai, aur phir `another_function` ko call kiya jaata hai aur uska message print hota hai.

-----

### Function Parameters (Function Parameters)

Functions ko **parameters** ke saath bhi define kiya jaa sakta hai, jo special variables hote hain aur function ke signature ka hissa hote hain. Jab kisi function mein parameters hote hain, toh aap un parameters ke liye concrete values provide kar sakte hain. Technically, concrete values ko **arguments** kehte hain, lekin casual conversation mein log **parameter** aur **argument** words ko interchangeably use karte hain, chahe woh function ki definition mein variables hon ya concrete values jo function call karte waqt pass ki jaati hain.

`another_function` ka neeche diya gaya rewritten version dikhata hai ki Rust mein parameters kaise dikhte hain:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {}", x);
}
```

Is program ko run karne ki koshish karein; aapko neeche diya gaya output milna chahiye:

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 1.21 secs
     Running `target/debug/functions`
The value of x is: 5
```

`another_function` ki declaration mein ek parameter hai jiska naam `x` hai. `x` ka type `i32` specify kiya gaya hai. Jab `5` ko `another_function` mein pass kiya jaata hai, toh `println!` macro `5` ko wahaan rakhta hai jahaan format string mein curly brackets ka pair tha.

Function signatures mein, aapko har **parameter ka type declare karna zaroori hai**. Yeh Rust ke design ka ek deliberate decision hai: function definitions mein type annotations ko require karne ka matlab hai ki compiler ko shayad hi kabhi code mein kahin aur unka upyog karne ki zaroorat padti hai, yeh pata lagane ke liye ki aapka kya matlab hai.

Jab aap chahte hain ki ek function mein multiple parameters hon, toh parameter declarations ko commas se alag karein, jaise is tarah:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    another_function(5, 6);
}

fn another_function(x: i32, y: i32) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
}
```

Yeh example ek function create karta hai jisme do parameters hain, dono `i32` types ke hain. Phir function apne dono parameters mein values ko print karta hai. Dhyan dein ki function parameters ka same type hona zaroori nahi hai, bas is example mein woh hain.

Aaiye is code ko run karne ki koshish karte hain. Apne `functions` project ke `src/main.rs` file mein maujood program ko upar diye gaye example se replace karein aur `cargo run` ka use karke isse run karein:

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/functions`
The value of x is: 5
The value of y is: 6
```

Kyunki humne function ko `x` ke liye `5` aur `y` ke liye `6` value ke saath call kiya, toh do strings in values ke saath print hote hain.

-----

### Function Bodies Contain Statements and Expressions (Function Bodies mein Statements aur Expressions hote hain)

Function bodies statements ki series se bante hain, jinke ant mein ek expression ho bhi sakta hai. Abhi tak, humne sirf un functions ko dekha hai jinke ant mein koi expression nahi hota, lekin aapne ek statement ke hisse ke roop mein ek expression dekha hai. Kyunki Rust ek expression-based language hai, yeh ek important distinction hai jise samajhna zaroori hai. Doosri languages mein aisi distinctions nahi hoti, toh aaiye dekhte hain ki statements aur expressions kya hain aur unke differences function bodies ko kaise affect karte hain.

Humne pehle hi statements aur expressions ka upyog kiya hai. **Statements** aisi instructions hain jo kuch action perform karti hain aur koi value return nahi karti. **Expressions** ek resulting value mein evaluate hote hain. Aaiye kuch examples dekhte hain.

`let` keyword se ek variable create karna aur usse ek value assign karna ek statement hai. Listing 3-1 mein, `let y = 6;` ek statement hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let y = 6;
}
```

\<span class="caption"\>Listing 3-1: Ek `main` function declaration jisme ek statement hai\</span\>

Function definitions bhi statements hote hain; poora upar diya gaya example khud mein ek statement hai.

Statements values return nahi karte. Isliye, aap ek `let` statement ko doosre variable mein assign nahi kar sakte, jaisa ki neeche diya gaya code karne ki koshish karta hai; aapko error milega:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let x = (let y = 6);
}
```

Jab aap is program ko run karenge, toh aapko aisi error milegi:

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error: expected expression, found statement (`let`)
 --> src/main.rs:2:14
  |
2 |      let x = (let y = 6);
  |              ^^^^^^^
  |
  = note: variable declaration using `let` is a statement
```

`let y = 6` statement koi value return nahi karta, isliye `x` ke pass bind hone ke liye kuch nahi hai. Yeh doosri languages, jaise C aur Ruby, se alag hai, jahan assignment assignment ki value return karta hai. Un languages mein, aap `x = y = 6` likh sakte hain aur `x` aur `y` dono ki value `6` hogi; Rust mein aisa nahi hota.

Expressions kuch na kuch evaluate karte hain aur Rust mein aap jo baaki code likhenge, uska zyadatar hissa banate hain. Ek simple math operation consider karein, jaise `5 + 6`, jo ek expression hai jo `11` value mein evaluate hota hai. Expressions statements ka hissa ho sakte hain: Listing 3-1 mein, statement `let y = 6;` mein `6` ek expression hai jo `6` value mein evaluate hota hai. Ek function ko call karna ek expression hai. Ek macro ko call karna ek expression hai. Woh block jiska upyog hum naye scopes create karne ke liye karte hain, `{}`, ek expression hai, jaise:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let x = 5;

    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

Yeh expression:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

ek block hai jo, is case mein, `4` mein evaluate hota hai. Woh value `let` statement ke hisse ke roop mein `y` se bind ho jaati hai. Dhyan dein ki `x + 1` line ke ant mein semicolon nahi hai, jo ab tak aapne dekhi gayi zyadatar lines se alag hai. Expressions mein ant mein semicolons shamil nahi hote. Agar aap ek expression ke ant mein semicolon laga dete hain, toh aap usse ek statement bana dete hain, jo phir koi value return nahi karega. Is baat ko dhyan mein rakhein jab aap aage function return values aur expressions explore karenge.

-----

### Functions with Return Values (Return Values wale Functions)

Functions unhe call karne wale code ko values return kar sakte hain. Hum return values ko naam nahi dete, lekin hum unka type ek arrow (`->`) ke baad declare karte hain. Rust mein, function ki return value, function ke body ke block mein final expression ki value ke barabar hoti hai. Aap `return` keyword aur ek value specify karke function se jaldi return kar sakte hain, lekin zyadatar functions last expression ko implicitly return karte hain. Yahaan ek example hai ek function ka jo ek value return karta hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {}", x);
}
```

`five` function mein koi function calls, macros, ya even `let` statements nahi hain—bas number `5` akela hai. Yeh Rust mein ek bilkul valid function hai. Dhyan dein ki function ka return type bhi specify kiya gaya hai, `-> i32` ke roop mein. Is code ko run karne ki koshish karein; output aisa dikhna chahiye:

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/functions`
The value of x is: 5
```

`five` mein `5` function ki return value hai, isliye return type `i32` hai. Aaiye isse aur detail mein dekhte hain. Do important baatein hain: pehla, `let x = five();` line dikhati hai ki hum ek variable ko initialize karne ke liye ek function ki return value ka upyog kar rahe hain. Kyunki `five` function `5` return karta hai, woh line neeche diye gaye ke samaan hai:

```rust
let x = 5;
```

Doosra, `five` function mein koi parameters nahi hain aur yeh return value ka type define karta hai, lekin function ka body ek akela `5` hai jisme koi semicolon nahi hai kyunki yeh ek expression hai jise hum return karna chahte hain.

Aaiye ek aur example dekhte hain:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

Is code ko run karne par `The value of x is: 6` print hoga. Lekin agar hum `x + 1` wali line ke ant mein ek semicolon laga dete hain, usse ek expression se statement mein badal dete hain, toh humein ek error milegi.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1;
}
```

Is code ko run karne par ek error aata hai, jaisa ki neeche diya gaya hai:

```text
error[E0308]: mismatched types
 --> src/main.rs:7:28
  |
7 |   fn plus_one(x: i32) -> i32 {
  |  ____________________________^
8 | |     x + 1;
  | |         - help: consider removing this semicolon
9 | | }
  | |_^ expected i32, found ()
  |
  = note: expected type `i32`
              found type `()`
```

Main error message, “mismatched types,” is code ke core issue ko reveal karta hai. `plus_one` function ki definition kehti hai ki woh ek `i32` return karega, lekin statements ek value mein evaluate nahi hote, jisse `()` (empty tuple) dwara vyakt kiya jaata hai. Isliye, kuch bhi return nahi hota, jo function definition ke vipreet hai aur isse error aati hai. Is output mein, Rust is issue ko shayad theek karne mein help karne ke liye ek message provide karta hai: yeh semicolon ko hatane ka sujhav deta hai, jisse error theek ho jaati.
