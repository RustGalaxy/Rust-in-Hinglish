## `Deref` Trait ke Saath Smart Pointers ko Regular References ki Tarah Treat Karna

`Deref` trait ko implement karke aap ***dereference operator***, `*` (multiplication ya glob operator se alag) ke behavior ko customize kar sakte hain. Jab aap `Deref` ko is tarah implement karte hain ki ek smart pointer ko ek regular reference ki tarah treat kiya ja sake, to aap aisa code likh sakte hain jo references par kaam karta hai aur us code ko smart pointers ke saath bhi istemaal kar sakte hain.

Chaliye pehle dekhte hain ki dereference operator regular references ke saath kaise kaam karta hai. Phir hum ek custom type banane ki koshish karenge jo `Box<T>` ki tarah behave kare, aur dekhenge ki dereference operator hamare naye type par reference ki tarah kaam kyun nahi karta. Hum explore karenge ki `Deref` trait ko implement karna kaise smart pointers ko references ki tarah kaam karne mein saksham banata hai. Phir hum Rust ke **deref coercion** feature ko dekhenge aur yeh kaise humein references ya smart pointers dono ke saath kaam karne deta hai.

-----

### Dereference Operator ke Saath Value tak Pointer ko Follow Karna

Ek regular reference ek tarah ka pointer hota hai, aur ek pointer ko aap ek teer (arrow) ke roop mein soch sakte hain jo kahin aur store ki gayi value ki taraf ishaara karta hai. Neeche diye gaye code mein, hum ek `i32` value ka reference banate hain aur phir dereference operator ka istemaal karke us reference ko data tak follow karte hain:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

\<span class="caption"\>Listing 15-6: Ek `i32` value tak reference ko follow karne ke liye dereference operator ka istemaal\</span\>

Variable `x` mein ek `i32` value `5` hai. Hum `y` ko `x` ke reference ke barabar set karte hain. Agar hum `y` mein maujood value ke baare mein assertion karna chahte hain, to humein `*y` ka istemaal karna hoga taaki hum us reference ko us value tak follow kar sakein jis par woh point kar raha hai (isliye isey *dereference* kehte hain). Jab hum `y` ko dereference karte hain, to humein us integer value tak access mil jaata hai jis par `y` point kar raha hai.

Agar hum `assert_eq!(5, y);` likhne ki koshish karte, to humein compilation error milta kyunki ek number aur ek number ke reference ko compare nahi kiya ja sakta, kyonki woh alag-alag types hain.

### `Box<T>` ko ek Reference ki Tarah Istemal Karna

Hum upar diye gaye code ko ek reference ke bajaye `Box<T>` ka istemaal karne ke liye likh sakte hain; dereference operator waise hi kaam karega:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

\<span class="caption"\>Listing 15-7: Ek `Box<i32>` par dereference operator ka istemaal\</span\>

Yahan `y` ab `x` ki value ko point karne wala ek box instance hai. Aakhri assertion mein, hum box ke pointer ko follow karne ke liye dereference operator ka istemaal usi tarah kar sakte hain jaise humne tab kiya tha jab `y` ek reference tha. Ab hum yeh explore karenge ki `Box<T>` mein aisa kya khaas hai jo humein dereference operator ka istemaal karne deta hai.

-----

### Apna Khud ka Smart Pointer Define Karna

Chaliye standard library dwara pradaan kiye gaye `Box<T>` type jaisa hi ek smart pointer banate hain. Isse hum samjhenge ki smart pointers default roop se references se alag kaise behave karte hain. Phir hum dekhenge ki dereference operator ka istemaal karne ki kshamata kaise jodi jaaye.

Hum ek `MyBox` naam ka struct define karte hain. Yeh ek tuple struct hai jismein ek element hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

\<span class="caption"\>Listing 15-8: Ek `MyBox<T>` type ko define karna\</span\>

Ab agar hum `main` function mein `Box<T>` ki jagah `MyBox<T>` ka istemaal karne ki koshish karte hain, to code compile nahi hoga kyunki Rust nahi jaanta ki `MyBox` ko kaise dereference karna hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y); // <-- Yeh line error degi
}
```

\<span class="caption"\>Listing 15-9: `MyBox<T>` ko references aur `Box<T>` ki tarah istemaal karne ki koshish\</span\>

Compilation error yeh hoga: ` type  `MyBox\<{integer}\>`  cannot be dereferenced `.

Hamara `MyBox<T>` type dereference nahi kiya ja sakta kyunki humne apne type par woh kshamata implement nahi ki hai. `*` operator ke saath dereferencing ko enable karne ke liye, humein **`Deref` trait** ko implement karna hoga.

-----

### `Deref` Trait Implement Karke Ek Type ko Reference ki Tarah Treat Karna

Standard library dwara pradan ki gayi `Deref` trait ke liye humein ek method `deref` ko implement karna zaroori hai. Yeh method `self` ko borrow karta hai aur andar ke data ka reference return karta hai. Neeche `MyBox` ke liye `Deref` ka implementation hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::ops::Deref;

# struct MyBox<T>(T);
impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```

\<span class="caption"\>Listing 15-10: `MyBox<T>` par `Deref` ko implement karna\</span\>

Hum `deref` method ke body mein `&self.0` likhte hain, taaki `deref` us value ka reference return kare jise hum `*` operator se access karna chahte hain. Ab `main` function (Listing 15-9) compile ho jaayega aur assertions pass ho jaayenge\!

`Deref` trait ke bina, compiler sirf `&` references ko dereference kar sakta hai. `deref` method compiler ko yeh kshamata deta hai ki woh kisi bhi type jo `Deref` implement karta hai, us par `deref` method call karke ek `&` reference praapt kar sake jise woh dereference karna jaanta hai.

Jab humne `*y` likha, to parde ke peeche (behind the scenes) Rust ne asal mein yeh code chalaya:

```rust,ignore
*(y.deref())
```

Rust `*` operator ko `deref` method ke call aur phir ek plain dereference se badal deta hai. Isse humein yeh sochna nahi padta ki `deref` method call karna hai ya nahi.

`deref` method value ka reference isliye return karta hai taaki ownership system barkarar rahe. Agar `deref` method seedhe value return karta, to value `self` se move ho jaati, aur hum inner value ki ownership nahi lena chahte.

-----

### Functions aur Methods ke Saath Implicit Deref Coercions

**Deref coercion** ek suvidha hai jo Rust functions aur methods ke arguments par perform karta hai. Deref coercion ek aise type ke reference ko jo `Deref` implement karta hai, us type ke reference mein convert karta hai jismein `Deref` original type ko convert kar sakta hai. Yeh tab hota hai jab hum kisi function ko ek aise type ka reference pass karte hain jo function ke parameter type se match nahi karta.

Deref coercion isliye joda gaya tha taaki programmers ko function aur method calls likhte waqt `&` aur `*` ke zariye bahut saare explicit references aur dereferences na add karne padein.

Chaliye ek `hello` function dekhte hain jiska parameter ek string slice hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}
```

\<span class="caption"\>Listing 15-11: Ek `hello` function jiska parameter `name` type `&str` ka hai\</span\>

Deref coercion ki vajah se `hello` function ko `MyBox<String>` ke reference ke saath call karna sambhav hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# // ... MyBox aur Deref ka pichla code ...
# fn hello(name: &str) { println!("Hello, {}!", name); }
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

\<span class="caption"\>Listing 15-12: `MyBox<String>` ke reference ke saath `hello` ko call karna, jo deref coercion ke kaaran kaam karta hai\</span\>

Yahan hum `&m` pass kar rahe hain, jo `&MyBox<String>` hai. Kyunki humne `MyBox<T>` par `Deref` trait implement kiya hai, Rust `&MyBox<String>` ko `deref` call karke `&String` mein badal sakta hai. Standard library `String` par `Deref` ka ek implementation pradaan karti hai jo ek string slice return karta hai. Rust phir se `deref` call karke `&String` ko `&str` mein badal deta hai, jo `hello` function ki definition se match karta hai.

Agar Rust mein deref coercion nahi hota, to humein yeh likhna padta:
`hello(&(*m)[..]);`

Yeh code padhna, likhna aur samajhna zyada mushkil hai. Deref coercion Rust ko in conversions ko hamare liye automatically handle karne deta hai. Yeh sab compile time par hota hai, isliye iska koi runtime penalty nahi hai\!

### Deref Coercion Mutability ke Saath Kaise Interact Karta hai

Jaise aap immutable references par `*` operator ko override karne ke liye `Deref` trait ka istemaal karte hain, waise hi aap mutable references par `*` operator ko override karne ke liye **`DerefMut`** trait ka istemaal kar sakte hain.

Rust teen cases mein deref coercion karta hai:

1.  `&T` se `&U` jab `T: Deref<Target=U>`
2.  `&mut T` se `&mut U` jab `T: DerefMut<Target=U>`
3.  `&mut T` se `&U` jab `T: Deref<Target=U>`

Teesra case thoda tricky hai: Rust ek **mutable reference ko immutable reference mein bhi coerce karega**. Lekin iska ulta sambhav **nahi** hai: immutable references kabhi bhi mutable references mein coerce nahi honge. Borrowing rules ke kaaran, agar aapke paas ek mutable reference hai, to woh us data ka ekमात्र reference hona chahiye. Ek mutable reference ko ek immutable reference mein convert karna kabhi bhi borrowing rules ko nahi todega. Lekin ek immutable ko mutable mein convert karna yeh maang karega ki data ka sirf ek hi immutable reference ho, jiski borrowing rules guarantee nahi dete.
