# The match Control Flow Operator

Rust mein ek bahut hi powerful control flow operator hai jise **`match`** kehte hain. Yeh aapko ek value ko patterns ki ek series ke saath compare karne deta hai aur phir us pattern ke aadhar par code execute karta hai jo match karta hai. Patterns literal values, variable names, wildcards, aur bahut si doosri cheezon se ban sakte hain; Chapter 18 sabhi alag-alag tarah ke patterns aur unke kaam ko cover karta hai. `match` ki power patterns ki expressiveness aur is fact se aati hai ki compiler yeh confirm karta hai ki sabhi possible cases ko handle kar liya gaya hai.

Ek `match` expression ko ek coin-sorting machine ki tarah sochein: coins ek track se niche slide karte hain jismein alag-alag size ke holes hote hain, aur har coin pehle hole se niche gir jata hai jismein woh fit hota hai. Isi tarah, values ek `match` ke har pattern se guzarati hain, aur pehle pattern par jismein value "fit" hoti hai, woh value us associated code block mein chali jati hai jise execution ke dauran use kiya jata hai.

Kyunki humne abhi coins ka zikr kiya, chaliye unka upyog `match` ke liye ek example ke roop mein karte hain\! Hum ek function likh sakte hain jo ek unknown United States coin le sakta hai aur, counting machine ke tarah, yeh determine kar sakta hai ki kaun sa coin hai aur uski value cents mein return kar sakta hai, jaisa ki Listing 6-3 mein dikhaya gaya hai.

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

\<span class="caption"\>Listing 6-3: Ek enum aur ek `match` expression jismein uske patterns ke roop mein enum ke variants hain\</span\>

Chaliye, `value_in_cents` function mein `match` ko todte hain. Sabse pehle, hum `match` keyword ko list karte hain jiske baad ek expression hota hai, jo is case mein `coin` value hai. Yeh `if` ke saath upyog kiye jane wale expression jaisa hi lagta hai, lekin ek bada difference hai: `if` ke saath, expression ko ek Boolean value return karne ki zaroorat hoti hai, lekin yahan, yeh koi bhi type ho sakta hai. Is example mein `coin` ka type `Coin` enum hai jise humne line 1 par define kiya hai.

Next hain `match` arms. Ek arm ke do hisse hote hain: ek **pattern** aur kuch **code**. Yahan pehle arm ka ek pattern hai jo `Coin::Penny` value hai aur phir `=>` operator jo pattern aur chalane wale code ko alag karta hai. Is case mein code sirf `1` value hai. Har arm ko agle se ek comma se alag kiya jata hai.

Jab `match` expression execute hota hai, toh yeh resulting value ko har arm ke pattern se compare karta hai, order mein. Agar ek pattern value se match karta hai, toh us pattern se associated code execute hota hai. Agar woh pattern value se match nahi karta, toh execution agle arm par chali jati hai, jaise coin-sorting machine mein hota hai. Humein jitne bhi arms chahiye, utne ho sakte hain: Listing 6-3 mein, hamare `match` mein char arms hain.

Har arm se associated code ek expression hai, aur matching arm mein expression ki resulting value woh value hai jo poore `match` expression ke liye return hoti hai.

Curly brackets ka upyog aam taur par tab nahi kiya jata jab match arm code chhota ho, jaisa Listing 6-3 mein hai jahan har arm bas ek value return karta hai. Agar aap ek match arm mein multiple lines of code chalana chahte hain, toh aap curly brackets ka upyog kar sakte hain. Udaharan ke liye, neeche diya gaya code har baar `Coin::Penny` ke saath method ko call karne par "Lucky penny\!" print karega lekin phir bhi block ki aakhri value, `1`, return karega:

```rust
# enum Coin {
#     Penny,
#     Nickel,
#     Dime,
#     Quarter,
# }
#
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

-----

### `Patterns that Bind to Values` (Values se Bind hone wale Patterns)

Match arms ka ek aur useful feature yeh hai ki woh patterns se match hone wale values ke parts se bind ho sakte hain. Is tarah se hum enum variants se values nikal sakte hain.

Udaharan ke roop mein, chaliye hum apne ek enum variants ko uske andar data hold karne ke liye badalte hain. 1999 se 2008 tak, United States ne quarters banaye jiske ek taraf 50 states mein se har ek ke liye alag designs the. Kisi aur coin mein state designs nahi mile, isliye sirf quarters mein yeh extra value hoti hai. Hum is jankari ko apne `enum` mein `Quarter` variant ko badal kar ek `UsState` value shamil karke jod sakte hain, jaisa humne Listing 6-4 mein kiya hai.

```rust
#[derive(Debug)] // takki hum state ko ek minute mein inspect kar sakein
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

\<span class="caption"\>Listing 6-4: Ek `Coin` enum jismein `Quarter` variant ek `UsState` value bhi hold karta hai\</span\>

Chaliye imagine karte hain ki hamara ek dost sabhi 50 state quarters collect karne ki koshish kar raha hai. Jab hum apne loose change ko coin type se sort karte hain, toh hum har quarter se associated state ka naam bhi batayenge taaki agar yeh uske paas na ho, toh woh use apne collection mein jod sake.

Is code ke liye match expression mein, hum `Coin::Quarter` variant ke values se match hone wale pattern mein `state` naam ka ek variable jodte hain. Jab ek `Coin::Quarter` match hota hai, toh `state` variable us quarter ke state ki value se bind ho jayega. Phir hum us arm ke code mein `state` ka upyog kar sakte hain, jaise is tarah:

```rust
# #[derive(Debug)]
# enum UsState {
#     Alabama,
#     Alaska,
# }
#
# enum Coin {
#     Penny,
#     Nickel,
#     Dime,
#     Quarter(UsState),
# }
#
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

Agar hum `value_in_cents(Coin::Quarter(UsState::Alaska))` ko call karte, toh `coin` `Coin::Quarter(UsState::Alaska)` hoga. Jab hum us value ko har match arm se compare karte hain, toh koi bhi match nahi karta jab tak hum `Coin::Quarter(state)` tak nahi pahunchte. Us point par, `state` ke liye binding `UsState::Alaska` value hogi. Phir hum us binding ko `println!` expression mein upyog kar sakte hain, is tarah `Quarter` ke liye `Coin` enum variant se inner state value nikal sakte hain.

-----

### Matching with Option<T>

Pichhle section mein, hum `Option<T>` ka upyog karte waqt `Some` case se inner `T` value nikalna chahte the; hum `Option<T>` ko `match` ka upyog karke bhi handle kar sakte hain jaise humne `Coin` enum ke saath kiya tha\! Coins ko compare karne ke bajaye, hum `Option<T>` ke variants ko compare karenge, lekin `match` expression ke kaam karne ka tarika wahi rehta hai.

Maan lijiye hum ek function likhna chahte hain jo ek `Option<i32>` leta hai aur, agar uske andar koi value hai, toh us value mein 1 jod deta hai. Agar uske andar koi value nahi hai, toh function ko `None` value return karni chahiye aur koi operation karne ki koshish nahi karni chahiye.

Yeh function `match` ki vajah se likhna bahut aasan hai, aur yeh Listing 6-5 jaisa dikhega.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

\<span class="caption"\>Listing 6-5: Ek function jo `Option<i32>` par ek `match` expression ka upyog karta hai\</span\>

Chaliye `plus_one` ke pehle execution ko aur detail mein dekhte hain. Jab hum `plus_one(five)` ko call karte hain, toh `plus_one` ki body mein `x` variable ki value `Some(5)` hogi. Phir hum usse har match arm se compare karte hain.

```rust,ignore
None => None,
```

`Some(5)` value `None` pattern se match nahi karti, isliye hum agle arm par continue karte hain.

```rust,ignore
Some(i) => Some(i + 1),
```

Kya `Some(5)` `Some(i)` se match karta hai? Haan bilkul\! Hamare paas wahi variant hai. `i` `Some` mein shamil value se bind hota hai, isliye `i` `5` value leta hai. Phir match arm mein code execute hota hai, isliye hum `i` ki value mein 1 jodte hain aur apne total `6` ke saath ek nayi `Some` value banate hain.

Ab chaliye Listing 6-5 mein `plus_one` ke doosre call ko consider karte hain, jahan `x` `None` hai. Hum `match` mein enter karte hain aur pehle arm se compare karte hain.

```rust,ignore
None => None,
```

Yeh match karta hai\! Jodne ke liye koi value nahi hai, isliye program ruk jata hai aur `=>` ke right side par `None` value return karta hai. Kyunki pehle arm ne match kiya, kisi aur arms ko compare nahi kiya jata.

`match` aur enums ko combine karna bahut si situations mein useful hai. Aap Rust code mein is pattern ko bahut dekhnge: ek enum ke khilaf `match` karna, andar ke data se ek variable ko bind karna, aur phir uske aadhar par code execute karna. Shuru mein yeh thoda mushkil ho sakta hai, lekin ek baar jab aap iske aadi ho jate hain, toh aap chahenge ki yeh sabhi languages mein ho. Yeh consistently user favorite hai.

-----

### Matches Are Exhaustive

`match` ka ek aur pehlu hai jise humein discuss karne ki zaroorat hai. `plus_one` function ke is version ko consider karein jismein ek bug hai aur yeh compile nahi hoga:

```rust,ignore
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```

Humne `None` case ko handle nahi kiya, isliye yeh code ek bug ka karan banega. Khushkismati se, yeh ek bug hai jise Rust pakadna janta hai. Agar hum is code ko compile karne ki koshish karte hain, toh humein yeh error milegi:

```text
error[E0004]: non-exhaustive patterns: `None` not covered
 -->
  |
6 |           match x {
  |                   ^ pattern `None` not covered
```

Rust janta hai ki humne har possible case ko cover nahi kiya aur yeh bhi janta hai ki hum kaun sa pattern bhool gaye\! Rust mein Matches **exhaustive** hote hain: code ko valid hone ke liye humein har aakhri possibility ko exhaust karna zaroori hai. Khas taur par `Option<T>` ke case mein, jab Rust humein explicitly `None` case ko handle karna bhoolne se rokta hai, toh yeh humein yeh manne se bachata hai ki hamare paas ek value hai jabki hamare paas null ho sakta hai, is tarah pehle discuss ki gayi billion-dollar mistake ko hone se rokta hai.

-----

### The _ Placeholder

Rust mein ek pattern bhi hai jise hum tab upyog kar sakte hain jab hum sabhi possible values ko list nahi karna chahte. Udaharan ke liye, ek `u8` ki valid values 0 se 255 tak ho sakti hain. Agar hum sirf 1, 3, 5, aur 7 values ki parwah karte hain, toh hum 0, 2, 4, 6, 8, 9 se lekar 255 tak sabhi ko list nahi karna chahenge. fortunately, humein aisa nahi karna padta: hum special pattern `_` ka upyog kar sakte hain:

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

`_` pattern kisi bhi value se match karega. Isse apne doosre arms ke baad rakhne se, `_` un sabhi possible cases se match karega jo isse pehle specify nahi kiye gaye hain. `()` bas unit value hai, isliye `_` case mein kuch nahi hoga. Natijan, hum keh sakte hain ki hum un sabhi possible values ke liye kuch nahi karna chahte jinhe hum `_` placeholder se pehle list nahi karte.

Lekin, `match` expression ek aisi situation mein thoda wordy ho sakta hai jismein hum sirf ek hi case ki parwah karte hain. Is situation ke liye, Rust **`if let`** provide karta hai.
