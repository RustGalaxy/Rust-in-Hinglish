## An Example Program Using Structs

Structs kab use karna chahiye yeh samajhne ke liye, chalo ek program likhte hain jo rectangle ka area calculate kare. Pehle hum single variables se start karenge aur phir step-by-step program ko refactor karke structs ka use karenge.

Sabse pehle ek naya binary project Cargo ke sath banate hain jiska naam rakhenge *rectangles*. Yeh program rectangle ki width aur height pixels me lega aur uska area calculate karega. Listing 5-8 me iska ek simple version dikhaya gaya hai:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

<span class="caption">Listing 5-8: Separate width aur height variables use karke rectangle ka area calculate karna</span>

Ab `cargo run` karke program run karo:

```text
The area of the rectangle is 1500 square pixels.
```

Yeh code sahi kaam karta hai, lekin `area` function ke signature me issue hai:

```rust,ignore
fn area(width: u32, height: u32) -> u32 {
```

`area` function ek rectangle ka area calculate karna chahiye, par yeh do parameters leta hai jo related hain, lekin program me wo relation kahi express nahi hai. Isse better hoga agar hum width aur height ko ek saath group karein. Chapter 3 me tuples ke baare me padha tha, toh chalo pehle tuples se refactor karte hain.

### Refactoring with Tuples

Listing 5-9 me tuples ka use dikhaya gaya hai:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

<span class="caption">Listing 5-9: Tuple ke sath rectangle ki width aur height specify karna</span>

Yeh version thoda better hai kyunki ab hum ek hi argument pass kar rahe hain. Lekin readability kam ho gayi hai kyunki tuples ke elements ke naam nahi hote. Hume yaad rakhna padega ki `0` index width hai aur `1` index height, jo confusion aur errors cause kar sakta hai.

### Refactoring with Structs: Adding More Meaning

Structs ka use karke hum data ko label kar sakte hain. Tuple ko struct me convert karke hum ek meaningful type bana sakte hain, jaise Listing 5-10 me:

<span class="filename">Filename: src/main.rs</span>

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

<span class="caption">Listing 5-10: `Rectangle` struct define karna</span>

Ab humne ek struct define kiya hai `Rectangle` ke naam se, jisme `width` aur `height` fields hain. `area` function ab ek `Rectangle` ka reference leta hai, taaki ownership `main` ke paas hi rahe.

Yeh version zyada clear hai kyunki humne values ko descriptive names diye hain aur tuple indices ka use nahi karna pad raha.

### Adding Useful Functionality with Derived Traits

Agar hum debugging ke time `Rectangle` instance ko directly print karna chahein toh error milega, jaise Listing 5-11 me:

```rust,ignore
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {}", rect1);
}
```

Error:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Display` is not satisfied
```

Structs ke liye `Display` trait by default implement nahi hota. Lekin compiler suggest karta hai ki `:?` ka use karein `Debug` format ke liye. Agar aisa karte hain toh phir bhi error aayega jab tak hum `#[derive(Debug)]` annotation nahi lagate. Jaise Listing 5-12 me:

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {:?}", rect1);
}
```

Output:

```text
rect1 is Rectangle { width: 30, height: 50 }
```

Agar `{:#?}` use karein toh output aur readable ho jata hai:

```text
rect1 is Rectangle {
    width: 30,
    height: 50
}
```

Rust ke paas aur bhi derived traits hain jo custom types ke sath useful functionality add karte hain. In traits ke baare me Appendix C me details hain.

Abhi hamara `area` function specific hai sirf rectangles ke liye. Isse aur closely tie karne ke liye hum `area` ko ek method bana sakte hain `Rectangle` struct ke andar.
