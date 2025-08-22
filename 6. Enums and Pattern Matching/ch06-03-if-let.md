## `Concise Control Flow with if let` (if let se Sankshipt Control Flow)

**`if let`** syntax aapko `if` aur `let` ko ek kam verbose tarike se jodkar un values ko handle karne deta hai jo ek pattern se match karte hain jabki baaki ko ignore kar diya jata hai. Listing 6-6 mein diye gaye program ko dekhein jo ek `Option<u8>` value par match karta hai lekin sirf tab code execute karna chahta hai jab value `3` ho.

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

\<span class="caption"\>Listing 6-6: Ek `match` jo sirf tab code execute karne ki parwah karta hai jab value `Some(3)` ho\</span\>

Hum `Some(3)` match ke saath kuch karna chahte hain lekin kisi aur `Some<u8>` value ya `None` value ke saath kuch bhi nahi karna chahte. `match` expression ko satisfy karne ke liye, humein sirf ek variant ko process karne ke baad `_ => ()` jodna padta hai, jo ki bahut zyada boilerplate code hai.

Iske bajaye, hum `if let` ka upyog karke ise ek chhote tarike se likh sakte hain. Neeche diya gaya code Listing 6-6 mein `match` jaisa hi behave karta hai:

```rust
# let some_u8_value = Some(0u8);
if let Some(3) = some_u8_value {
    println!("three");
}
```

**`if let`** syntax ek pattern aur ek equal sign se separate kiye gaye ek expression ko leta hai. Yeh `match` jaisa hi kaam karta hai, jahan expression ko `match` ko diya jata hai aur pattern uska pehla arm hai.

`if let` ka upyog karne ka matlab hai kam typing, kam indentation, aur kam boilerplate code. Lekin, aap **exhaustive checking** kho dete hain jo `match` enforce karta hai. `match` aur `if let` ke beech chunav aapki particular situation par depend karta hai ki kya brevity (sankshiptata) prapt karna exhaustive checking khone ke liye ek appropriate trade-off hai.

Dusre shabdon mein, aap **`if let`** ko ek `match` ke liye syntax sugar maan sakte hain jo tab code chalata hai jab value ek pattern se match karti hai aur phir doosri sabhi values ko ignore kar deta hai.

Hum `if let` ke saath ek `else` bhi shamil kar sakte hain. Jo code block `else` ke saath jata hai, woh wahi code block hota hai jo us `match` expression mein `_` case ke saath jata jiske barabar `if let` aur `else` hai. `Coin` enum definition ko yaad karein Listing 6-4 mein, jahan `Quarter` variant ek `UsState` value bhi hold karta tha. Agar hum quarters ke state ka naam batate hue sabhi non-quarter coins ko ginnana chahte hain, toh hum aisa ek `match` expression ke saath kar sakte hain jaise is tarah:

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
# let coin = Coin::Penny;
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
```

Ya hum is tarah se ek `if let` aur `else` expression ka upyog kar sakte hain:

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
# let coin = Coin::Penny;
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

Agar aapke paas aisi situation hai jismein aapke program mein logic `match` ka upyog karke express karne ke liye bahut verbose hai, toh yaad rakhein ki `if let` bhi aapke Rust toolbox mein hai.

-----

### `Summary` (Saraansh)

Humne ab cover kar liya hai ki custom types banane ke liye enums ka upyog kaise kiya jata hai jo enumerated values ke ek set mein se ek ho sakte hain. Humne dikhaya hai ki standard library ka `Option<T>` type aapko type system ka upyog karke errors ko rokne mein kaise madad karta hai. Jab enum values ke andar data hota hai, toh aap `match` ya `if let` ka upyog un values ko nikalne aur upyog karne ke liye kar sakte hain, is baat par depend karte hue ki aapko kitne cases handle karne ki zaroorat hai.

Ab aapke Rust programs aapke domain mein concepts ko structs aur enums ka upyog karke express kar sakte hain. Apne API mein upyog karne ke liye custom types banana type safety ensure karta hai: compiler yeh certain karega ki aapke functions ko sirf wahi values milein jo har function expect karta hai.

Apne users ko ek well-organized API provide karne ke liye jo upyog karne mein aasan ho aur sirf wahi expose kare jo aapke users ko zaroorat hogi, chaliye ab Rust ke modules par chalte hain.
