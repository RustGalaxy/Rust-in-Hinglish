## Cargo Workspaces

Chapter 12 mein, humne ek package banaya tha jismein ek binary crate aur ek library crate shaamil tha. Jaise-jaise aapka project develop hota hai, ho sakta hai ki library crate bada hota jaye aur aap apne package ko aur bhi kai library crates mein todna chahein. Aisi situation mein, Cargo ek feature offer karta hai jise **workspaces** kehte hain, jo ek saath develop kiye ja rahe multiple related packages ko manage karne mein madad kar sakta hai.

-----

### Ek Workspace Banana

Ek **workspace** packages ka ek set hota hai jo ek hi *Cargo.lock* aur output directory share karte hain. Chaliye ek workspace ka use karke ek project banate hain—hum simple code use karenge taaki hum workspace ke structure par focus kar sakein. Ek workspace ko structure karne ke kai tareeke hain; hum ek aam tareeka dikhayenge. Hamare paas ek workspace hoga jismein ek binary aur do libraries hongi. Binary, jo main functionality dega, woh do libraries par depend karega. Ek library `add_one` function degi, aur doosri `add_two` function. Yeh teeno crates ek hi workspace ka hissa honge. Hum workspace ke liye ek nayi directory banakar shuruaat karenge:

```text
$ mkdir add
$ cd add
```

Iske baad, *add* directory mein, hum ek *Cargo.toml* file banayenge jo poore workspace ko configure karegi. Is file mein `[package]` section ya woh metadata nahi hoga jo humne doosri *Cargo.toml* files mein dekha hai. Iske bajaye, yeh `[workspace]` section se shuru hogi jo humein apne binary crate ka path specify karke workspace mein members add karne ki anumati dega; is maamle mein, woh path *adder* hai:

\<span class="filename"\>Filename: Cargo.toml\</span\>

```toml
[workspace]

members = [
    "adder",
]
```

Ab, hum *add* directory ke andar `cargo new` chalaakar `adder` binary crate banayenge:

```text
$ cargo new --bin adder
     Created binary (application) `adder` project
```

Is point par, hum `cargo build` chalaakar workspace ko build kar sakte hain. Aapki *add* directory mein files is tarah dikhni chahiye:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Workspace mein top level par ek hi **target** directory hoti hai jahan compiled files rakhi jaati hain; `adder` crate ki apni koi *target* directory nahi hoti. Agar hum *adder* directory ke andar se bhi `cargo build` chalate, tab bhi compiled files *add/adder/target* ke bajaye *add/target* mein hi aati. Cargo workspace mein *target* directory ko is tarah structure karta hai kyunki workspace ke crates ek doosre par depend karne ke liye bane hote hain. Agar har crate ki apni *target* directory hoti, to har crate ko workspace ke doosre crates ko recompile karna padta. Ek hi *target* directory share karke, crates bematlab ke rebuilding se bach jaate hain.

-----

### Workspace Mein Doosra Crate Banana

Ab, chaliye workspace mein ek aur member crate banate hain aur use `add-one` naam dete hain. Top-level *Cargo.toml* ko badlein aur `members` list mein *add-one* ka path specify karein:

\<span class="filename"\>Filename: Cargo.toml\</span\>

```toml
[workspace]

members = [
    "adder",
    "add-one",
]
```

Phir `add-one` naam ka ek naya library crate generate karein:

```text
$ cargo new add-one
     Created library `add-one` project
```

Aapki *add* directory mein ab yeh directories aur files honi chahiye:

```text
├── Cargo.lock
├── Cargo.toml
├── add-one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

*add-one/src/lib.rs* file mein, chaliye ek `add_one` function add karte hain:

\<span class="filename"\>Filename: add-one/src/lib.rs\</span\>

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

Ab jab hamare workspace mein ek library crate hai, hum binary crate `adder` ko library crate `add-one` par depend karva sakte hain. Pehle, humein *adder/Cargo.toml* mein `add-one` ke liye ek path dependency add karni hogi.

\<span class="filename"\>Filename: adder/Cargo.toml\</span\>

```toml
[dependencies]

add-one = { path = "../add-one" }
```

Cargo yeh maan kar nahi chalta ki ek workspace ke andar ke crates ek doosre par depend karenge, isliye humein in crates ke beech dependency relationships ko **saaf-saaf batana** padta hai.

Ab, `adder` crate mein `add-one` crate se `add_one` function ka use karte hain. *adder/src/main.rs* file kholen aur file ke top par ek `extern crate` line add karein taaki `add-one` library crate scope mein aa jaye. Phir `main` function ko `add_one` function call karne ke liye badlein:

\<span class="filename"\>Filename: adder/src/main.rs\</span\>

```rust,ignore
extern crate add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {} plus one is {}!", num, add_one::add_one(num));
}
```

\<span class="caption"\>Listing 14-7: `adder` crate se `add-one` library crate ka istemaal\</span\>

Chaliye top-level *add* directory mein `cargo build` chalaakar workspace ko build karte hain\!

```text
$ cargo build
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68 secs
```

*add* directory se binary crate ko run karne ke liye, humein `-p` argument aur package ka naam `cargo run` ke saath dekar yeh batana hoga ki hum workspace mein kaun sa package use karna chahte hain:

```text
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Yeh *adder/src/main.rs* mein code ko run karta hai, jo `add-one` crate par depend karta hai.

-----

#### Workspace mein External Crate par Depend Karna

Dhyan dein ki workspace mein har crate ki directory mein *Cargo.lock* hone ke bajaye, top level par sirf ek hi *Cargo.lock* file hai. Yeh sunishchit karta hai ki sabhi crates sabhi dependencies ke **same version** ka use kar rahe hain. Agar hum *adder/Cargo.toml* aur *add-one/Cargo.toml* dono mein `rand` crate add karte hain, to Cargo dono ko `rand` ke ek hi version par resolve karega aur use ek hi *Cargo.lock* mein record karega.

Yeh sunishchit karna ki workspace ke sabhi crates same dependencies ka use karein, iska matlab hai ki workspace ke crates hamesha ek doosre ke saath compatible rahenge.

Agar hum sirf *add-one/Cargo.toml* mein `rand` crate add karte hain:
\<span class="filename"\>Filename: add-one/Cargo.toml\</span\>

```toml
[dependencies]
rand = "0.8.5" # Using a modern version
```

Toh `cargo build` chalaane par `rand` crate compile ho jaayega. Lekin, bhale hi `rand` workspace mein kahin use ho raha ho, hum use doosre crates mein tab tak use nahi kar sakte jab tak hum `rand` ko unki *Cargo.toml* files mein bhi add na karein. `adder` crate mein `rand` ko use karne ki koshish karne par error aayega.

Ise theek karne ke liye, `adder` crate ke *Cargo.toml* ko bhi edit karein aur batayein ki `rand` us crate ke liye bhi ek dependency hai. Aisa karne se `rand` crate dobara download nahi hoga. Cargo ne yeh sunishchit kiya hai ki workspace mein `rand` crate ka upyog karne wala har crate same version ka upyog karega. Isse space bachta hai aur crates ke beech compatibility bani rehti hai.

-----

#### Workspace Mein Test Add Karna

Chaliye `add_one` crate ke andar `add_one::add_one` function ka ek test add karte hain:

\<span class="filename"\>Filename: add-one/src/lib.rs\</span\>

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```

Ab top-level *add* directory mein `cargo test` chalaayein:

```text
$ cargo test
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
   ...
     Running target/debug/deps/add_one-f0253159197f7841
running 1 test
test tests::it_works ... ok
...
     Running target/debug/deps/adder-f88af9d2cc175a5e
running 0 tests
...
```

Output ka pehla section dikhata hai ki `add-one` crate mein `it_works` test pass ho gaya. Agla section dikhata hai ki `adder` crate mein zero tests paaye gaye. Is tarah ke workspace mein `cargo test` chalaane se **workspace ke sabhi crates ke tests run hote hain**.

Hum top-level directory se `-p` flag ka use karke kisi ek particular crate ke liye bhi tests run kar sakte hain:

```text
$ cargo test -p add-one
...
running 1 test
test tests::it_works ... ok
...
```

Yeh output dikhata hai ki `cargo test` ne sirf `add-one` crate ke tests chalaye aur `adder` crate ke tests nahi chalaye.

Agar aap workspace ke crates ko *crates.io* par publish karte hain, to workspace ke har crate ko **alag-alag publish** karna hoga. `cargo publish` command mein `--all` ya `-p` jaisa koi flag nahi hota hai, isliye aapko har crate ki directory mein jaakar us par `cargo publish` chalaana hoga.

Practice ke liye, is workspace mein `add-one` crate ki tarah hi ek `add-two` crate add karein\!

Jaise-jaise aapka project badhta hai, workspace ka use karne par vichaar karein: ek bade code ke bajaye chhote, individual components ko samajhna aasan hota hai. Saath hi, crates ko ek workspace mein rakhne se unke beech coordination aasan ho jaata hai agar unmein aksar ek hi samay mein badlav kiye jaate hain.
