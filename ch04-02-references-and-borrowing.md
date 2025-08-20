Rust me, **references** ek value ka sandarbh dene ke liye use kiye jate hain, bina uski **ownership** liye. Yeh ek tarah se data ko "borrow" karne jaisa hai.

-----

### References aur Borrowing

Listing 4-5 mein tuple code ke saath issue yeh tha ki humein `String` ko calling function mein wapas karna padta tha, taaki hum `calculate_length` call ke baad bhi `String` ka upyog kar sakein, kyunki `String` ko `calculate_length` mein `move` kar diya gaya tha.

Yahaan, `calculate_length` function ko define aur use karne ka tarika hai jismein ek value ki ownership lene ke bajaye, parameter ke roop mein ek object ka reference liya gaya hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

Sabse pehle, dhyan dein ki variable declaration aur function return value mein saara tuple code hat gaya hai. Dusra, dhyan dein ki hum `&s1` ko `calculate_length` mein pass karte hain, aur iski definition mein, hum `String` ke bajaye `&String` lete hain.

Yeh ampersands **references** hain, aur yeh aapko kisi value ka sandarbh dene ki permission dete hain, bina uski ownership liye. Figure 4-5 mein ek diagram dikhaya gaya hai.

\<span class="caption"\>Figure 4-5: `&String s` ka `String s1` ko point karte hue diagram\</span\>

> **Note:** `&` ka upyog karke referencing ka ulta **dereferencing** hai, jise dereference operator, `*`, se poora kiya jata hai. Hum Chapter 8 mein dereference operator ke kuch upyog dekhenge aur Chapter 15 mein dereferencing ke details par charcha karenge.

Chaliye, yahaan function call ko aur gaur se dekhte hain:

```rust
# fn calculate_length(s: &String) -> usize {
#     s.len()
# }
let s1 = String::from("hello");

let len = calculate_length(&s1);
```

`&s1` syntax humein ek reference banane deta hai jo `s1` ki value ko **refer** karta hai, lekin uski ownership nahi leta hai. Kyunki yeh uski ownership nahi leta hai, toh jab yeh reference scope se bahar jata hai, toh yeh jisse point kar raha hai woh value drop nahi hogi.

Isi tarah, function ka signature `&` ka upyog karta hai yeh batane ke liye ki parameter `s` ka type ek reference hai. Chaliye kuch explanatory annotations jodte hain:

```rust
fn calculate_length(s: &String) -> usize { // s ek String ka reference hai
    s.len()
} // Yahan, s scope se bahar chala jata hai. Lekin kyunki iske paas uski ownership nahi hai
  // jise yeh refer kar raha hai, kuch nahi hota hai.
```

Jis scope mein variable `s` valid hai, woh kisi bhi function parameter ke scope jaisa hi hai, lekin hum usko drop nahi karte jise reference point kar raha hai, kyunki hamare paas ownership nahi hai. Jab functions mein parameters ke roop mein actual values ke bajaye references hote hain, toh humein ownership wapas dene ke liye values ko return karne ki zaroorat nahi padegi, kyunki hamare paas kabhi ownership thi hi nahi.

Function parameters ke roop mein references hone ko hum **borrowing** kehte hain. Jaise real life mein, agar kisi ke paas kuch hai, toh aap unse borrow kar sakte hain. Jab aapka kaam ho jaye, toh aapko use wapas karna padta hai.

Toh kya hota hai agar hum usko modify karne ki koshish karte hain jise hum borrow kar rahe hain? Listing 4-6 mein diye gaye code ko try karein. Spoiler: yeh kaam nahi karta\!

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

\<span class="caption"\>Listing 4-6: Borrow ki gayi value ko modify karne ka prayas\</span\>

Yahaan error hai:

```text
error[E0596]: cannot borrow immutable borrowed content `*some_string` as mutable
 --> error.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                         ------- use `&mut String` here to make mutable
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^ cannot borrow as mutable
```

Jaise variables by default **immutable** hote hain, waise hi references bhi hote hain. Hum usko modify nahi kar sakte jiska hamare paas reference hai.

-----

### `Mutable References` (Mutable References)

Hum Listing 4-6 se code mein ek chhota sa tweak karke error theek kar sakte hain:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

Sabse pehle, humein `s` ko `mut` banana pada. Phir humein `&mut s` ke saath ek mutable reference banana pada aur `some_string: &mut String` ke saath ek mutable reference accept karna pada.

Lekin mutable references ki ek badi restriction hai: aap ek particular scope mein ek particular data ke liye sirf ek mutable reference rakh sakte hain. Yeh code fail ho jayega:

```rust,ignore
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;
```

Yahaan error hai:

```text
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> borrow_twice.rs:5:19
  |
4 |     let r1 = &mut s;
  |                     - first mutable borrow occurs here
5 |     let r2 = &mut s;
  |                   ^ second mutable borrow occurs here
6 | }
  | - first borrow ends here
```

Yeh restriction mutation ki permission deta hai, lekin ek bahut hi controlled tarike se. Yeh ek aisi cheez hai jisse naye Rustaceans ko mushkil hoti hai, kyunki zyadatar languages aapko jab chahe tab mutate karne deti hain.

Is restriction ka fayda yeh hai ki Rust compile time par **data races** ko rok sakta hai. Ek **data race** ek race condition jaisa hi hai aur tab hota hai jab yeh teen behaviors hote hain:

  * Do ya do se zyada pointers ek hi samay mein ek hi data ko access karte hain.
  * Pointers mein se kam se kam ek data ko likhne ke liye upyog kiya ja raha hai.
  * Data ko access karne ke liye koi mechanism use nahi ho raha hai.

Data races **undefined behavior** ka karan bante hain aur runtime par unhe dhundhna aur theek karna mushkil ho sakta hai; Rust is problem ko hone se rokta hai, kyunki yeh data races wale code ko compile hi nahi hone dega\!

Jaisa ki hamesha hota hai, hum ek naya scope banane ke liye curly brackets ka upyog kar sakte hain, jisse multiple mutable references ki permission milti hai, bas woh **simultaneous** na hon:

```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;

} // r1 yahan scope se bahar chala jata hai, isliye hum bina kisi problem ke ek naya reference bana sakte hain.

let r2 = &mut s;
```

Mutable aur immutable references ko combine karne ke liye ek similar rule maujood hai. Yeh code error deta hai:

```rust,ignore
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM
```

Yahaan error hai:

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as
immutable
 --> borrow_thrice.rs:6:19
  |
4 |     let r1 = &s; // no problem
  |                    - immutable borrow occurs here
5 |     let r2 = &s; // no problem
6 |     let r3 = &mut s; // BIG PROBLEM
  |                   ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

Hum ek **mutable reference** nahi rakh sakte jab hamare paas ek **immutable reference** ho. Immutable reference ka upyog karne wale users ko yeh ummeed nahi hoti ki values achanak se unke neeche se badal jayengi\! Lekin, multiple immutable references theek hain, kyunki jo bhi sirf data padh raha hai uske paas kisi aur ke data padhne ko prabhavit karne ki shamta nahi hai.

Halanki yeh errors kabhi-kabhi frustrating ho sakte hain, yaad rakhein ki Rust compiler ek potential bug ko shuru mein hi (compile time par na ki runtime par) point out kar raha hai aur aapko exact problem kahan hai, yeh dikha raha hai. Phir aapko yeh track karne ki zaroorat nahi padegi ki aapka data woh kyun nahi hai jo aapne socha tha.

-----

### `Dangling References` (Dangling References)

Pointers wali languages mein, ek **dangling pointer** banana aasan hota hai, jo ek aisi memory location ko refer karta hai jo shayad kisi aur ko di gayi ho, kuch memory ko free karke jabki us memory ka pointer preserve kiya jata hai. Iske vipreet, Rust mein, compiler guarantee deta hai ki references kabhi bhi dangling references nahi honge: agar aapke paas kuch data ka reference hai, toh compiler yeh ensure karega ki woh data reference ke scope se bahar jane se pehle scope se bahar na chala jaye.

Chaliye ek dangling reference banane ki koshish karte hain, jise Rust compile-time error ke saath rokega:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

Yahaan error hai:

```text
error[E0106]: missing lifetime specifier
 --> main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is
  no value for it to be borrowed from
  = help: consider giving it a 'static lifetime
```

Yeh error message ek feature ko refer karta hai jise humne abhi tak cover nahi kiya hai: **lifetimes**. Hum Chapter 10 mein lifetimes par detail mein charcha karenge. Lekin, agar aap lifetimes ke bare mein parts ko disregard karte hain, toh message mein yeh key hai ki yeh code problem kyun hai:

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from.
```

Chaliye gaur se dekhte hain ki hamare `dangle` code ke har stage par kya ho raha hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn dangle() -> &String { // dangle ek String ka reference return karta hai

    let s = String::from("hello"); // s ek naya String hai

    &s // hum String, s ka ek reference return karte hain
} // Yahan, s scope se bahar chala jata hai, aur drop ho jata hai. Iski memory chali jati hai.
  // Danger!
```

Kyunki `s` `dangle` ke andar banaya gaya hai, jab `dangle` ka code khatam ho jata hai, toh `s` deallocate ho jayega. Lekin humne uska reference return karne ki koshish ki. Iska matlab hai ki yeh reference ek invalid `String` ko point kar raha hoga. Yeh theek nahi hai\! Rust humein yeh karne nahi dega.

Yahaan solution yeh hai ki `String` ko sidhe return kiya jaye:

```rust
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

Yeh bina kisi problem ke kaam karta hai. Ownership bahar move ho jati hai, aur kuch bhi deallocate nahi hota hai.

-----

### `The Rules of References` (References ke Niyam)

Chaliye, humne references ke bare mein jo discuss kiya hai, use recap karte hain:

  * Ek samay mein, aapke paas **ya toh** ek mutable reference **ho sakta hai, ya** kitne bhi **immutable** references ho sakte hain.
  * References hamesha **valid** hone chahiye.

Aage, hum ek alag tarah ke reference ko dekhenge: **slices**.
