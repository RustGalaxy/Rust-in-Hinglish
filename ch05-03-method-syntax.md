## Method Syntax

*Methods* functions jaise hi hote hain: inhe `fn` keyword aur unke naam ke sath declare kiya jata hai, inme parameters aur return value ho sakti hai, aur inke andar code hota hai jo tab run hota hai jab unhe kahin aur se call kiya jata hai. Lekin, methods functions se is sense me alag hote hain ki unhe ek struct (ya enum ya trait object, jinke baare me hum Chapters 6 aur 17 me padhenge) ke context me define kiya jata hai, aur unka pehla parameter hamesha `self` hota hai, jo us struct ke instance ko represent karta hai jiske upar method call ho raha hai.

### Defining Methods

Chaliye ab `area` function ko change karte hain jo ek `Rectangle` instance ko parameter ke roop me leta tha, aur usse ek `area` method banate hain jo `Rectangle` struct ke andar define hoga (Listing 5-13 me dikhaya gaya hai).

<span class="filename">Filename: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

<span class="caption">Listing 5-13: `Rectangle` struct me ek `area` method define karna</span>

`Rectangle` ke context me function define karne ke liye hum ek `impl` block start karte hain. Fir hum `area` function ko `impl` ke curly brackets ke andar le aate hain aur uska pehla (aur is case me sirf ek) parameter `self` kar dete hain. Ab `main` function me jaha humne `area` function ko call kiya tha aur `rect1` pass kiya tha, waha ab hum *method syntax* use kar sakte hain: yani instance ke baad dot (`.`), method ka naam, aur parentheses.

Method signature me hum `&self` use karte hain instead of `rectangle: &Rectangle`, kyunki Rust already samajh jata hai ki `self` ka type `Rectangle` hai. Dhyaan rahe ki `self` ke aage hume `&` lagana padta hai, bilkul waise hi jaise hum `&Rectangle` likhte the. Methods `self` ka ownership le sakte hain, usse immutable borrow kar sakte hain (`&self`), ya mutable borrow kar sakte hain (`&mut self`), bilkul normal parameters ki tarah.

Humne yaha `&self` isliye use kiya hai kyunki hum ownership nahi lena chahte, bas struct ke data ko read karna chahte hain. Agar hum instance ko change karna chahte, to hum `&mut self` use karte. Aur agar hume method ke andar se instance ko consume karna hota (rare case), tab hum `self` parameter directly dete.

Methods ka main benefit ye hai ki aapko type repeat nahi karna padta aur code organized ho jata hai. Sare operations jo `Rectangle` ke sath kiye ja sakte hain, ek hi `impl` block me grouped hote hain.

> ### Where’s the `->` Operator?
>
> C aur C++ me methods call karne ke liye do operators hote hain: `.` jab aap object directly use kar rahe ho aur `->` jab aap object ke pointer ko use kar rahe ho. Rust me `->` operator nahi hota. Rust me ek feature hota hai *automatic referencing and dereferencing*. Jab aap `object.something()` likhte ho, Rust automatically `&`, `&mut`, ya `*` add kar deta hai taaki object method signature ke sath match kar sake.
>
> Example:
>
> ```rust
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> Dono same kaam karte hain. Rust ye isliye kar pata hai kyunki methods ka ek clear receiver hota hai (`self`). Rust ye deduce kar leta hai ki method `&self`, `&mut self`, ya `self` expect kar raha hai.

### Methods with More Parameters

Ab ek aur method banate hain jo ek rectangle ko dusre rectangle ke andar fit hone ka check kare. Iska naam hoga `can_hold`, aur yeh ek aur `Rectangle` ka reference parameter lega. Listing 5-14 me dikhaya gaya code is method ko call karta hai.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = Rectangle { width: 10, height: 40 };
    let rect3 = Rectangle { width: 60, height: 45 };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

<span class="caption">Listing 5-14: `can_hold` method ka use</span>

Expected output:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Ab `can_hold` method ko implement karte hain:

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

<span class="caption">Listing 5-15: `can_hold` method implement karna</span>

### Associated Functions

`impl` block ke andar hum aise functions bhi define kar sakte hain jo `self` nahi lete. Inhe *associated functions* kehte hain. Ye methods nahi hote, bas struct ke saath associated normal functions hote hain. Example: `String::from`.

Hum ek constructor jaisa associated function bana sakte hain jo ek square rectangle return kare:

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

Call karte waqt: `let sq = Rectangle::square(3);`

### Multiple `impl` Blocks

Ek struct ke multiple `impl` blocks ho sakte hain. Example (Listing 5-16):

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

Yeh syntax valid hai aur kuch cases me useful hota hai (jaise Chapter 10 me generic types aur traits ke sath).

## Summary

Structs aapko custom types banane ki facility dete hain jo aapke domain ke liye meaningful hote hain. Inke through aap related data ko connect karke rak sakte ho. Methods aapko allow karte hain ki aap instances ke behavior define karo, aur associated functions aapko struct ke saath namespace ke through functionality dene dete hain.

Ab agla topic: Rust ke enums, jo ek aur tool hain custom types banane ke liye.
