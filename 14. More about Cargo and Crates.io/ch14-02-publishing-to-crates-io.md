## Ek Crate ko Crates.io par Publish Karna

Humne abhi tak apne project mein dependencies ke taur par [crates.io](https://crates.io) se packages use kiye hain, lekin aap apne khud ke packages publish karke apna code doosre logon ke saath bhi share kar sakte hain. [crates.io](https://crates.io) ki crate registry aapke packages ka source code distribute karti hai, isliye yeh zyadatar open-source code hi host karti hai.

Rust aur Cargo mein aise features hain jisse aapka publish kiya gaya package logon ke liye use karna aur use dhoondhna aasan ho jaata hai. Hum aage inmein se kuch features ke baare mein baat karenge aur phir batayenge ki ek package kaise publish karte hain.

-----

### Kaam ke Documentation Comments Banana

Apne packages ko theek se document karne se doosre users ko yeh samajhne mein madad milti hai ki unhe kab aur kaise use karna hai, isliye documentation likhne mein time invest karna ahem hai. Chapter 3 mein, humne `//` (do slashes) ka use karke Rust code ko comment karna seekha tha. Rust mein documentation ke liye ek khaas tarah ka comment bhi hota hai, jise ***documentation comment*** kehte hain, jo HTML documentation generate karta hai. Yeh HTML un public API items ke liye documentation comments ka content dikhata hai jo un programmers ke liye hai jo yeh jaanna chahte hain ki aapke crate ko *kaise use karna hai*, na ki yeh ki aapka crate *kaise implement kiya gaya hai*.

Documentation comments mein do ki jagah **teen slashes (`///`)** ka istemaal hota hai aur yeh text ko format karne ke liye Markdown notation support karte hain. Documentation comments ko us item se theek pehle rakhein jise woh document kar rahe hain. Niche `my_crate` naam ke ek crate mein `add_one` function ke liye documentation comments dikhaye gaye hain:

\<span class="filename"\>Filename: src/lib.rs\</span\>

````rust,ignore
/// Diye gaye number mein one add karta hai.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, my_crate::add_one(5));
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
````

\<span class="caption"\>Listing 14-1: Ek function ke liye documentation comment\</span\>

Yahan, humne bataya hai ki `add_one` function kya karta hai, `Examples` heading se ek section shuru kiya hai, aur phir ek code example diya hai jo batata hai ki `add_one` function ko kaise use karna hai. Hum is documentation comment se `cargo doc` command chala kar HTML documentation generate kar sakte hain. Yeh command Rust ke saath aane wale `rustdoc` tool ko chalata hai aur generate ki hui HTML documentation ko *target/doc* directory mein daal deta hai.

Suvidha ke liye, `cargo doc --open` chalaane se aapke current crate (aur uski saari dependencies) ke liye HTML documentation banakar use web browser mein khol dega. `add_one` function par navigate karein aur aap dekhenge ki documentation comments ka text kaise render hota hai.

#### Aam Taur par Use Hone Wale Sections

Humne upar HTML mein "Examples" title ka section banane ke liye `# Examples` Markdown heading ka use kiya tha. Yahan kuch aur sections hain jo crate authors aam taur par apne documentation mein use karte hain:

  * **Panics**: Woh scenarios jinmein function panic kar sakta hai. Jo log is function ko call kar rahe hain aur nahi chahte ki unke program panic karein, unhe yeh sunishchit karna chahiye ki woh in situations mein function ko call na karein.
  * **Errors**: Agar function ek `Result` return karta hai, to yeh batana ki kis tarah ki errors aa sakti hain aur kin conditions mein woh errors aa sakti hain, callers ke liye madadgaar ho sakta hai. Isse woh alag-alag tarah ki errors ko handle karne ke liye code likh sakte hain.
  * **Safety**: Agar function call karna `unsafe` hai (hum Chapter 19 mein unsafety par charcha karenge), to ek section hona chahiye jo samjhaye ki function unsafe kyun hai aur woh kya-kya shartein (invariants) hain jinka callers ko paalan karna chahiye.

Zyadatar documentation comments ko in sabhi sections ki zaroorat nahi hoti, lekin yeh ek achhi checklist hai jo aapko aapke code ke un pehluon ki yaad dilati hai jinke baare mein aapke code ko call karne wale log jaanna chahenge.

#### Documentation Comments as Tests

Apne documentation comments mein code examples add karne se yeh dikhane mein madad milti hai ki aapki library ko kaise use karna hai, aur iska ek extra fayda bhi hai: **`cargo test` chalaane se aapke documentation ke code examples tests ke taur par run honge\!** Examples ke saath documentation se behtar kuch nahi hota. Lekin aise examples se bura kuch nahi hota jo kaam nahi karte kyunki documentation likhe jaane ke baad code badal gaya hai. Agar hum upar wale `add_one` function ke documentation ke saath `cargo test` chalaate hain, to humein test results mein ek section dikhega:

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Ab agar hum function ya example ko badal dete hain taaki example ka `assert_eq!` panic ho jaye aur `cargo test` dobara chalate hain, to hum dekhenge ki doc tests yeh pakad lenge ki example aur code ek doosre se sync mein nahi hain\!

#### Containing Items ko Comment Karna

Ek aur tarah ka doc comment, `//!`, comments ke *baad* aane wale items ko document karne ke bajaye us item ko document karta hai jiske *andar* woh comments hain. Hum aam taur par in doc comments ka istemaal crate root file (*src/lib.rs*) ke andar ya kisi module ke andar karte hain taaki poore crate ya module ko document kiya ja sake.

For example, agar hum `my_crate` jismein `add_one` function hai, uske purpose ko describe karna chahte hain, to hum *src/lib.rs* file ki shuruaat mein `//!` se shuru hone wale documentation comments add kar sakte hain:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
//! # My Crate
//!
//! `my_crate` kuch calculations ko aasan banane ke liye utilities ka
//! ek collection hai.

/// Diye gaye number mein one add karta hai.
// --snip--
```

\<span class="caption"\>Listing 14-2: Poore `my_crate` crate ke liye documentation\</span\>

Dhyan dein ki `//!` se shuru hone wali aakhri line ke baad koi code nahi hai. Kyunki humne comments `///` ke bajaye `//!` se shuru kiye, hum us item ko document kar rahe hain jiske andar yeh comment hai, na ki iske baad aane wale item ko. Is case mein, yeh *src/lib.rs* file hai, jo crate root hai. Yeh comments poore crate ko describe karte hain.

-----

### `pub use` se ek Aasan Public API Export Karna

Aapke crate ka jo structure aapko development ke waqt sahi lagta hai, woh shayad aapke users ke liye utna suvidhajanak na ho. Aap apne structs ko kai levels ki hierarchy mein organize karna chah sakte hain, lekin tab jo log aapke hierarchy mein gehraai mein define kiye gaye type ko use karna chahte hain, unhe use dhoondhne mein mushkil ho sakti hai.

Ek crate publish karte waqt aapki **public API** ka structure ek bada consideration hai. Agar structure doosri library se use karne ke liye suvidhajanak *nahi* hai, to aapko apne internal organization ko badalne ki zaroorat nahi hai: iske bajaye, aap `pub use` ka istemaal karke items ko **re-export** kar sakte hain taaki ek public structure banaya ja sake jo aapke private structure se alag ho. Re-exporting ek jagah ke public item ko lekar use doosri jagah bhi public bana deta hai, jaise ki woh wahin par define kiya gaya ho.

Isse aapke users ko lamba path jaise `use my_crate::some_module::another_module::UsefulType;` likhne ke bajaye `use my_crate::UsefulType;` likhne ki suvidha milti hai. `pub use` ka istemaal karke aap types ko top level par re-export kar sakte hain, jisse aapke crate ko istemaal karne wale logon ka experience kaafi behtar ho jaata hai.

Ek achha public API structure banana science se zyada ek art hai, aur aap apne users ke liye sabse behtar API dhoondhne ke liye iterate kar sakte hain.

-----

### Crates.io Account Setup Karna

Koi bhi crate publish karne se pehle, aapko [crates.io](https://crates.io) par ek account banana hoga aur ek API token lena hoga. Iske liye, [crates.io](https://crates.io) ke home page par jaakar apne GitHub account se login karein. Login karne ke baad, apni account settings ([https://crates.io/me/](https://crates.io/me/)) par jaakar apna API key haasil karein. Phir `cargo login` command ko apne API key ke saath chalayein:

```text
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

Yeh command Cargo ko aapke API token ke baare mein bata dega aur use locally *\~/.cargo/credentials* mein store kar dega. Yaad rakhein ki yeh token ek **secret** hai: ise kisi ke saath share na karein.

-----

### Ek Naye Crate mein Metadata Add Karna

Ab jab aapka account ban gaya hai, to publish karne se pehle aapko apne crate ke *Cargo.toml* file ke `[package]` section mein kuch metadata add karna hoga.

Aapke crate ka naam **unique** hona chahiye. [crates.io](https://crates.io) par crate ke naam first-come, first-served basis par milte hain. Iske alawa, aapko ek **description** aur ek **license** bhi add karna hoga taaki logon ko pata chale ki aapka crate kya karta hai aur woh ise kin sharton par use kar sakte hain.

License ke liye, aapko ek *license identifier value* deni hogi. Aap [SPDX (Software Package Data Exchange)](http://spdx.org/licenses/) par identifiers ki list dekh sakte hain. Rust community mein bahut se log `MIT OR Apache-2.0` dual license ka istemaal karte hain.

Ek publish ke liye taiyar project ki *Cargo.toml* file kuch aisi dikh sakti hai:
\<span class="filename"\>Filename: Cargo.toml\</span\>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

-----

### Crates.io par Publish Karna

Ab jab aapne account bana liya hai, API token save kar liya hai, crate ka naam chun liya hai, aur zaroori metadata specify kar diya hai, to aap publish karne ke liye taiyar hain\!

Publish karte waqt savdhan rahein kyunki ek publish **permanent** hota hai. Version ko kabhi bhi overwrite nahi kiya ja sakta, aur code ko delete nahi kiya ja sakta. `cargo publish` command chalayein, aur agar sab theek raha to yeh safal ho jayega:

```text
$ cargo publish
 Updating registry `https://github.com/rust-lang/crates.io-index`
 Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
 Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
 Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
  Finished dev [unoptimized + debuginfo] target(s) in 0.19 secs
 Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

Badhai ho\! Aapne apna code Rust community ke saath share kar diya hai.

-----

### Ek Maujooda Crate ka Naya Version Publish Karna

Jab aap apne crate mein changes kar lete hain aur ek naya version release karne ke liye taiyar hote hain, to aap apne *Cargo.toml* file mein `version` value ko badalte hain aur republish karte hain. [Semantic Versioning rules][semver] ka istemaal karke decide karein ki aapke changes ke aadhar par agla version number kya hona chahiye. Phir naya version upload karne ke liye `cargo publish` chalayein.

-----

### `cargo yank` se Crates.io se Versions Hatana

Halaanki aap crate ke pichle versions ko remove nahi kar sakte, aap future projects ko unhe ek nayi dependency ke roop mein add karne se rok sakte hain. Yeh tab kaam aata hai jab koi crate version kisi wajah se achanak se kaam nahi kar raha hota hai. Aise situations mein, Cargo ek crate version ko ***yank*** karne ka support karta hai.

Ek version ko yank karne se naye projects us version par depend nahi kar paate, jabki sabhi maujooda projects jo us par depend karte hain, use download karna aur depend karna jaari rakh sakte hain.

Ek crate ke version ko yank karne ke liye, `cargo yank` chalayein aur batayein ki aap kaun sa version yank karna chahte hain:

```text
$ cargo yank --vers 1.0.1
```

Aap `--undo` flag ke saath ek yank ko undo bhi kar sakte hain:

```text
$ cargo yank --vers 1.0.1 --undo
```

Ek yank koi code **delete nahi karta**. Yeh feature galti se upload kiye gaye secrets ko delete karne ke liye nahi hai. Agar aisa hota hai, to aapko un secrets ko turant reset karna chahiye.

[semver]: https://www.google.com/search?q=%5Bhttp://semver.org/%5D\(http://semver.org/\)
