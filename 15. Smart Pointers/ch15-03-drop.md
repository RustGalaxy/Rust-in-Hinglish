## `Drop` Trait ke Saath Cleanup par Code Run Karna

Smart pointer pattern ke liye doosra zaroori trait hai `Drop`, jo aapko yeh customize karne deta hai ki jab koi value scope se bahar jaane waali ho tab kya hoga. Aap kisi bhi type par `Drop` trait ke liye ek implementation de sakte hain, aur jo code aap specify karte hain, uska istemal files ya network connections jaise resources ko release karne ke liye kiya ja sakta hai. Hum `Drop` ko smart pointers ke sandarbh mein pesh kar rahe hain kyunki `Drop` trait ki functionality lagbhag hamesha tab istemal hoti hai jab ek smart pointer implement kiya jaata hai. Udaharan ke liye, `Box<T>` `Drop` ko customize karta hai taaki heap par us space ko deallocate kar sake jise box point kar raha hai.

Kuchh languages mein, programmer ko har baar smart pointer ka instance istemal karne ke baad memory ya resources ko free karne ke liye code call karna padta hai. Agar woh bhool jaate hain, toh system overload hokar crash ho sakta hai. Rust mein, aap yeh specify kar sakte hain ki jab bhi koi value scope se bahar jaaye, ek khaas code run ho, aur compiler is code ko automatically daal dega. Iske parinaamswaroop, aapko ek program mein har jagah cleanup code lagane ke baare mein savdhaan rehne ki zaroorat nahi hai jahan ek particular type ka instance kaam khatam kar chuka hai—aap phir bhi resources leak nahi karenge\!

Jab koi value scope se bahar jaaye toh kaun sa code run karna hai, yeh `Drop` trait ko implement karke bataya jaata hai. `Drop` trait ke liye aapko `drop` naam ka ek method implement karna hota hai jo `self` ka ek mutable reference leta hai. Yeh dekhne ke liye ki Rust `drop` kab call karta hai, aaiye abhi ke liye `println!` statements ke saath `drop` implement karte hain.

Listing 15-14 ek `CustomSmartPointer` struct dikhata hai jiski ek hi custom functionality hai ki jab instance scope se bahar jayega toh woh `Dropping CustomSmartPointer!` print karega. Yeh example dikhata hai ki Rust `drop` function kab run karta hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    println!("CustomSmartPointers created.");
}
```

\<span class="caption"\>Listing 15-14: Ek `CustomSmartPointer` struct jo `Drop` trait ko implement karta hai jahan hum apna cleanup code rakhenge\</span\>

`Drop` trait prelude mein shaamil hai, isliye humein ise import karne ki zaroorat nahi hai. Hum `CustomSmartPointer` par `Drop` trait ko implement karte hain aur `drop` method ke liye ek implementation dete hain jo `println!` call karta hai. `drop` function ki body woh jagah hai jahan aap koi bhi logic rakhenge jo aap apne type ke instance ke scope se bahar jaane par run karna chahte hain. Hum yahan kuchh text print kar rahe hain yeh dikhane ke liye ki Rust `drop` kab call karega.

`main` mein, hum `CustomSmartPointer` ke do instances banate hain aur phir `CustomSmartPointers created.` print karte hain. `main` ke end mein, hamare `CustomSmartPointer` ke instances scope se bahar chale jayenge, aur Rust woh code call karega jo humne `drop` method mein daala hai, hamara aakhri message print karte hue. Dhyan dein ki humein `drop` method ko explicitly call karne ki zaroorat nahi padi.

Jab hum is program ko run karenge, toh humein neeche diya gaya output dikhega:

```text
CustomSmartPointers created.
Dropping CustomSmartPointer with data `other stuff`!
Dropping CustomSmartPointer with data `my stuff`!
```

Rust ne hamare liye `drop` ko automatically call kiya jab hamare instances scope se bahar gaye, aur us code ko call kiya jo humne specify kiya tha. Variables unke banne ke **reverse order** mein drop hote hain, isliye `d` ko `c` se pehle drop kiya gaya. Yeh example aapko ek visual guide deta hai ki `drop` method kaise kaam karta hai; aam taur par aap apne type ke liye zaroori cleanup code likhte hain, na ki ek print message.

-----

## Kisi Value ko Jaldi Drop Karna `std::mem::drop` ke Saath

Durbhagya se, automatic `drop` functionality ko disable karna seedha nahi hai. `drop` ko disable karna aam taur par zaroori nahi hota; `Drop` trait ka poora matlab hi yahi hai ki iska dhyan automatically rakha jaata hai. Kabhi-kabhi, haalaanki, aap kisi value ko jaldi clean up karna chah sakte hain. Ek udaharan hai jab aap smart pointers ka istemal kar rahe hon jo locks manage karte hain: aap `drop` method ko force karna chah sakte hain jo lock ko release karta hai taaki usi scope mein doosra code lock haasil kar sake. Rust aapko `Drop` trait ke `drop` method ko manually call karne nahi deta; iske bajaye aapko standard library dwara pradan kiya gaya `std::mem::drop` function call karna hoga agar aap kisi value ko uske scope ke khatam hone se pehle drop karne ke liye force karna chahte hain.

Agar hum Listing 15-14 se `main` function ko modify karke `Drop` trait ke `drop` method ko manually call karne ki koshish karte hain, jaisa ki Listing 15-15 mein dikhaya gaya hai, toh humein ek compiler error milega:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer created.");
    c.drop();
    println!("CustomSmartPointer dropped before the end of main.");
}
```

\<span class="caption"\>Listing 15-15: Jaldi cleanup ke liye `Drop` trait se `drop` method ko manually call karne ki koshish\</span\>

Jab hum is code ko compile karne ki koshish karenge, toh humein yeh error milega:

```text
error[E0040]: explicit use of destructor method
  --> src/main.rs:14:7
   |
14 |     c.drop();
   |       ^^^^ explicit destructor calls not allowed
```

Yeh error message batata hai ki humein `drop` ko explicitly call karne ki anumati nahi hai. Error message mein *destructor* shabd ka istemal kiya gaya hai, jo ek general programming term hai us function ke liye jo ek instance ko clean up karta hai. Ek *destructor* ek *constructor* ke samaan hai, jo ek instance banata hai. Rust mein `drop` function ek vishesh destructor hai.

Rust humein `drop` ko explicitly call karne nahi deta kyunki Rust phir bhi `main` ke end mein us value par automatically `drop` call karega. Yeh ek **double free** error hoga kyunki Rust ek hi value ko do baar clean up karne ki koshish kar raha hoga.

Hum `drop` ke automatic insertion ko disable nahi kar sakte jab koi value scope se bahar jaati hai, aur hum `drop` method ko explicitly call nahi kar sakte. Isliye, agar humein kisi value ko jaldi clean up karne ke liye force karna hai, toh hum `std::mem::drop` function ka istemal kar sakte hain.

`std::mem::drop` function `Drop` trait ke `drop` method se alag hai. Hum ise us value ko argument ke roop mein pass karke call karte hain jise hum jaldi drop karna chahte hain. Yeh function prelude mein hai, isliye hum Listing 15-15 mein `main` ko modify karke `drop` function ko call kar sakte hain, jaisa ki Listing 15-16 mein dikhaya gaya hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# struct CustomSmartPointer {
#     data: String,
# }
#
# impl Drop for CustomSmartPointer {
#     fn drop(&mut self) {
#         println!("Dropping CustomSmartPointer!");
#     }
# }
#
fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```

\<span class="caption"\>Listing 15-16: `std::mem::drop` ko call karke ek value ko scope se bahar jaane se pehle explicitly drop karna\</span\>

Is code ko run karne par neeche diya gaya output print hoga:

```text
CustomSmartPointer created.
Dropping CustomSmartPointer with data `some data`!
CustomSmartPointer dropped before the end of main.
```

Text ``Dropping CustomSmartPointer with data `some data`!`` `CustomSmartPointer created.` aur `CustomSmartPointer dropped before the end of main.` ke beech mein print hota hai, yeh dikhate hue ki `drop` method ka code `c` ko drop karne ke liye us point par call kiya gaya hai.

Aap `Drop` trait implementation mein specify kiye gaye code ka istemal cleanup ko suvidhajanak aur surakshit banane ke liye kai tarikon se kar sakte hain: jaise ki, aap iska istemal apna khud ka memory allocator banane ke liye kar sakte hain\! `Drop` trait aur Rust ke ownership system ke saath, aapko cleanup karna yaad rakhne ki zaroorat nahi hai kyunki Rust yeh automatically kar deta hai.

Aapko un samasyaon ke baare mein bhi chinta karne ki zaroorat nahi hai jo galti se abhi bhi istemal ho rahi values ko clean up karne se ho sakti hain: ownership system jo yeh sunishchit karta hai ki references hamesha valid rahen, woh yeh bhi sunishchit karta hai ki `drop` sirf ek baar call ho jab value ka istemal nahi ho raha ho.

Ab jab humne `Box<T>` aur smart pointers ki kuchh visheshtaon ki jaanch kar li hai, aaiye standard library mein define kiye gaye kuchh aur smart pointers ko dekhen.
