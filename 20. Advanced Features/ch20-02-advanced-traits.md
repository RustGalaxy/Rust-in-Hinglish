Bilkul, yahan Hinglish mein anuvaad hai:

## Advanced Traits

Humne pehli baar traits ke baare mein Chapter 10 ke "Traits: Defining Shared Behavior" mein cover kiya tha, lekin humne zyada advanced details discuss nahi ki thi. Ab jab aap Rust ke baare mein aur jaante hain, hum iski gehri baaton mein ja sakte hain.

-----

### Associated Types

**Associated types** ek type placeholder ko ek trait se jodte hain, taaki trait method definitions apne signatures mein in placeholder types ka istemal kar sakein. Trait ka implementor yeh batayega ki us particular implementation ke liye placeholder type ki jagah kaunsa concrete type istemal kiya jaayega. Is tarah, hum ek aisa trait define kar sakte hain jo kuch types ka istemal karta hai, bina yeh jaane ki woh types aakhir hain kya, jab tak ki trait implement na ho jaaye.

Humne is chapter mein zyada-tar advanced features ko aise bataya hai jinki zaroorat kam padti hai. Associated types beech mein kahin aate hain: yeh baaki kitaab mein samjhaye gaye features se kam istemal hote hain, lekin is chapter mein discuss kiye gaye kai doosre features se zyada aam hain.

Ek associated type wale trait ka ek example `Iterator` trait hai jo standard library provide karti hai. Associated type ka naam `Item` hai aur yeh un values ke type ke liye stand-in hai jin par `Iterator` trait implement karne wala type iterate kar raha hai. `Iterator` trait ki definition Listing 20-13 mein di gayi hai.

\<Listing number="20-13" caption="`Iterator` trait ki definition jismein ek associated type `Item` hai"\>

```rust,noplayground
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

\</Listing\>

`Item` type ek placeholder hai, aur `next` method ki definition dikhati hai ki yeh `$Option<Self::Item>$` type ki values return karega. `Iterator` trait ke implementors `Item` ke liye concrete type specify karenge, aur `next` method us concrete type ki value wala `Option` return karega.

Associated types generics jaisa concept lag sakta hai, kyunki generics bhi humein ek function define karne dete hain bina yeh bataye ki woh kaunse types handle kar sakta hai. Dono concepts ke beech ka fark samjhne ke liye, hum `Counter` naam ke ek type par `Iterator` trait ka ek implementation dekhenge jo specify karta hai ki `Item` type `u32` hai:

\<Listing file-name="src/lib.rs"\>

```rust,ignore
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // ...
    }
}
```

\</Listing\>

Yeh syntax generics ke syntax jaisa lagta hai. Toh kyun na `Iterator` trait ko generics ke saath define kiya jaaye, jaisa ki Listing 20-14 mein dikhaya gaya hai?

\<Listing number="20-14" caption="Generics ka istemal karke `Iterator` trait ki ek kalpanik definition"\>

```rust,noplayground
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

\</Listing\>

Fark yeh hai ki jab generics ka istemal karte hain, jaisa ki Listing 20-14 mein hai, humein har implementation mein types ko annotate karna padta hai; kyunki hum `Iterator<String> for Counter` ya koi aur type bhi implement kar sakte hain, `Counter` ke liye `Iterator` ke multiple implementations ho sakte hain. Doosre shabdon mein, jab ek trait mein generic parameter hota hai, toh use ek type ke liye kai baar implement kiya ja sakta hai, har baar generic type parameters ke concrete types ko badal kar. Jab hum `Counter` par `next` method ka istemal karte, toh humein type annotations deni padti yeh batane ke liye ki hum `Iterator` ka kaunsa implementation istemal karna chahte hain.

Associated types ke saath, humein types ko annotate karne ki zaroorat nahi hoti kyunki hum ek trait ko ek type par kai baar implement nahi kar sakte. Listing 20-13 mein, jahan definition associated types ka istemal karti hai, hum sirf ek baar chun sakte hain ki `Item` ka type kya hoga kyunki sirf ek hi `impl Iterator for Counter` ho sakta hai. Humein har jagah yeh batane ki zaroorat nahi hai ki humein `Counter` par `next` call karte samay `u32` values ka iterator chahiye.

-----

### Default Generic Type Parameters aur Operator Overloading

Jab hum generic type parameters ka istemal karte hain, toh hum generic type ke liye ek **default concrete type** specify kar sakte hain. Isse trait ke implementors ko concrete type specify karne ki zaroorat nahi padti agar default type kaam kar jaata hai. Aap generic type declare karte samay `<PlaceholderType=ConcreteType>` syntax ke saath ek default type specify karte hain.

Is technique ka ek behtareen example hai **operator overloading**, jismein aap kisi operator (jaise `+`) ke behavior ko khaas situations mein customize karte hain.

Rust aapko apne operators banane ya manmaane operators ko overload karne ki anumati nahi deta. Lekin aap `std::ops` mein listed operations aur unke corresponding traits ko implement karke overload kar sakte hain. Udaharan ke liye, Listing 20-15 mein hum `+` operator ko overload karte hain taaki do `Point` instances ko ek saath jod sakein. Hum yeh `Add` trait ko `Point` struct par implement karke karte hain.

\<Listing number="20-15" file-name="src/main.rs" caption="`Point` instances ke liye `+` operator ko overload karne ke liye `Add` trait ko implement karna"\>

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
```

\</Listing\>

`add` method do `Point` instances ke `x` values aur `y` values ko jodkar ek naya `Point` banata hai. `Add` trait mein `Output` naam ka ek associated type hai jo `add` method se return hone wale type ko determine karta hai.

Is code mein default generic type `Add` trait ke andar hai. Yahan uski definition hai:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Yeh code jaana-pehchana lagna chahiye: ek trait jismein ek method aur ek associated type hai. Naya hissa hai `Rhs=Self`: is syntax ko **default type parameters** kehte hain. `Rhs` generic type parameter ("right-hand side" ka short form) `add` method mein `rhs` parameter ka type define karta hai. Agar hum `Add` trait implement karte samay `Rhs` ke liye koi concrete type specify nahi karte, toh `Rhs` ka type `Self` par default ho jaayega, jo ki woh type hoga jis par hum `Add` implement kar rahe hain.

-----

### Ek Hi Naam Wale Methods ke Beech Antar Spasht Karna

Rust mein aisa kuch nahi hai jo ek trait ko kisi doosre trait ke method ke naam wala method rakhne se roke, na hi Rust aapko ek hi type par dono traits implement karne se rokta hai. Yeh bhi sambhav hai ki aap kisi type par seedhe ek method implement karein jiska naam traits ke methods jaisa hi ho.

Jab ek hi naam wale methods ko call karte hain, toh aapko Rust ko batana hoga ki aap kaunsa istemal karna chahte hain. Listing 20-17 ke code par gaur karein jahan humne do traits, `Pilot` aur `Wizard`, define kiye hain, jin dono mein `fly` naam ka ek method hai. Phir hum dono traits ko `Human` type par implement karte hain, jis par pehle se hi `fly` naam ka ek method implement hai. Har `fly` method kuch alag karta hai.

\<Listing number="20-17" file-name="src/main.rs" caption="Do traits define kiye gaye hain jinmein `fly` method hai aur `Human` type par implement kiye gaye hain, aur ek `fly` method `Human` par seedhe implement kiya gaya hai."\>

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
```

\</Listing\>

Jab hum ek `Human` ke instance par `fly` call karte hain, toh compiler us method ko call karne ke liye default karta hai jo seedhe us type par implement kiya gaya hai, jaisa ki Listing 20-18 mein dikhaya gaya hai.

\<Listing number="20-18" file-name="src/main.rs" caption="Ek `Human` ke instance par `fly` call karna"\>

```rust
fn main() {
    let person = Human;
    person.fly();
}
```

\</Listing\>

Is code ko chalane par `*waving arms furiously*` print hoga, jo dikhata hai ki Rust ne `Human` par seedhe implement kiye gaye `fly` method ko call kiya.

`Pilot` trait ya `Wizard` trait se `fly` methods ko call karne ke liye, humein zyada explicit syntax ka istemal karna hoga yeh batane ke liye ki humara matlab kaunse `fly` method se hai. Listing 20-19 is syntax ko darshata hai.

\<Listing number="20-19" file-name="src/main.rs" caption="Yeh batana ki hum kaunse trait ka `fly` method call karna chahte hain"\>

```rust
fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

\</Listing\>

Method ke naam se pehle trait ka naam batane se Rust ko spasht ho jaata hai ki hum `fly` ka kaunsa implementation call karna chahte hain.

Lekin, associated functions jo methods nahi hote, unmein `self` parameter nahi hota. Jab kai types ya traits hote hain jo ek hi naam ke non-method functions define karte hain, toh Rust hamesha nahi jaanta ki aapka matlab kaunse type se hai jab tak aap **fully qualified syntax** ka istemal na karein.

Humein Rust ko yeh batane ke liye ki hum `Dog` ke liye `Animal` ka implementation istemal karna chahte hain, na ki kisi aur type ke liye `Animal` ka implementation, humein fully qualified syntax ka istemal karna hoga. Listing 20-22 dikhata hai ki fully qualified syntax ka istemal kaise karein.

\<Listing number="20-22" file-name="src/main.rs" caption="Fully qualified syntax ka istemal karke yeh batana ki hum `Animal` trait se `baby_name` function ko call karna chahte hain jaisa ki `Dog` par implement kiya gaya hai"\>

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

\</Listing\>

Hum angle brackets ke andar Rust ko ek type annotation de rahe hain, jo yeh batata hai ki hum `Animal` trait se `baby_name` method ko call karna chahte hain jaisa ki `Dog` par implement kiya gaya hai, yeh keh kar ki hum is function call ke liye `Dog` type ko ek `Animal` ki tarah treat karna chahte hain. Ab yeh code wahi print karega jo hum chahte hain: `A baby dog is called a puppy`.

Aam taur par, fully qualified syntax is tarah se define kiya jaata hai:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Aapko yeh verbose syntax sirf unhi cases mein istemal karna hoga jahan ek hi naam ka istemal karne wale multiple implementations hote hain aur Rust ko yeh pehchanne mein madad chahiye ki aap kaunsa implementation call karna chahte hain.

-----

### Supertraits ka Istemal

Kabhi-kabhi aap ek aisi trait definition likh sakte hain jo doosre trait par nirbhar karti hai: ek type ke liye pehla trait implement karne ke liye, aap yeh zaroori banana chahte hain ki woh type doosra trait bhi implement kare. Aap aisa isliye karenge taaki aapki trait definition doosre trait ke associated items ka istemal kar sake. Jis trait par aapki trait definition nirbhar kar rahi hai, use aapke trait ka **supertrait** kaha jaata hai.

Udaharan ke liye, maan lijiye hum ek `OutlinePrint` trait banana chahte hain jismein ek `outline_print` method ho jo di gayi value ko is tarah format karke print karega ki woh asterisks (`*`) mein framed ho. Iske liye humein `Display` trait ki functionality ka istemal karna hoga. Isliye, humein yeh batana hoga ki `OutlinePrint` trait sirf unhi types ke liye kaam karega jo `Display` bhi implement karte hain. Hum yeh trait definition mein `OutlinePrint: Display` specify karke kar sakte hain. Listing 20-23 `OutlinePrint` trait ka ek implementation dikhata hai.

\<Listing number="20-23" file-name="src/main.rs" caption="`OutlinePrint` trait ko implement karna jo `Display` se functionality ki maang karta hai"\>

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}

struct Point {
    x: i32,
    y: i32,
}

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

impl OutlinePrint for Point {}

fn main() {
    let p = Point { x: 1, y: 3 };
    p.outline_print();
}
```

\</Listing\>

Agar hum `OutlinePrint` ko ek aise type par implement karne ki koshish karte jo `Display` implement nahi karta, toh humein ek error milega. Isko theek karne ke liye, hum us type par `Display` implement karte hain, taaki `OutlinePrint` ki zaroorat poori ho sake.

-----

### External Traits ko Implement Karne ke liye Newtype Pattern ka Istemal

Chapter 10 mein, humne **orphan rule** ka zikr kiya tha jo kehta hai ki hum ek trait ko ek type par tabhi implement kar sakte hain jab ya toh trait ya type, ya dono, hamare crate ke liye local hon. Is pabandi se bachne ke liye **newtype pattern** ka istemal kiya ja sakta hai, jismein ek tuple struct mein ek naya type banaya jaata hai.

Tuple struct mein ek field hoga aur yeh us type ke liye ek aasan sa wrapper hoga jis par hum trait implement karna chahte hain. Phir wrapper type hamare crate ke liye local ho jaata hai, aur hum wrapper par trait implement kar sakte hain. Is pattern ka istemal karne par koi runtime performance penalty nahi hoti, aur wrapper type compile time par hata diya jaata hai.

Ek example ke taur par, maan lijiye hum `Display` ko `Vec<T>` par implement karna chahte hain, jise orphan rule seedhe taur par rokta hai kyunki `Display` trait aur `Vec<T>` type dono hamare crate ke bahar define kiye gaye hain. Hum ek `Wrapper` struct bana sakte hain jo `Vec<T>` ka ek instance rakhta hai; phir hum `Wrapper` par `Display` implement kar sakte hain aur `Vec<T>` value ka istemal kar sakte hain, jaisa ki Listing 20-24 mein dikhaya gaya hai.

\<Listing number="20-24" file-name="src/main.rs" caption="`Display` ko implement karne ke liye `Vec&lt;String&gt;` ke aas-paas ek `Wrapper` type banana"\>

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

\</Listing\>

`Display` ka implementation `self.0` ka istemal inner `Vec<T>` ko access karne ke liye karta hai.

Is technique ka nuksaan yeh hai ki `Wrapper` ek naya type hai, isliye usmein us value ke methods nahi hote jise woh hold kar raha hai. Humein `Wrapper` par `Vec<T>` ke sabhi methods ko seedhe implement karna padega taaki methods `self.0` ko delegate karein. Agar hum chahte hain ki naye type mein inner type ke har method hon, toh `Wrapper` par `Deref` trait implement karna ek samadhan hoga.