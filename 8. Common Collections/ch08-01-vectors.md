# Vectors se Values ki Lists Store karna 

Sabse pehle, hum **`Vec<T>`**, jise **vector** bhi kehte hain, ko dekhenge. Vectors aapko ek hi data structure mein ek se zyaada values store karne ki suvidha dete hain, jo saari values ko memory mein ek doosre ke bagal mein rakhta hai. Vectors sirf **ek hi type ki values** store kar sakte hain. Ye tab useful hote hain jab aapke paas items ki list ho, jaise ki ek file mein text ki lines ya shopping cart mein items ke daam.

-----

### Naya Vector Banana 🏗️

Ek naya, khali vector banane ke liye, hum **`Vec::new`** function ko call karte hain, jaisa **Listing 8-1** mein dikhaya gaya hai.

```rust
let v: Vec<i32> = Vec::new();
```

\<span class="caption"\>Listing 8-1: `i32` type ki values ko hold karne ke liye ek naya, khali vector banaana\</span\>

Yahan humne ek type annotation add kiya hai. Kyunki hum is vector mein koi value nahi daal rahe hain, Rust ko pata nahi chalta ki hum kis tarah ke elements store karna chahte hain. Vectors generics ka use karke implement kiye jaate hain. Filhaal, bas itna jaaniye ki standard library ka `Vec<T>` type kisi bhi type ko hold kar sakta hai, aur jab ek specific vector ek specific type ko hold karta hai, to us type ko angle brackets ke andar specify kiya jaata hai. **Listing 8-1** mein, humne Rust ko bataya hai ki `v` mein `Vec<i32>` `i32` type ke elements hold karega.

Zyaada realistic code mein, jab aap vectors mein values daalte hain, to Rust aksar type ka anumaan (infer) laga leta hai, isliye type annotation ki zaroorat kam hi padti hai.

Initial values ke saath ek `Vec<T>` banana zyaada common hai, aur is suvidha ke liye Rust **`vec!` macro** provide karta hai. Ye macro aapke diye gaye values ko lekar ek naya vector banata hai. **Listing 8-2** mein, hum ek naya `Vec<i32>` bana rahe hain jismein `1`, `2`, aur `3` values hain.

```rust
let v = vec![1, 2, 3];
```

\<span class="caption"\>Listing 8-2: Values ke saath ek naya vector banaana\</span\>

Yahan humne initial `i32` values di hain, isliye Rust `v` ka type `Vec<i32>` infer kar sakta hai, aur type annotation ki zaroorat nahi hai. Ab hum dekhte hain ki vector ko update kaise karte hain.

-----

### Vector ko Update karna ✍️

Ek vector banaane aur fir usme elements add karne ke liye, hum **`push` method** ka use kar sakte hain, jaisa **Listing 8-3** mein dikhaya gaya hai.

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

\<span class="caption"\>Listing 8-3: `push` method ka upyog karke vector mein values add karna\</span\>

Kisi bhi variable ki tarah, agar hum uski value ko change karna chahte hain, to humein usko **`mut` keyword** ka use karke mutable banana hoga. Jo numbers hum daal rahe hain, wo sab `i32` type ke hain, aur Rust data se iska anumaan laga leta hai, isliye `Vec<i32>` annotation ki zaroorat nahi hai.

-----

### Vector ke Drop Hone par Uske Elements bhi Drop hote hain 🗑️

Kisi bhi doosre `struct` ki tarah, ek vector jab scope se bahar chala jaata hai, to wo free ho jaata hai, jaisa **Listing 8-4** mein dikhaya gaya hai.

```rust
{
    let v = vec![1, 2, 3, 4];

    // do stuff with v

} // <- v scope se bahar chala jaata hai aur yahan free ho jaata hai
```

\<span class="caption"\>Listing 8-4: Dikhana ki vector aur uske elements kahan drop hote hain\</span\>

Jab vector drop hota hai, to uske andar ke saare contents bhi drop ho jaate hain, matlab jin integers ko wo hold kar raha tha wo clean up ho jaayenge. Ye baat sidhi-sadi lag sakti hai, lekin jab aap vector ke elements ke **references** ko introduce karte hain to ye thoda complicated ho jaata hai. Aage dekhte hain\!

-----

### Vectors ke Elements ko Padhna 👀

Ab jab aapko vectors banana, update karna aur destroy karna aa gaya hai, to unke contents ko padhna agla kadam hai. Vector mein stored value ko reference karne ke do tareeke hain. **Listing 8-5** mein, dono methods dikhaye gaye hain: **indexing syntax** ya **`get` method**.

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
let third: Option<&i32> = v.get(2);
```

\<span class="caption"\>Listing 8-5: Indexing syntax ya `get` method ka upyog karke vector mein item ko access karna\</span\>

Yahan do details par dhyaan dein. Pehle, hum teesre element ko paane ke liye **index `2`** ka use karte hain, kyunki vectors mein indexing zero se shuru hoti hai. Doosra, teesre element ko paane ke do tareeke hain: **`&` aur `[]` ka upyog karna**, jo humein ek **reference** deta hai, ya fir **`get` method** ka use karna, jo humein ek **`Option<&T>`** deta hai.

Rust mein elements ko reference karne ke do tareeke isliye hain taaki aap chun saken ki aapka program kaise behave karega jab aap ek aise index ka use karte hain jiska vector mein koi element nahi hai. Jaise, **Listing 8-6** mein, ek program ek vector jismein paanch elements hain, mein index **100** ko access karne ki koshish karta hai.

```rust,should_panic
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

\<span class="caption"\>Listing 8-6: Paanch elements wale vector mein index 100 par element ko access karne ki koshish\</span\>

Jab hum is code ko run karte hain, to **`[]` method** program ko **panic** kar dega, kyunki ye ek aise element ko reference karta hai jo exist hi nahi karta. Is method ka upyog tab sabse accha hai jab aap chahte hain ki agar koi vector ke end ke baad ke element ko access karne ki koshish kare to aapka program crash ho jaaye.

Jab `get` method ko ek aisa index diya jaata hai jo vector ke bahar hai, to ye **panic hone ki bajaye `None` return karta hai**. Aap is method ka upyog tab karenge jab vector ki range ke bahar ka element access karna normal circumstances mein kabhi-kabhi ho sakta hai. Aapke code mein fir **`Some(&element)`** ya **`None`** ko handle karne ka logic hoga.

Jab program ke paas ek valid reference hota hai, to **borrow checker** ownership aur borrowing rules (jiske baare mein Chapter 4 mein bataya gaya hai) ko enforce karta hai taaki ye reference aur vector ke contents ke liye koi bhi doosra reference valid rahe. **Listing 8-7** mein, hum vector ke pehle element ka ek immutable reference hold karte hain aur fir end mein ek element add karne ki koshish karte hain, jo kaam nahi karega.

```rust,ignore
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);
```

\<span class="caption"\>Listing 8-7: Ek item ka reference hold karte hue vector mein ek element add karne ki koshish\</span\>

Is code ko compile karne par ye error aayegi:

```text
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 -->
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^ mutable borrow occurs here
7 |
8 | }
  | - immutable borrow ends here
```

Ye error isliye aati hai, kyunki ek naya element add karne ke liye vector ko naya memory allocate karni pad sakti hai aur purane elements ko wahan copy karna pad sakta hai. Aise mein, pehle element ka reference deallocated memory ko point karega. Borrowing rules programs ko is situation mein aane se rokta hai.

-----

### Vector ke Values par Iterate karna 🔁

Agar hum vector ke har element ko ek-ek karke access karna chahte hain, to hum indexes ka use karne ki bajaye saare elements par iterate kar sakte hain. **Listing 8-8** mein dikhaya gaya hai ki ek `for` loop ka use karke `i32` values ke vector mein har element ka **immutable reference** kaise paate hain aur use print karte hain.

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```

\<span class="caption"\>Listing 8-8: `for` loop ka use karke vector ke har element ko iterate karke print karna\</span\>

Hum **mutable vector** ke har element par **mutable references** ke through iterate karke unhein change bhi kar sakte hain. **Listing 8-9** mein `for` loop har element mein `50` add karega.

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

\<span class="caption"\>Listing 8-9: Vector ke elements par mutable references ke through iterate karna\</span\>

Mutable reference jis value ko refer karta hai, use change karne ke liye, humein **dereference operator (`*`)** ka use karna padta hai taaki hum `+=` operator ka use karne se pehle `i` mein value tak pahunch sakein.

-----

### Multiple Types Store karne ke liye Enum ka Upyog 📦

Is chapter ki shuruat mein humne kaha tha ki vectors sirf **ek hi type ki values** store kar sakte hain. Ye thoda asuvidhaajanak ho sakta hai, kyunki alag-alag types ke items ki list store karne ki zaroorat pad sakti hai. Khushkismati se, ek enum ke variants ko ek hi **enum type** ke andar define kiya jaata hai, to jab humein ek vector mein alag-alag type ke elements store karne ki zaroorat hoti hai, to hum ek enum define aur use kar sakte hain\!

For example, maan lijiye hum spreadsheet ki ek row se values lena chahte hain jismein kuch columns mein integers hain, kuch mein floating-point numbers, aur kuch mein strings. Hum ek enum define kar sakte hain jiske variants alag-alag value types ko hold karenge, aur fir saare enum variants ko ek hi type ka mana jaayega: us enum ka type. Fir hum ek vector bana sakte hain jo us enum ko hold karega, aur is tarah se, alag-alag types ko hold karega. Humne ise **Listing 8-10** mein dikhaya hai.

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

\<span class="caption"\>Listing 8-10: Ek `enum` ko define karna taaki ek hi vector mein alag-alag types ki values store ki ja sakein\</span\>

Rust ko compile time par ye jaanna zaroori hota hai ki vector mein kis tarah ke types honge, taki use pata ho ki har element ko store karne ke liye heap par kitni memory ki zaroorat padegi. Dusra fayda ye hai ki hum explicitly bata sakte hain ki is vector mein kaun se types allowed hain. Agar Rust ek vector ko kisi bhi type ko hold karne deta, to chances the ki ek ya ek se zyaada types vector ke elements par perform kiye jaane wale operations se errors cause kar sakte the. Enum ka use karne ke saath `match` expression ka use karne se Rust compile time par ensure karega ki har possible case ko handle kiya gaya hai.

Agar aap ek program likh rahe hain aur aapko run time par ye nahi pata ki vector mein kaun se exhaustive set of types store kiye jaayenge, to enum technique kaam nahi karegi. Iski jagah, aap **trait object** ka use kar sakte hain, jise hum **Chapter 17** mein cover karenge.

Ab jab humne vectors ko use karne ke kuch sabse common tareeke discuss kar liye hain, to `Vec<T>` par standard library dwara define kiye gaye aur bhi useful methods ke liye API documentation ko zaroor dekhein. Jaise, `push` ke alawa, `pop` method last element ko remove karke return karta hai. Chaliye ab agle collection type par chalte hain: **`String`**\!
