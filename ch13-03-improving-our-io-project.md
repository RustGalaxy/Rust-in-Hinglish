# Improving Our I/O Project

Iterators ke baare mein is nayi knowledge se, hum Chapter 12 ke I/O project ko improve kar sakte hain. Iterators ka use karke hum code ko aur bhi clear aur concise bana sakte hain. Chaliye dekhte hain ki iterators kaise hamare `Config::new` function aur `search` function ko improve kar sakte hain.

-----

### Removing a `clone` Using an Iterator

Listing 12-6 mein, humne ek code add kiya tha jo `String` values ka ek slice leta tha aur slice ko index karke aur values ko clone karke `Config` struct ka ek instance banata tha, jisse `Config` struct un values ko **own** kar paaye. Listing 13-24 mein, humne `Config::new` function ko reproduce kiya hai jaisa ki woh Listing 12-23 mein tha:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

\<span class="caption"\>Listing 13-24: Listing 12-23 se `Config::new` function ka reproduction\</span\>

Uss samay, humne kaha tha ki **inefficient `clone` calls** ki chinta mat karna, kyunki hum unhe future mein remove kar denge. Well, ab woh samay aa gaya hai\! 🥳

Hume yahaan `clone` ki zaroorat thi kyunki hamare paas parameter `args` mein `String` elements ka ek slice hai, lekin `new` function `args` ko own nahi karta. `Config` instance ki ownership return karne ke liye, humein `Config` ke `query` aur `filename` fields se values ko clone karna pada, taki `Config` instance apni values ko **own** kar paaye.

Iterators ke baare mein hamari nayi knowledge ke saath, hum `new` function ko change kar sakte hain taki woh ek slice borrow karne ke bajaye ek iterator ki ownership le. Hum slice ki length check karne aur specific locations ko index karne wale code ki jagah iterator functionality ka use karenge. Isse `Config::new` function kya kar raha hai, yeh aur bhi clear ho jayega kyunki iterator hi values ko access karega.

Ek baar jab `Config::new` iterator ki ownership le lega aur indexing operations ka use karna band kar dega jo borrow karte hain, tab hum `clone` call karne aur ek naya allocation banane ke bajaye `String` values ko iterator se `Config` mein **move** kar sakte hain.

#### Using the Returned Iterator Directly

Apne I/O project ki *src/main.rs* file kholein, jo is tarah dikhni chahiye:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
}
```

Hum `main` function ki shuruat ko Listing 12-24 se Listing 13-25 ke code mein change kar denge. Yeh tab tak compile nahi hoga jab tak hum `Config::new` ko bhi update nahi karte.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let config = Config::new(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
}
```

\<span class="caption"\>Listing 13-25: `env::args` ki return value ko `Config::new` mein pass karna\</span\>

`env::args` function ek iterator return karta hai\! Iterator values ko ek vector mein collect karne aur fir `Config::new` ko ek slice pass karne ke bajaye, ab hum `env::args` se return hone wale iterator ki ownership seedhe `Config::new` ko pass kar rahe hain.

Next, humein `Config::new` ki definition ko update karna hoga. Apne I/O project ki *src/lib.rs* file mein, chaliye `Config::new` ke signature ko Listing 13-26 jaisa banate hain. Yeh ab bhi compile nahi hoga kyunki humein function body ko update karna baaki hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        // --snip--
```

\<span class="caption"\>Listing 13-26: `Config::new` ke signature ko iterator expect karne ke liye update karna\</span\>

Standard library documentation `env::args` function ke liye dikhati hai ki jo iterator yeh return karta hai, uski type `std::env::Args` hai. Humne `Config::new` function ke signature ko update kiya hai taki parameter `args` ki type `&[String]` ke bajaye `std::env::Args` ho. Kyunki hum `args` ki ownership le rahe hain aur isko iterate karke `args` ko **mutate** karenge, hum `args` parameter ke specification mein `mut` keyword add kar sakte hain.

#### Using `Iterator` Trait Methods Instead of Indexing

Next, hum `Config::new` ki body ko fix karenge. Standard library documentation yeh bhi mention karti hai ki `std::env::Args` `Iterator` trait ko implement karta hai, isliye hum is par `next` method call kar sakte hain\! Listing 13-27 Listing 12-23 ke code ko `next` method use karne ke liye update karti hai:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# fn main() {}
# use std::env;
#
# struct Config {
#     query: String,
#     filename: String,
#     case_sensitive: bool,
# }
#
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let filename = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file name"),
        };

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

\<span class="caption"\>Listing 13-27: `Config::new` ki body ko iterator methods use karne ke liye change karna\</span\>

Yaad rahe ki `env::args` ke return value mein pehli value program ka naam hota hai. Hum usse ignore karna chahte hain aur agli value par jaana chahte hain, isliye pehle hum `next` call karte hain aur return value ke saath kuch nahi karte. Dusri baar, hum `next` call karte hain taaki hum woh value le sakein jo hum `Config` ke `query` field mein daalna chahte hain. Agar `next` `Some` return karta hai, to hum `match` ka use karke value ko extract karte hain. Agar yeh `None` return karta hai, to iska matlab hai ki enough arguments nahi diye gaye the aur hum ek `Err` value ke saath early return kar jaate hain. Hum `filename` value ke liye bhi same cheez karte hain.

-----

### Making Code Clearer with Iterator Adaptors

Hum apne I/O project ke `search` function mein bhi iterators ka faida utha sakte hain, jo yahaan Listing 13-28 mein reproduce kiya gaya hai jaisa ki woh Listing 12-19 mein tha:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

\<span class="caption"\>Listing 13-28: Listing 12-19 se `search` function ka implementation\</span\>

Hum is code ko **iterator adaptor methods** ka use karke aur bhi concise tareeke se likh sakte hain. Aisa karne se humein ek mutable intermediate `results` vector rakhne se bhi bachata hai. Functional programming style **mutable state** ko kam se kam rakhna pasand karta hai taki code clearer ho. Mutable state ko hatane se future mein searching ko parallel mein karne ki possibility ho sakti hai, kyunki humein `results` vector tak concurrent access ko manage nahi karna padega. Listing 13-29 yeh change dikhati hai:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents.lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

\<span class="caption"\>Listing 13-29: `search` function ke implementation mein iterator adaptor methods ka use karna\</span\>

Yaad rahe ki `search` function ka purpose `contents` mein un sabhi lines ko return karna hai jinme `query` ho. Listing 13-19 ke `filter` example ki tarah, yeh code `filter` adaptor ka use karta hai taaki un lines ko rakha ja sake jin par `line.contains(query)` `true` return karta hai. Fir hum matching lines ko `collect` ke saath ek aur vector mein collect karte hain. Kitna simple hai\! Aap `search_case_insensitive` function mein bhi same change kar sakte hain.

Agla logical sawal yeh hai ki aapko apne code mein kaunsa style choose karna chahiye aur kyun: Listing 13-28 mein original implementation ya Listing 13-29 mein iterators ka version. Zyadaatar Rust programmers **iterator style** ka use karna pasand karte hain. Shuruat mein iski aadat daalna thoda mushkil ho sakta hai, lekin ek baar jab aapko various iterator adaptors aur woh kya karte hain, iska feel aa jaata hai, to iterators ko samajhna aasan ho jata hai. Looping ke various bits aur naye vectors banane ki jagah, code loop ke **high-level objective** par focus karta hai. Isse kuch commonplace code abstract ho jata hai, taki is code ke liye unique concepts ko dekhna aasan ho, jaise ki filtering condition jise iterator mein har element ko pass karna hoga.

Lekin kya dono implementations sach mein equivalent hain? Ek intuitive assumption yeh ho sakti hai ki jo zyada low-level loop hai, woh faster hoga. Chaliye performance ke baare mein baat karte hain.
