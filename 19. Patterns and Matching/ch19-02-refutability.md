## Refutability: Kya ek Pattern Match Karne mein Fail Ho Sakta Hai

Patterns do roop mein aate hain: **refutable** aur **irrefutable**. 🤔

Aise patterns jo kisi bhi sambhavit (possible) value ke liye match karenge, **irrefutable** kehlate hain. Iska ek udaharan `let x = 5;` statement mein `x` hai, kyunki `x` kisi bhi cheez se match karta hai aur isliye match karne mein fail nahi ho sakta. ✅

Aise patterns jo kisi sambhavit value ke liye match karne mein fail ho sakte hain, **refutable** kehlate hain. Iska ek udaharan `if let Some(x) = a_value` expression mein `Some(x)` hai, kyunki agar `a_value` variable mein `Some` ke bajaye `None` hai, to `Some(x)` pattern match nahi karega. ❌

-----

Function parameters, `let` statements, aur `for` loops sirf **irrefutable patterns** hi accept kar sakte hain kyunki jab values match nahi karti hain to program kuch bhi arthpurn (meaningful) nahi kar sakta. `if let` aur `while let` expressions **refutable aur irrefutable patterns** dono ko accept karte hain, lekin compiler irrefutable patterns ke khilaaf chetavani (warns) deta hai, kyunki unka uddeshya sambhavit failure ko handle karna hai.

Aam taur par, aapko refutable aur irrefutable patterns ke beech ke antar ke baare mein chinta karne ki zaroorat nahi hai; halaanki, aapko refutability ke concept se parichit hona chahiye taaki jab aap ise ek error message mein dekhein to aap pratikriya de sakein.

Chaliye ek udaharan dekhte hain ki kya hota hai jab hum ek refutable pattern ka istemal karne ki koshish karte hain jahan Rust ko ek irrefutable pattern ki avashyakta hoti hai. Listing 19-8 mein ek `let` statement dikhaya gaya hai, lekin pattern ke liye, humne `Some(x)` specify kiya hai, jo ek refutable pattern hai. Yah code compile nahi hoga.

```rust,ignore,does_not_compile
// Filename: src/main.rs
fn main() {
    let some_option_value: Option<i32> = None;
    let Some(x) = some_option_value;
}
```

*Listing 19-8: `let` ke saath ek refutable pattern ka istemal karne ka prayas*

Agar `some_option_value` ek `None` value hota, to yah pattern `Some(x)` se match karne mein fail ho jaata. Halaanki, `let` statement keval ek irrefutable pattern hi accept kar sakta hai. Compile time par, Rust shikayat karega:

```console
error: refutable pattern in local binding: `None` not covered
 --> src/main.rs:3:9
  |
3 |     let Some(x) = some_option_value;
  |         ^^^^^^^ pattern `None` not covered
```

Kyunki humne pattern `Some(x)` ke saath har valid value ko cover nahi kiya, Rust ne sahi tarike se ek compiler error utpann kiya.

Agar hamare paas ek refutable pattern hai jahan ek irrefutable pattern ki zaroorat hai, to hum us code ko badal kar ise theek kar sakte hain jo pattern ka istemal karta hai: `let` ka istemal karne ke bajaye, hum `if let` ya `let...else` ka istemal kar sakte hain. Phir, agar pattern match nahi karta hai, to code bas curly brackets mein code ko chhod dega, jisse use valid roop se jaari rakhne ka ek tareeka mil jaayega.

```rust
// Filename: src/main.rs
fn main() {
    let some_option_value: Option<i32> = None;
    if let Some(x) = some_option_value {
        println!("{}", x);
    }
}
```

*Listing 19-9: `let` ke bajaye refutable patterns ke saath `if let` ka istemal karna*

Humne code ko ek raasta de diya\! Yah code bilkul valid hai.

Iske vipreet, agar hum `if let` ko ek aisa pattern dete hain jo hamesha match karega, jaise `x`, to compiler ek chetavani dega.

```rust
// Filename: src/main.rs
fn main() {
    if let x = 5 {
        println!("{}", x);
    };
}
```

*Listing 19-10: `if let` ke saath ek irrefutable pattern ka istemal karne ka prayas*

Rust shikayat karta hai ki `if let` ka ek irrefutable pattern ke saath istemal karne ka koi matlab nahi hai:

```console
warning: irrefutable `if let` pattern
 --> src/main.rs:2:8
  |
2 |     if let x = 5 {
  |        ^^^^^^^^
  |
  = note: `#[warn(irrefutable_let_patterns)]` on by default
  = note: this pattern will always match, so the `if let` is useless
  = help: consider replacing the `if let` with a `let`
```

Isi kaaran se, `match` arms ko refutable patterns ka istemal karna chahiye, sivay aakhri arm ke, jise kisi bhi shesh (remaining) value ko ek irrefutable pattern ke saath match karna chahiye.

Ab jab aap jaante hain ki patterns ka istemal kahan karna hai aur refutable aur irrefutable patterns ke beech kya antar hai, chaliye un sabhi syntax ko cover karte hain jinka istemal hum patterns banane ke liye kar sakte hain.