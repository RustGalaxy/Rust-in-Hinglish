# Programming a Guessing Game  

Chalo Rust ko ek hands-on project ke through explore karte hain!  
Is chapter me tumhe kuch common Rust concepts sikhnay ko milenge, real program likh kar.  
Tum seekhoge `let`, `match`, methods, associated functions, external crates aur aur bhi.  
Agle chapters me in sabko detail me cover karenge, lekin yaha basics practice karenge.  

Hum ek classic beginner problem implement karenge: **Guessing Game**.  

Game aise kaam karega:  
- Program ek random integer generate karega **1 se 100 ke beech**.  
- Player ko ek guess input karna hoga.  
- Program batayega guess **bohot chhota** hai ya **bohot bada**.  
- Agar guess sahi hai → program ek congratulatory message print karega aur exit ho jaayega.  

---

## Setting Up a New Project  

Naya project banane ke liye, *projects* directory (jo tumne Chapter 1 me banayi thi) me jao aur Cargo use karke ek naya project banao:  

```bash
$ cargo new guessing_game --bin
$ cd guessing_game
```

- Pehla command `cargo new` project ka naam (`guessing_game`) leta hai.  
- `--bin` flag Cargo ko bolta hai ki ek binary project banao (jaise Chapter 1 me tha).  
- Dusra command project directory me move karta hai.  

Ab *Cargo.toml* file ko dekho:  

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
```

Agar Cargo ne galat author info uthaya hai, toh file edit karke sahi kar lo.  

Jaise tumne Chapter 1 me dekha tha, `cargo new` ek “Hello, world!” program generate karta hai.  
*src/main.rs* file ko dekho:  

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Ab is program ko compile aur run karte hain ek hi step me, using `cargo run`:  

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50 secs
     Running `target/debug/guessing_game`
Hello, world!
```

`cargo run` kaafi useful hai jab tum project pe quickly iterate karna chahte ho, jaise hum is game me karenge.  

Ab *src/main.rs* reopen karo. Saara code isi file me likhna hai.  

---

## Processing a Guess  

Guessing game ka pehla part user se input lena aur usko process karna hoga.  
Sabse pehle, user ko ek guess input karne do. Ye code *src/main.rs* me likho:  

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

<span class="caption">Listing 2-1: User se guess lena aur usko print karna</span>  

---

### Line by Line Explanation  

Sabse pehle, input/output (I/O) library import karni padti hai:  

```rust,ignore
use std::io;
```

Rust automatically *prelude* me kuch basic types import karta hai, lekin har cheez nahi.  
Agar tumhe koi type use karna hai jo prelude me nahi hai, toh tumhe `use` karke usko scope me lana padta hai.  

---

### Entry Point: `main` function  

```rust,ignore
fn main() {
```

- `fn` → function declare karta hai.  
- `()` → no parameters.  
- `{}` → function body.  

Aur input prompt print karne ke liye:  

```rust,ignore
println!("Guess the number!");
println!("Please input your guess.");
```

---

### Storing Values with Variables  

User input ko store karne ke liye:  

```rust,ignore
let mut guess = String::new();
```

Yaha:  
- `let` → ek variable banata hai.  
- Default Rust variables **immutable** hote hain. Agar mutable chahiye toh `mut` lagana padta hai.  

Example:  

```rust,ignore
let foo = 5;      // immutable
let mut bar = 5;  // mutable
```

`String::new()` ek naya empty `String` return karta hai.  

---

### Reading Input  

```rust,ignore
io::stdin().read_line(&mut guess)
    .expect("Failed to read line");
```

- `io::stdin()` → terminal se standard input handle deta hai.  
- `.read_line(&mut guess)` → user input le kar usko `guess` me daalta hai.  
- `&mut` → mutable reference, taaki method `guess` ko modify kar sake.  

---

### Handling Errors with `Result`  

`read_line` ek `Result` return karta hai → ya toh `Ok` ya `Err`.  

Agar error aata hai, toh:  

```rust,ignore
.expect("Failed to read line");
```

program crash karega aur yeh message dikhayega.  
Agar `Ok` aata hai, toh user input successfully read ho jaata hai.  

Rust agar tum `expect` na likho, toh warning deta hai:  

```bash
warning: unused `std::result::Result` which must be used
```

Iska matlab hai ki tumne error handle nahi kiya.  
Abhi ke liye crash karna theek hai, lekin Chapter 9 me tum proper error recovery seekhoge.  

### `println!` ke saath Values Print karna (Placeholders)

Closing curly brackets ke alawa, ek aur line hai jiske baare me baat karni hai
jo abhi tak code me add hui hai:

```rust,ignore
println!("You guessed: {}", guess);
```

Ye line user ka input (jo humne string me save kiya) print karti hai. Curly
brackets `{}` ek placeholder hai: socho `{}` chhoti crab ke claws jaise hain
jo value ko hold karte hain. Tum ek se zyada values bhi print kar sakte ho curly
brackets ke through: pehli curly brackets pehli value ko hold karegi, dusri dusri
value ko, aur aise hi. Ek example:

```rust
let x = 5;
let y = 10;

println!("x = {} and y = {}", x, y);
```

Ye code print karega: `x = 5 and y = 10`.

---

### First Part ka Testing

Ab chalo guessing game ka first part test karein. Run karo `cargo run` ke saath:

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

Abhi tak humne pehla part complete kar liya hai: input keyboard se le rahe hain
aur usko print kar rahe hain.

---

## Secret Number Generate karna

Ab hume ek secret number generate karna hai jo user guess karega. Har baar
alag number aana chahiye taki game interesting bane. Hum 1 se 100 ke beech
random number lenge. Rust ki standard library me abhi random number ka feature
nahi hai. Lekin Rust team ek crate deti hai jiska naam hai [`rand`][randcrate].

[randcrate]: https://crates.io/crates/rand

---

### Extra Functionality ke liye Crate use karna

Yaad rakho, crate ek Rust code ka package hota hai. Jo project hum bana rahe
hain wo ek *binary crate* hai (jo executable hota hai). Lekin `rand` ek
*library crate* hai (jo dusre programs me use hone ke liye code deta hai).

Cargo crates ko manage karne me best hai. `rand` use karne ke liye hume
*Cargo.toml* file me `rand` dependency add karni hogi. File khol ke neeche ye
line add karo:

<span class="filename">Filename: Cargo.toml</span>

```toml
[dependencies]

rand = "0.3.14"
```

`Cargo.toml` me `[dependencies]` section me hum external crates aur unke version
likhte hain jo project ko chahiye. Yahaan humne `rand` crate `0.3.14` version ke
saath specify kiya hai. Cargo Semantic Versioning (SemVer) follow karta hai,
iska matlab ye shorthand hai `^0.3.14`. Matlab: koi bhi version jo public API me
compatible ho `0.3.14` ke saath.

[semver]: http://semver.org

---

### Project Build karna

Ab bina code change kiye project build karo:

```text
$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rand v0.3.14
 Downloading libc v0.2.14
   Compiling libc v0.2.14
   Compiling rand v0.3.14
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
```

<span class="caption">Listing 2-2: `cargo build` ka output jab `rand` crate add
kiya gaya dependency ke roop me</span>

Yahaan tumhe alag version numbers bhi dikh sakte hain, lekin code compatible
rahega (SemVer ki wajah se).

Cargo registry (*Crates.io* ka data) se sab dependencies download karta hai.
Humne `rand` add kiya tha, lekin `rand` khud `libc` pe depend karta hai, isliye
Cargo ne `libc` bhi download kiya.

Agar dubara `cargo build` run karoge bina kuch badle, sirf `Finished` line aayegi.
Cargo ko pata hai dependencies aur code dono me koi change nahi hua.

Agar tum *src/main.rs* me chhota sa change karoge aur dubara build karoge:

```text
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
```

Sirf tumhare code ka rebuild hoga, dependencies reuse hongi.

---

#### *Cargo.lock* se Reproducible Builds

Cargo ensure karta hai ki har build same result de. Jab tumne pehli baar
`cargo build` run kiya tha, usne *Cargo.lock* file banayi. Usme versions lock ho
gaye jo use hue the. Jab tak tum explicitly update nahi karte, wahi versions
use honge.

---

#### Crate Update karna

Agar tum update karna chahte ho, `cargo update` command use karo. Ye
*Cargo.lock* ignore karke naye versions fetch karega.

Example:

```text
$ cargo update
    Updating registry `https://github.com/rust-lang/crates.io-index`
    Updating rand v0.3.14 -> v0.3.15
```

Ab *Cargo.lock* me version update ho jayega.

Agar `0.4.0` version chahiye, to `Cargo.toml` me update karo:

```toml
[dependencies]

rand = "0.4.0"
```

Agli baar build pe naya version use hoga.

---

### Random Number Generate karna

Ab hum `rand` crate ko use karenge *src/main.rs* me (Listing 2-3):

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate rand;

use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

<span class="caption">Listing 2-3: Random number generate karna</span>

- `extern crate rand;` likhne se Rust ko batate hain ki hum external crate use
kar rahe hain.  
- `use rand::Rng;` likhne se `Rng` trait scope me aata hai jo random number
methods deta hai.  
- `rand::thread_rng()` ek random number generator deta hai jo OS ke seed se
initialize hota hai.  
- `.gen_range(1, 101)` call karne se 1 aur 100 ke beech number milega (upper
bound exclusive hota hai).

Note: Har crate ki documentation me methods kaise use karne hain diye hote hain.
Cargo ke through tum `cargo doc --open` run karke local documentation dekh
sakte ho.

Ab run karke test karo:

```text
$ cargo run
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4
```

Har run pe alag number aayega 1 aur 100 ke beech. 🎉

## Guess ko Secret Number ke saath Compare karna

Ab jab humare paas user input aur ek random number hai, hum in dono ko compare kar sakte hain. Yeh step Listing 2-4 me dikhaya gaya hai. Dhyan rahe ki yeh code abhi compile nahi hoga, jaise ki hum explain karenge.

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {

    // ---snip---

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

Sabse pehla naya cheez hai ek aur `use` statement jo `std::cmp::Ordering` type ko scope me laata hai standard library se. `Result` ki tarah, `Ordering` bhi ek enum hai, jiske variants hain `Less`, `Greater`, aur `Equal`. Yeh teen hi outcomes ho sakte hain jab aap do values ko compare karte hain.

Phir hum bottom me 5 nayi lines add karte hain jo `Ordering` type use karti hain. `cmp` method do values ko compare karta hai aur isse kisi bhi cheez pe call kiya ja sakta hai jo compare ho sakti hai. Yeh reference leta hai jis se aap compare karna chahte hain: yahan hum `guess` ko `secret_number` ke saath compare kar rahe hain. Phir yeh `Ordering` enum ka ek variant return karta hai jo humne `use` statement ke through scope me laya. Hum `match` expression use karte hain decide karne ke liye ki next kya karna hai based on kaunsa `Ordering` variant `cmp` call se return hua.

Ek `match` expression *arms* se bana hota hai. Ek arm me hota hai ek *pattern* aur code jo run hoga agar value match kar jaaye arm ke pattern se. Rust value ko match me leta hai aur har arm ka pattern check karta hai. `match` aur patterns Rust me powerful features hain jo allow karte hain ki aap apne code ke multiple situations ko handle kar sake.

Chalo ek example dekhte hain. Agar user ne guess kiya 50 aur secret number 38 hai, to `cmp` method `Ordering::Greater` return karega kyunki 50 > 38. `match` expression `Ordering::Greater` ko check karta hai har arm ke pattern ke against. Pehla arm `Ordering::Less` se match nahi hota, isliye code ignore hota hai aur next arm me jata hai. Dusra arm `Ordering::Greater` se match ho jaata hai! Is arm ka code execute hota hai aur screen pe print karta hai "Too big!".

Ab hum `String` input ko number me convert karenge taaki comparison possible ho. Yeh do lines add kar ke kiya jaa sakta hai:

```rust,ignore
let guess: u32 = guess.trim().parse()
    .expect("Please type a number!");
```

Yahan humne `guess` variable shadow kiya taaki original `String` ko `u32` me convert kar sake.

Loop aur multiple guesses ke liye code:

```rust,ignore
loop {
    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    let guess: u32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => {
            println!("You win!");
            break;
        }
    }
}
```

Final complete code:

```rust,ignore
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

## Summary

Ab tak aap successfully guessing game bana chuke hain. 🎉

Is project se aapne Rust ke naye concepts seekhe: `let`, `match`, methods, external crates, aur aur bhi. Next chapters me inko detail me cover kiya jayega:

* Chapter 3: Variables, data types, functions
* Chapter 4: Ownership
* Chapter 5: Structs aur methods
* Chapter 6: Enums
