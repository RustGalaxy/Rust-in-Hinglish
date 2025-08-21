# Traits: Shared Behavior ko Define Karna 🤝

Ek **trait** Rust compiler ko kisi particular type mein maujood functionality ke baare mein batata hai jise wo dusre types ke saath share kar sakta hai. Hum traits ka upyog **shared behavior** ko ek **abstract** tareeke se define karne ke liye kar sakte hain. Hum **trait bounds** ka upyog ye specify karne ke liye kar sakte hain ki ek generic kisi bhi type ka ho sakta hai jismein ek certain behavior ho.

> Note: Traits doosri languages mein aksar **interfaces** ke naam se jane jaate hain, par kuch differences ke saath.

-----

### Trait ko Define karna 📝

Ek type ka behavior un methods se banta hai jinhein hum us type par call kar sakte hain. Different types same behavior share karte hain agar hum un sabhi types par same methods call kar saken. **Trait definitions** ek tareeka hai jisse hum method signatures ko ek saath group kar sakte hain.

Jaise, humare paas `NewsArticle` aur `Tweet` jaise structs hain. Hum ek media aggregator library banana chahte hain jo `NewsArticle` ya `Tweet` instance mein stored data ki summary display kar sake. Iske liye, humein har type se ek summary chahiye, aur humein us summary ko ek `summarize` method call karke request karna hoga. **Listing 10-12** ek `Summary` trait ki definition dikhati hai jo is behavior ko express karta hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

\<span class="caption"\>Listing 10-12: Ek `Summary` trait jo `summarize` method dwara provide kiye gaye behavior se banta hai\</span\>

Yahan, hum **`trait` keyword** aur fir trait ka naam, jo is case mein `Summary` hai, use karke ek trait declare karte hain. Curly brackets ke andar, hum method signatures declare karte hain jo un types ke behaviors ko describe karte hain jo is trait ko implement karenge. Is case mein, ye `fn summarize(&self) -> String` hai.

Method signature ke baad, curly brackets ke andar implementation dene ke bajaye, hum semicolon ka upyog karte hain. Is trait ko implement karne wale har type ko method body ke liye apna khud ka custom behavior dena hoga. Compiler **enforce** karega ki jo bhi type `Summary` trait rakhta hai, usmein `summarize` method isi signature ke saath defined hoga.

-----

### Trait ko Type par Implement karna ✍️

Ab jab humne `Summary` trait ka use karke desired behavior define kar liya hai, hum ise hamare media aggregator ke types par implement kar sakte hain. **Listing 10-13** `NewsArticle` struct par `Summary` trait ka ek implementation dikhati hai jo `summarize` ki return value banane ke liye headline, author, aur location ka upyog karta hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# pub trait Summary {
#     fn summarize(&self) -> String;
# }
#
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

\<span class="caption"\>Listing 10-13: `Summary` trait ko `NewsArticle` aur `Tweet` types par implement karna\</span\>

Ek type par trait implement karna regular methods implement karne jaisa hi hai. Fark ye hai ki `impl` ke baad, hum us trait ka naam likhte hain jise hum implement karna chahte hain, fir **`for` keyword** ka upyog karte hain, aur fir us type ka naam specify karte hain jis par hum trait implement karna chahte hain.

Trait implement karne ke baad, hum `NewsArticle` aur `Tweet` ke instances par methods ko usi tarah call kar sakte hain jaise hum regular methods ko call karte hain.

```rust,ignore
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summarize());
```

`1 new tweet: horse_ebooks: of course, as you probably already know, people` print hoga.

Ek restriction ye hai ki hum ek trait ko ek type par tabhi implement kar sakte hain jab ya to wo **trait ya wo type hamari crate ke andar ho**. For example, hum `Display` trait ko `Tweet` par implement kar sakte hain, kyunki `Tweet` hamare `aggregator` crate ke andar hai. Lekin hum `Display` trait ko `Vec<T>` par hamare `aggregator` crate ke andar implement nahi kar sakte, kyunki `Display` aur `Vec<T>` dono standard library mein define hain aur hamari `aggregator` crate mein local nahi hain. Is restriction ko **coherence** kehte hain, aur specifically **orphan rule** kehte hain.

-----

### Default Implementations 🧰

Kabhi-kabhi, ek trait mein kuch ya sabhi methods ke liye **default behavior** dena useful hota hai. Tab, jab hum trait ko kisi particular type par implement karte hain, to hum har method ke default behavior ko rakh sakte hain ya override kar sakte hain.

**Listing 10-14** `Summary` trait ke `summarize` method ke liye ek default string specify karna dikhati hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

\<span class="caption"\>Listing 10-14: `Summary` trait ki definition jismein `summarize` method ki default implementation hai\</span\>

Default implementation ka upyog karne ke liye, hum `impl Summary for NewsArticle {}` ke saath ek khali `impl` block specify karte hain.

Isse `NewsArticle` ke instance par `summarize` method ko call kiya ja sakta hai, jaise:

```rust,ignore
let article = NewsArticle {
    headline: String::from("Penguins win the Stanley Cup Championship!"),
    location: String::from("Pittsburgh, PA, USA"),
    author: String::from("Iceburgh"),
    content: String::from("The Pittsburgh Penguins once again are the best
    hockey team in the NHL."),
};

println!("New article available! {}", article.summarize());
```

Ye code `New article available! (Read more...)` print karega.

-----

### Trait Bounds 📐

Ab jab aap jaante hain ki traits ko kaise define aur implement kiya jaata hai, hum generic type parameters ke saath traits ka upyog karna explore kar sakte hain. Hum **trait bounds** ka upyog generic types ko constrain karne ke liye kar sakte hain taaki type unhi tak limited rahe jismein ek particular behavior ho.

**Listing 10-13** mein, humne `NewsArticle` aur `Tweet` types par `Summary` trait implement kiya tha. Hum ek `notify` function define kar sakte hain jo apne parameter `item` par `summarize` method ko call karta hai, jo generic type `T` ka hai. `summarize` ko `item` par call karne ke liye, hum `T` par trait bounds ka upyog kar sakte hain.

```rust,ignore
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
```

Hum **generic type parameter ke declaration ke saath, colon ke baad aur angle brackets ke andar** trait bounds rakhte hain.

Hum `+` syntax ka upyog karke generic type par multiple trait bounds specify kar sakte hain. For example, `T: Summary + Display`.

Agar bahut saare trait bounds ho, to function signature padhna mushkil ho jaata hai. Isliye, Rust ke paas **`where` clause** ka alternative syntax hai.

```rust,ignore
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

Ye function signature kam cluttered hai.

-----

### Trait Bounds se `largest` Function ko Fix karna 🛠️

Hum `largest` function ko fix karne ke liye **Listing 10-5** par wapas jaate hain. Hum `largest` ki body mein `>` operator ka use karke do `T` type ki values ko compare karna chahte the. Kyunki ye operator `std::cmp::PartialOrd` trait par defined hai, humein `T` ke liye trait bounds mein `PartialOrd` specify karne ki zaroorat hai.

Is baar jab hum compile karenge, to humein ek alag error milegi: `cannot move out of type [T], a non-copy slice`. `i32` aur `char` jaise types `Copy` trait ko implement karte hain. Lekin jab humne `largest` function ko generic banaya, to `list` parameter mein aise types hona possible ho gaya jo `Copy` trait ko implement nahi karte.

Isliye, humein `T` ke trait bounds mein `Copy` add karna hoga. **Listing 10-15** ek generic `largest` function ka complete code dikhati hai jo compile ho jaayega.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
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

\<span class="caption"\>Listing 10-15: `largest` function ki ek working definition jo kisi bhi generic type par kaam karti hai jo `PartialOrd` aur `Copy` traits ko implement karta hai\</span\>

-----

### Trait Bounds ka Upyog Karke Conditionally Methods Implement karna 🎯

`impl` block ke saath trait bound ka upyog karke, hum types ke liye **conditionally methods** implement kar sakte hain jo specified traits ko implement karte hain. For example, **Listing 10-16** mein `Pair<T>` type hamesha `new` function implement karta hai. Lekin `Pair<T>` sirf `cmp_display` method ko tabhi implement karta hai jab uska inner type `T` **`PartialOrd`** aur **`Display`** traits ko implement karta hai.

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

\<span class="caption"\>Listing 10-16: Trait bounds ke aadhar par generic type par conditionally methods implement karna\</span\>

Hum ek trait ko kisi bhi type ke liye conditionally implement kar sakte hain jo doosre trait ko implement karta hai. Trait implementations jo trait bounds ko satisfy karne wale kisi bhi type par lagoo hoti hain unhein **blanket implementations** kehte hain. For example, standard library `ToString` trait ko kisi bhi type par implement karta hai jo `Display` trait ko implement karta hai. Is wajah se, hum `to_string` method ko kisi bhi aise type par call kar sakte hain jo `Display` implement karta hai.

Traits aur trait bounds humein generic type parameters ka upyog karke duplication ko kam karne wala code likhne dete hain, lekin compiler ko ye bhi specify karte hain ki hum chahte hain ki generic type mein particular behavior ho. Isse compiler check kar sakta hai ki hamare code ke saath use kiye gaye saare concrete types sahi behavior provide karte hain. Isse **compile time** par hi errors pakdi jaati hain aur **performance** improve hoti hai.

Agla generic type **lifetimes** hai, jo references ko valid rakhne ka kaam karte hain.
