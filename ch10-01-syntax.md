# Generic Data Types ⚙️

Hum generics ka upyog functions aur structs jaise items ke liye definitions banane ke liye kar sakte hain, jise hum kai alag-alag **concrete data types** ke saath use kar sakte hain. Aaiye dekhte hain ki generics ka upyog karke functions, structs, enums aur methods ko kaise define kiya jaata hai. Phir hum discuss karenge ki generics code ki performance ko kaise affect karte hain.

-----

### Function Definitions mein 📝

Jab ek function define karte hain jo **generics** ka upyog karta hai, to hum generics ko function ke signature mein wahan rakhte hain jahan hum aam taur par parameters aur return value ke data types specify karte hain. Aisa karne se hamara code zyaada flexible banta hai aur hamare function ke callers ko zyaada functionality milti hai, jabki code duplication bhi ruk jaati hai.

Hamare `largest` function ko aage badhate hue, **Listing 10-4** do functions dikhati hain jo dono ek slice mein sabse badi value dhoondhti hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);
#   assert_eq!(result, 100);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
#   assert_eq!(result, 'y');
}
```

\<span class="caption"\>Listing 10-4: Do functions jinke naam aur signature ke types hi alag hain\</span\>

`largest_i32` function ek slice mein sabse bada `i32` dhoondhta hai, jabki `largest_char` function ek slice mein sabse bada `char` dhoondhta hai. Dono functions ki bodies mein same code hai, to aaiye ek single function mein generic type parameter introduce karke is duplication ko eliminate karte hain.

Naye function mein types ko parameterize karne ke liye, humein type parameter ka naam dena hoga, jaisa ki hum function ke value parameters ke liye karte hain. Aap type parameter naam ke roop mein koi bhi identifier use kar sakte hain. Lekin hum `T` ka upyog karenge, kyunki convention ke mutabik, Rust mein parameter names chote hote hain, aksar sirf ek letter, aur Rust ki type-naming convention **CamelCase** hai. "Type" ke liye short, `T` zyadatar Rust programmers ka default choice hai.

Jab hum ek function ki body mein ek parameter ka upyog karte hain, to humein signature mein parameter ka naam declare karna hota hai taaki compiler ko pata chale ki us naam ka kya matlab hai. Isi tarah, jab hum ek function signature mein type parameter ka naam use karte hain, to humein use use karne se pehle declare karna hota hai. Generic `largest` function ko define karne ke liye, type name declarations ko **angle brackets (`<>`)** ke andar function ke naam aur parameter list ke beech rakhein, is tarah:

```rust,ignore
fn largest<T>(list: &[T]) -> T {
```

Hum is definition ko aise padhte hain: `largest` function kisi type `T` par generic hai. Is function mein `list` naam ka ek parameter hai, jo type `T` ki values ka ek slice hai. `largest` function usi type `T` ki ek value return karega.

**Listing 10-5** generic data type ka upyog karke `largest` function definition dikhati hai. Isme ye bhi dikhaya gaya hai ki hum is function ko `i32` values ya `char` values ke slice ke saath kaise call kar sakte hain. Note karein ki ye code abhi compile nahi hoga, lekin hum ise is chapter mein aage fix karenge.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

\<span class="caption"\>Listing 10-5: `largest` function ki ek definition jo generic type parameters ka upyog karti hai, lekin abhi compile nahi hogi\</span\>

Agar hum is code ko abhi compile karte hain, to humein ye error milegi:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:5:12
  |
5 |         if item > largest {
  |            ^^^^^^^^^^^^^^
  |
  = note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

Ye note `std::cmp::PartialOrd` ka zikr karta hai, jo ek **trait** hai. Hum aage **"Trait Bounds"** section mein discuss karenge. Filhaal, ye error batata hai ki `largest` ki body saare possible types ke liye kaam nahi karegi jo `T` ho sakta hai. Kyunki hum body mein `T` type ki values ko compare karna chahte hain, hum sirf un types ka upyog kar sakte hain jinki values ko order kiya ja sakta hai.

-----

### Struct Definitions mein 🗃️

Hum structs ko bhi ek ya ek se zyaada fields mein generic type parameter ka upyog karne ke liye define kar sakte hain. **Listing 10-6** dikhati hai ki kaise ek `Point<T>` struct ko kisi bhi type ke `x` aur `y` coordinate values ko hold karne ke liye define kiya jaata hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

\<span class="caption"\>Listing 10-6: Ek `Point<T>` struct jo `T` type ki `x` aur `y` values ko hold karta hai\</span\>

Struct definitions mein generics ka upyog karne ka syntax function definitions mein use kiye gaye syntax ke jaisa hi hai. Sabse pehle, hum struct ke naam ke theek baad angle brackets ke andar type parameter ka naam declare karte hain. Phir hum struct definition mein generic type ka upyog kar sakte hain jahan hum aam taur par concrete data types specify karte.

Gaur karein ki kyunki humne `Point<T>` ko define karne ke liye sirf ek hi generic type ka upyog kiya hai, to ye definition kehti hai ki `Point<T>` struct kisi type `T` par generic hai, aur `x` aur `y` fields dono usi same type ke hain, jo bhi wo type ho. Agar hum ek `Point<T>` ka instance banate hain jismein alag-alag types ki values hain, jaisa ki **Listing 10-7** mein hai, to hamara code compile nahi hoga.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

\<span class="caption"\>Listing 10-7: Fields `x` aur `y` ka type same hona chahiye kyunki dono ka generic data type `T` hai\</span\>

Is example mein, jab hum integer value 5 ko `x` ko assign karte hain, to hum compiler ko batate hain ki `Point<T>` ke is instance ke liye generic type `T` ek integer hoga. Phir jab hum `y` ke liye 4.0 specify karte hain, jise humne `x` ke jaisa hi type define kiya hai, to humein type mismatch error milegi.

Ek `Point` struct define karne ke liye jahan `x` aur `y` dono generics hain lekin unke types alag-alag ho sakte hain, hum **multiple generic type parameters** ka upyog kar sakte hain. For example, **Listing 10-8** mein, hum `Point` ki definition ko `T` aur `U` types par generic banane ke liye badal sakte hain jahan `x` type `T` ka hai aur `y` type `U` ka hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

\<span class="caption"\>Listing 10-8: Ek `Point<T, U>` do types par generic hai, taaki `x` aur `y` alag-alag types ki values ho saken\</span\>

Ab `Point` ke saare instances allowed hain\! Aap ek definition mein jitne chahe utne generic type parameters ka upyog kar sakte hain, lekin kuch se zyaada use karna aapke code ko padhna mushkil bana deta hai.

-----

### Enum Definitions mein 📥

Hum structs ki tarah, enums ko bhi unke variants mein generic data types ko hold karne ke liye define kar sakte hain. Standard library ke `Option<T>` enum par ek aur nazar daalte hain, jise humne **Chapter 6** mein use kiya tha:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Ye definition ab aapko zyaada samajh mein aani chahiye. Jaisa ki aap dekh sakte hain, `Option<T>` ek enum hai jo type `T` par generic hai aur uske do variants hain: `Some`, jo `T` type ki ek value ko hold karta hai, aur `None` variant jo koi value hold nahi karta. `Option<T>` enum ka upyog karke, hum ek optional value hone ke abstract concept ko express kar sakte hain, aur kyunki `Option<T>` generic hai, hum is abstraction ka upyog kar sakte hain, bhale hi optional value ka type kuch bhi ho.

-----

### Method Definitions mein 🛠️

Hum structs aur enums par methods ko implement kar sakte hain (jaisa ki humne **Chapter 5** mein kiya tha) aur unki definitions mein generic types ka bhi upyog kar sakte hain. **Listing 10-9** `Point<T>` struct dikhati hai jise humne **Listing 10-6** mein define kiya tha, us par `x` naam ka ek method implement kiya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

\<span class="caption"\>Listing 10-9: `Point<T>` struct par `x` naam ka ek method implement karna jo `T` type ke `x` field ka reference return karega\</span\>

Yahan, humne `Point<T>` par `x` naam ka ek method define kiya hai jo `x` field ke data ka reference return karta hai.

Gaur karein ki humein `impl` ke theek baad `T` ko declare karna padta hai taaki hum iska upyog ye specify karne ke liye kar sakein ki hum `Point<T>` type par methods implement kar rahe hain. `impl` ke baad `T` ko ek generic type ke roop mein declare karke, Rust ye pehchan sakta hai ki `Point` ke angle brackets mein type ek generic type hai na ki ek concrete type.

For example, hum sirf `Point<f32>` instances par methods implement kar sakte hain, na ki kisi bhi generic type ke saath `Point<T>` instances par. **Listing 10-10** mein hum concrete type `f32` ka upyog karte hain, matlab hum `impl` ke baad koi types declare nahi karte hain.

```rust
# struct Point<T> {
#     x: T,
#     y: T,
# }
#
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

\<span class="caption"\>Listing 10-10: Ek `impl` block jo sirf ek struct par lagoo hota hai jiske generic type parameter `T` ke liye ek particular concrete type hai\</span\>

Is code ka matlab hai ki `Point<f32>` type mein `distance_from_origin` naam ka ek method hoga aur `Point<T>` ke doosre instances jahan `T` `f32` type ka nahi hai, unmein ye method defined nahi hoga.

-----

### Generics ka Upyog Karne Wale Code ki Performance ⚡

Aap soch sakte hain ki generic type parameters ka upyog karte samay koi **runtime cost** lagta hai ya nahi. Achi khabar ye hai ki Rust generics ko is tarah se implement karta hai ki aapka code generic types ka upyog karke utna slow nahi chalta jitna wo concrete types ke saath chalta.

Rust ye **compile time** par code ke **monomorphization** ko perform karke achieve karta hai. **Monomorphization** generic code ko compile hone par use hone wale concrete types ko fill karke specific code mein badalne ka process hai.

Is process mein, compiler un saari jagahon ko dekhta hai jahan generic code ko call kiya gaya hai aur generic code ko jis concrete types ke saath call kiya gaya hai unke liye code generate karta hai.

Aaiye ek example dekhte hain jo standard library ke `Option<T>` enum ka upyog karta hai:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Jab Rust is code ko compile karta hai, to ye **monomorphization** perform karta hai. Is process ke dauran, compiler `Option<T>` instances mein use kiye gaye values ko padhta hai aur do tarah ke `Option<T>` ko identify karta hai: ek `i32` hai aur doosra `f64`. Is tarah, ye generic definition ko `Option_i32` aur `Option_f64` mein expand karta hai.

Code ka monomorphized version is tarah dikhta hai. Generic `Option<T>` ko compiler dwara banayi gayi specific definitions se replace kar diya jaata hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

Kyunki Rust generic code ko us code mein compile karta hai jo har instance mein type ko specify karta hai, to generics ka upyog karne ke liye humein koi **runtime cost** nahi lagta. Jab code run hota hai, to ye bilkul waise hi perform karta hai jaise humne har definition ko haath se duplicate kiya hota. Monomorphization ka process Rust ke generics ko runtime par extremely efficient banata hai.
