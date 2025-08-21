# Lifetimes se References ko Valid banana ✅

**Chapter 4** mein "References and Borrowing" section mein humne ek detail miss kar di thi ki Rust mein har reference ka ek **lifetime** hota hai, jo us reference ke valid hone ka **scope** hai. Zyadatar samay, lifetimes implicit (chhupe hue) aur infer kiye jaate hain, jaise ki types ko infer kiya jaata hai. Hamein tabhi types ko annotate karna padta hai jab multiple types possible ho. Isi tarah, humein tabhi **lifetimes ko annotate** karna padta hai jab references ki lifetimes alag-alag tareekon se related ho sakti hain. Rust isse **generic lifetime parameters** ka upyog karke annotate karne ki zaroorat rakhta hai, taaki runtime par use hone wale references hamesha valid hon.

Lifetimes ka concept doosri programming languages ke tools se alag hai, isliye lifetimes ko Rust ka sabse alag feature mana jaata hai.

-----

### Lifetimes se Dangling References ko Rokna 🚫

Lifetimes ka main maqsad **dangling references** ko rokna hai. Dangling references ki wajah se program us data ko refer karta hai jise wo refer karna nahi chahta. **Listing 10-17** mein diye gaye program ko dekhiye jismein ek outer scope aur ek inner scope hai.

```rust,ignore
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

\<span class="caption"\>Listing 10-17: Ek reference ka upyog karne ki koshish jiski value scope se bahar chali gayi hai\</span\>

Ye code compile nahi hoga, kyunki `r` jis value ko refer kar raha hai, wo uske use hone se pehle hi scope se bahar chali gayi hai. Error message ye hoga:

```text
error[E0597]: `x` does not live long enough
  --> src/main.rs:7:5
   |
6  |         r = &x;
   |              - borrow occurs here
7  |     }
   |     ^ `x` dropped here while still borrowed
...
10 | }
   | - borrowed value needs to live until here
```

`x` ka lifetime itna lamba nahi hai. `x` inner scope ke end hone par scope se bahar ho jaayega. Lekin `r` outer scope ke liye abhi bhi valid hai. Agar Rust is code ko kaam karne deta, to `r` us memory ko refer kar raha hota jo `x` ke scope se bahar jaane par deallocate ho gayi thi, aur `r` ke saath hum jo kuch bhi karne ki koshish karte wo sahi se kaam nahi karta.

-----

### The Borrow Checker 🕵️

Rust compiler ke paas ek **borrow checker** hai jo scopes ko compare karke determine karta hai ki saare borrows valid hain ya nahi. **Listing 10-18** mein, Listing 10-17 jaisa hi code hai, lekin ismein variables ki lifetimes ko annotate kiya gaya hai.

```rust,ignore
{
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```

\<span class="caption"\>Listing 10-18: `r` aur `x` ki lifetimes ka annotation, jise respectively `'a` aur `'b` naam diya gaya hai\</span\>

Yahan, humne `r` ki lifetime ko `'a` se aur `x` ki lifetime ko `'b` se annotate kiya hai. Jaisa ki aap dekh sakte hain, inner `'b` block outer `'a` lifetime block se bahut chota hai. Compile time par, Rust dono lifetimes ke size ko compare karta hai aur dekhta hai ki `r` ki lifetime `'a` hai lekin ye `'b` lifetime wali memory ko refer karta hai. Program ko reject kar diya jaata hai kyunki `'b` `'a` se chota hai.

-----

### Functions mein Generic Lifetimes 🧬

Aaiye ek function likhte hain jo do string slices mein se lambe wale ko return kare. Ye function do string slices lega aur ek string slice return karega. **Listing 10-21** mein dikhaya gaya `longest` function compile nahi hoga.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

\<span class="caption"\>Listing 10-21: `longest` function ka implementation jo do string slices mein se lambe wale ko return karta hai lekin abhi compile nahi hota\</span\>

Iske bajaye, humein ek error milegi:

```text
error[E0106]: missing lifetime specifier
 --> src/main.rs:1:33
  |
1 | fn longest(x: &str, y: &str) -> &str {
  |                                 ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the
signature does not say whether it is borrowed from `x` or `y`
```

Help text batata hai ki return type mein ek generic lifetime parameter ki zaroorat hai, kyunki Rust bata nahi sakta ki return kiya gaya reference `x` ko refer karta hai ya `y` ko. Borrow checker bhi ye decide nahi kar sakta, kyunki use nahi pata ki `x` aur `y` ki lifetimes return value ki lifetime se kaise related hain. Is error ko fix karne ke liye, hum **generic lifetime parameters** add karenge jo references ke beech ka relationship define karte hain.

-----

### Lifetime Annotation ka Syntax 🖋️

Lifetime annotations references kitna lamba live karte hain, ismein koi change nahi late. Bilkul waise hi jaise functions kisi bhi type ko accept kar sakte hain, functions kisi bhi lifetime ke references ko generic lifetime parameter specify karke accept kar sakte hain.

Lifetime annotations ka syntax thoda unusual hai: lifetime parameters ke naam apostrophe (`'`) se shuru hone chahiye aur aam taur par lowercase aur bahut chote hote hain, jaise ki `T`. Zyadatar log `'a` naam ka use karte hain. Hum reference ke `&` ke baad lifetime parameter annotations rakhte hain.

```rust,ignore
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

-----

### Function Signatures mein Lifetime Annotations 🎯

Ab hum `longest` function ke context mein lifetime annotations ko dekhte hain. Generic type parameters ki tarah, humein generic lifetime parameters ko angle brackets ke andar function ke naam aur parameter list ke beech declare karna padta hai. Jo constraint hum is signature mein express karna chahte hain, wo ye hai ki parameters aur return value mein saare references ki lifetime same honi chahiye. Hum lifetime ko `'a` naam denge aur use har reference mein add karenge, jaisa **Listing 10-22** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

\<span class="caption"\>Listing 10-22: `longest` function definition ye specify karti hai ki signature mein saare references ki same lifetime `'a` honi chahiye\</span\>

Ye code compile hona chahiye aur hamein desired result dena chahiye. Function signature ab Rust ko batati hai ki kisi lifetime `'a` ke liye, function do parameters leta hai, jinmein se dono string slices hain jo lifetime `'a` jitna ya usse zyaada lambe live karte hain. Function signature ye bhi batati hai ki function se return hone wala string slice bhi lifetime `'a` jitna ya usse zyaada lamba live karega.

**Listing 10-23** mein, `string1` outer scope ke end tak valid hai, `string2` inner scope ke end tak valid hai, aur `result` ek aisi cheez ko refer karta hai jo inner scope ke end tak valid hai. Ye code compile hoga.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
#     if x.len() > y.len() {
#         x
#     } else {
#         y
#     }
# }
#
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

\<span class="caption"\>Listing 10-23: `longest` function ka upyog `String` values ke references ke saath karna jinki concrete lifetimes alag hain\</span\>

Lekin **Listing 10-24** mein, hum `result` ko inner scope ke bahar declare karte hain. Jab hum is code ko compile karne ki koshish karte hain, to hamein error milti hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

\<span class="caption"\>Listing 10-24: `result` ka upyog karne ki koshish `string2` ke scope se bahar jaane ke baad\</span\>

Borrow checker ye code allow nahi karta kyunki humne Rust ko bataya hai ki `longest` function se return hone wale reference ki lifetime do parameters ki choti lifetime ke barabar hoti hai.

-----

### Lifetimes ke Terms mein Sochna 🤔

Jab ek function se reference return karte hain, to return type ke liye lifetime parameter ko kisi ek parameter ke lifetime parameter se match karna zaroori hai. Agar return kiya gaya reference kisi parameter ko refer nahi karta, to wo ek **dangling reference** hoga. **Listing 10-25** mein, ek struct `ImportantExcerpt` hai jo ek string slice ko hold karta hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

\<span class="caption"\>Listing 10-25: Ek struct jo ek reference ko hold karta hai, isliye uski definition ko ek lifetime annotation chahiye\</span\>

Ye struct ka ek field `part` hai jo ek string slice ko hold karta hai. Generic data types ki tarah, hum generic lifetime parameter ka naam struct ke naam ke baad angle brackets ke andar declare karte hain. Is annotation ka matlab hai ki `ImportantExcerpt` ka instance us reference se lamba live nahi kar sakta jo wo apne `part` field mein hold karta hai.

-----

### Lifetime Elision 🔕

**Lifetime elision** rules kehte hain ki Rust compiler kuch predictable situations mein lifetimes ko infer kar sakta hai, isliye aapko explicit annotations nahi likhni padti. Ye rules programmers ko follow karne ke liye nahi hain, balki ye kuch khaas cases ka set hai jise compiler consider karega.

Compiler teen rules ka upyog karta hai taaki wo pata laga sake ki references ki lifetimes kya hain jab explicit annotations nahi hoti.

1.  Har parameter jo reference hai, use apna khud ka lifetime parameter milta hai.
2.  Agar bilkul ek input lifetime parameter hai, to wo lifetime saare output lifetime parameters ko assign kar diya jaata hai.
3.  Agar multiple input lifetime parameters hain, lekin unmein se ek `&self` ya `&mut self` hai, to `self` ki lifetime saare output lifetime parameters ko assign kar di jaati hai. Ye teesra rule methods ko padhne aur likhne mein bahut aasan banata hai.

-----

### Static Lifetime 🕰️

Ek special lifetime hai **`'static`**, jo program ki poori duration ko denote karta hai. Saare string literals ki `'static` lifetime hoti hai.

```rust
let s: &'static str = "I have a static lifetime.";
```

Ye string ka text directly aapke program ki binary mein store hota hai, jo hamesha available hota hai. Isliye, saare string literals ki lifetime `'static` hoti hai.

-----

### Generic Type Parameters, Trait Bounds, aur Lifetimes Ek Saath ✨

Aaiye ek function ke syntax ko dekhte hain jismein generic type parameters, trait bounds, aur lifetimes sab ek saath hain\!

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Ye **Listing 10-22** ka `longest` function hai. Lekin ab ismein ek extra parameter `ann` hai jiska generic type `T` hai, jo `Display` trait ko implement karne wale kisi bhi type se fill kiya ja sakta hai.

-----

## Summary 📝

Humne is chapter mein bahut kuch cover kiya hai\! Ab jab aap generic type parameters, traits aur trait bounds, aur generic lifetime parameters ke baare mein jaante hain, to aap bina repetition ke code likhne ke liye taiyar hain jo kayi alag-alag situations mein kaam karta hai. Generic type parameters aapko alag-alag types par code apply karne dete hain. Traits aur trait bounds ensure karte hain ki generic types hone ke bawjood, unmein wo behavior hoga jo code ko chahiye. Aapne lifetime annotations ka upyog karna seekha taaki flexible code mein koi dangling references na ho. Aur ye sab analysis compile time par hota hai, jiska runtime performance par koi asar nahi padta\!

Aage, aap Rust mein tests likhna seekhenge taaki aap ensure kar saken ki aapka code bilkul sahi kaam kar raha hai.
