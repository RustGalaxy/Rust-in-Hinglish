## Pattern Syntax

Is section mein, hum patterns mein valid sabhi syntax ko ikattha karte hain aur charcha karte hain ki aap har ek ka istemal kyon aur kab karna chahenge.

-----

### Literals ko Match Karna

Jaisa ki aapne Chapter 6 mein dekha, aap patterns ko seedhe literals ke khilaaf match kar sakte hain. Nimnalikhit code kuch udaharan deta hai:

```rust
fn main() {
    let x = 1;

    match x {
        1 => println!("one"),
        2 => println!("two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
}
```

Yah code `one` print karega kyunki `x` mein value `1` hai. Yah syntax tab upyogi hota hai jab aap chahte hain ki aapka code ek vishesh concrete value milne par koi action le.

-----

### Named Variables ko Match Karna

Named variables irrefutable patterns hain jo kisi bhi value se match karte hain, aur humne is kitaab mein unka kai baar istemal kiya hai. Halaanki, jab aap `match` expressions mein named variables ka istemal karte hain to ek jatilta (complication) aati hai. Kyunki `match` ek naya scope shuru karta hai, iske andar pattern ke hisse ke roop mein declare kiye gaye variables bahar ke usi naam ke variables ko **shadow** karenge.

Listing 19-11 mein, hum `x` naamak ek variable `Some(5)` value ke saath aur `y` naamak ek variable `10` value ke saath declare karte hain. Phir hum `x` value par ek `match` expression banate hain. Match arms mein patterns aur ant mein `println!` ko dekhein, aur is code ko chalane se pehle yah anuman lagane ki koshish karein ki code kya print karega.

```rust
// Filename: src/main.rs
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {y}"),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {y}", x);
}
```

*Listing 19-11: Ek `match` expression jiska ek arm ek naya variable introduce karta hai jo ek maujooda variable `y` ko shadow karta hai*

Chaliye dekhen ki `match` expression chalne par kya hota hai:

1.  Pehla arm (`Some(50)`) match nahi hota.
2.  Doosra arm (`Some(y)`) ek naya variable `y` introduce karta hai. Yah `match` expression ke andar ek naya scope hai, isliye yah naya `y` bahar wale `y` (jiski value 10 hai) se alag hai. Yah naya `y` `Some` ke andar ki kisi bhi value se match karega. `x` mein value `Some(5)` hai, isliye yah naya `y` `5` se bind ho jaata hai. Isliye, expression `Matched, y = 5` print karta hai.
3.  `match` expression ke khatm hone par, inner `y` ka scope bhi khatm ho jaata hai.
4.  Aakhri `println!` `at the end: x = Some(5), y = 10` produce karta hai.

-----

### Multiple Patterns

`match` expressions mein, aap `|` syntax ka istemal karke multiple patterns ko match kar sakte hain, jo pattern *or* operator hai.

```rust
fn main() {
    let x = 1;

    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
}
```

Yah code `one or two` print karega.

-----

### `..=` ke Saath Values ki Ranges ko Match Karna

`..=` syntax humein values ki ek inclusive range se match karne ki anumati deta hai.

```rust
fn main() {
    let x = 5;

    match x {
        1..=5 => println!("one through five"),
        _ => println!("something else"),
    }
}
```

Agar `x` `1`, `2`, `3`, `4`, ya `5` hai, to pehla arm match karega. Ranges keval numeric ya `char` values ke saath hi anumati prapt hain.

-----

### Values ko Todne ke Liye Destructuring

Hum structs, enums, aur tuples ko destructure karne ke liye bhi patterns ka istemal kar sakte hain.

#### Destructuring Structs

```rust
// Filename: src/main.rs
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);

    // Shorthand
    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

*Listing 19-12 & 19-13: Ek struct ke fields ko alag-alag variables mein destructure karna*

Hum literals ke saath bhi destructure kar sakte hain. Yah code points ko x-axis, y-axis, ya kisi par bhi nahi, mein alag karta hai.

```rust
// Filename: src/main.rs
# struct Point {
#     x: i32,
#     y: i32,
# }
fn main() {
    let p = Point { x: 0, y: 7 };
    match p {
        Point { x, y: 0 } => println!("On the x axis at {x}"),
        Point { x: 0, y } => println!("On the y axis at {y}"),
        Point { x, y } => {
            println!("On neither axis: ({x}, {y})");
        }
    }
}
```

*Listing 19-14: Ek hi pattern mein literal values ko destructure aur match karna*

#### Destructuring Enums

Hum enums ko bhi destructure kar sakte hain, jaisa ki humne Chapter 6 mein dekha tha. Pattern enum ke andar store kiye gaye data ke anuroop hona chahiye.

```rust
// Filename: src/main.rs
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => { /* ... */ }
        Message::Move { x, y } => { /* ... */ }
        Message::Write(text) => { /* ... */ }
        Message::ChangeColor(r, g, b) => { /* ... */ }
    }
}
```

*Listing 19-15: Alag-alag prakaar ke values hold karne wale enum variants ko destructure karna*

-----

### Ek Pattern mein Values ko Ignore Karna

Kabhi-kabhi ek pattern mein values ko ignore karna upyogi hota hai. Iske kuch tareeke hain: `_`, `_` ko doosre pattern ke andar istemal karna, `_` se shuru hone wala naam, ya `..` ka istemal karna.

#### Poori Value ko `_` ke Saath Ignore Karna

`_` ek wildcard pattern hai jo kisi bhi value se match karega lekin use bind nahi karega.

#### Nested `_` ke Saath Value ke Hisse ko Ignore Karna

Hum ek value ke sirf ek hisse ko ignore karne ke liye `_` ka istemal doosre pattern ke andar kar sakte hain.

```rust
// Filename: src/main.rs
fn main() {
    let mut setting_value = Some(5);
    let new_setting_value = Some(10);

    match (setting_value, new_setting_value) {
        (Some(_), Some(_)) => {
            println!("Can't overwrite an existing customized value");
        }
        _ => {
            setting_value = new_setting_value;
        }
    }
    println!("setting is {:?}", setting_value);
}
```

*Listing 19-18: `Some` variants match karne wale patterns ke andar `_` ka istemal karna jab humein `Some` ke andar ki value ka istemal karne ki zaroorat nahi hoti*

#### `_` se Naam Shuru Karke Ek Anused Variable ko Ignore Karna

Agar aap ek variable banate hain lekin uska istemal nahi karte hain, to Rust aam taur par ek warning dega. Variable ka naam underscore (`_`) se shuru karke aap Rust ko unused variable ke baare mein warning na dene ke liye keh sakte hain.

`_x` abhi bhi value ko variable se bind karta hai (aur is prakaar ownership le sakta hai), jabki `_` bilkul bhi bind nahi karta.

#### `..` ke Saath Value ke Baaki Hisson ko Ignore Karna

Kai hisson wali values ke saath, hum vishisht hisson ka istemal karne aur baaki ko ignore karne ke liye `..` syntax ka istemal kar sakte hain.

```rust
// Filename: src/main.rs
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

fn main() {
    let origin = Point { x: 0, y: 0, z: 0 };

    match origin {
        Point { x, .. } => println!("x is {}", x),
    }
}
```

*Listing 19-23: `..` ka istemal karke `Point` ke sabhi fields ko `x` ke alawa ignore karna*

`..` ka istemal spasht (unambiguous) hona chahiye. Agar yah aspast hai ki kaun si values match karne ke liye hain aur kaun si ignore ki jaani chahiye, to Rust error dega.

-----

### Match Guards ke Saath Extra Conditionals

Ek **match guard** ek atirikt `if` condition hai, jo `match` arm mein pattern ke baad specify kiya jaata hai, jo us arm ko chune jaane ke liye bhi match hona chahiye.

```rust
// Filename: src/main.rs
fn main() {
    let num = Some(4);

    match num {
        Some(x) if x % 2 == 0 => println!("The number {x} is even"),
        Some(x) => println!("The number {x} is odd"),
        None => (),
    }
}
```

*Listing 19-26: Ek pattern mein match guard jodna*

Yahan, pehla arm tabhi match karega jab `x` ek sam sankhya (even number) ho. Match guard mein `if` condition `|` operator dwara combine kiye gaye sabhi patterns par laagu hoti hai.

-----

### `@` Bindings

**At operator (`@`)** humein ek variable banane deta hai jo ek value hold karta hai jab hum us value ko ek pattern match ke liye test kar rahe hote hain.

```rust
// Filename: src/main.rs
enum Message {
    Hello { id: i32 },
}

fn main() {
    let msg = Message::Hello { id: 5 };

    match msg {
        Message::Hello {
            id: id_variable @ 3..=7,
        } => println!("Found an id in range: {}", id_variable),
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => println!("Found some other id: {}", id),
    }
}
```

*Listing 19-29: Ek pattern mein value se bind karne ke liye `@` ka istemal karna jabki use test bhi kiya ja raha ho*

`id_variable @ 3..=7` specify karke, hum range `3..=7` se match hone wali kisi bhi value ko `id_variable` naamak variable mein capture kar rahe hain, jabki yah bhi test kar rahe hain ki value range pattern se match hui hai.

-----

## Saaraansh (Summary)

Rust ke patterns alag-alag prakaar ke data ke beech antar karne mein bahut upyogi hain. `match` expressions mein istemal kiye jaane par, Rust yah sunishchit karta hai ki aapke patterns har sambhav value ko cover karein, ya aapka program compile nahi hoga. Hum apni zarooraton ke anuroop saral ya jatil patterns bana sakte hain.