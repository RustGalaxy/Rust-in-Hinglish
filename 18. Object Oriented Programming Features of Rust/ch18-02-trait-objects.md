## Sajha Behavior par Abstract Karne ke Liye Trait Objects ka Istemal

Chapter 8 mein, humne bataya tha ki vectors ki ek seema (limitation) yah hai ki ve keval ek hi type ke elements store kar sakte hain. Humne iska ek samadhan Listing 8-9 mein banaya tha jahan humne ek `SpreadsheetCell` enum define kiya tha jiske variants integers, floats, aur text hold kar sakte the. Yah ek bilkul sahi samadhan hai jab hamare interchangeable items ek nishchit set of types ke hon jinhe hum code compile karte samay jaante hain.

Halaanki, kabhi-kabhi hum chahte hain ki hamari library ka user un types ke set ko extend kar sake jo ek vishesh sthiti mein valid hain. Ise dikhane ke liye, hum ek udaharan graphical user interface (GUI) tool banayenge jo items ki ek list par iterate karta hai, aur har ek par ek `draw` method call karke use screen par draw karta hai—GUI tools ke liye ek aam technique. Hum `gui` naamak ek library crate banayenge. Is crate mein `Button` ya `TextField` jaise kuch types ho sakte hain. Iske alawa, `gui` ke users apne khud ke types banana chahenge jinhe draw kiya ja sake: jaise, ek programmer `Image` jod sakta hai aur doosra `SelectBox`.

Library likhte samay, hum un sabhi types ko nahi jaante jo doosre programmers banana chahenge. Lekin hum yah jaante hain ki `gui` ko kai alag-alag types ki values ka track rakhna hoga, aur use in sabhi alag-alag typed values par ek `draw` method call karna hoga.

Inheritance wali bhasha mein aisa karne ke liye, hum `Component` naamak ek class define kar sakte hain jismein `draw` method ho. `Button`, `Image`, aur `SelectBox` jaisi anya classes `Component` se inherit karengi aur is prakaar `draw` method ko bhi inherit karengi. Lekin kyunki Rust mein inheritance nahi hai, humein `gui` library ko is tarah se structure karne ke liye ek aur tareeke ki zaroorat hai.

-----

### Sajha Behavior ke Liye ek Trait Define Karna

Is behavior ko implement karne ke liye, hum `Draw` naamak ek trait define karenge jismein `draw` naamak ek method hoga. Phir hum ek vector define kar sakte hain jo ek **trait object** leta hai. Ek *trait object* ek aise type ke instance aur us type par trait methods ko runtime par lookup karne ke liye istemal hone wali ek table, dono par point karta hai. Hum ek trait object ko kisi prakaar ke pointer, jaise `&` reference ya `Box<T>` smart pointer, phir `dyn` keyword, aur phir प्रासंगिक trait ko specify karke banate hain.

Hum trait objects ka istemal ek generic ya concrete type ki jagah kar sakte hain. Jahan bhi hum ek trait object ka istemal karte hain, Rust ka type system compile time par yah sunishchit karega ki us context mein istemal ki gayi koi bhi value trait object ke trait ko implement karegi. Parinaamsvaroop, humein compile time par sabhi sambhavit types ko jaanne ki zaroorat nahi hai.

Trait objects doosri bhashaon ke objects se is maamle mein alag hain ki hum ek trait object mein data nahi jod sakte. Unka vishesh uddeshya saajha vyavhaar (common behavior) par abstraction ki anumati dena hai.

Listing 18-3 dikhata hai ki `Draw` naamak ek trait ko kaise define kiya jaaye.

```rust
// Filename: src/lib.rs
pub trait Draw {
    fn draw(&self);
}
```

*Listing 18-3: `Draw` trait ki definition*

-----

Ab kuch naya syntax aata hai: Listing 18-4 ek `Screen` naamak struct define karta hai jismein `components` naamak ek vector hai. Is vector ka type `Box<dyn Draw>` hai, jo ek trait object hai; yah `Box` ke andar kisi bhi type ke liye ek stand-in hai jo `Draw` trait ko implement karta hai.

```rust
// Filename: src/lib.rs
# pub trait Draw {
#     fn draw(&self);
# }
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}
```

*Listing 18-4: Ek `Screen` struct ki definition jiska `components` field trait objects ka ek vector hold karta hai*

-----

`Screen` struct par, hum `run` naamak ek method define karenge jo apne har ek `components` par `draw` method ko call karega.

```rust
// Filename: src/lib.rs
# pub trait Draw {
#     fn draw(&self);
# }
# pub struct Screen {
#     pub components: Vec<Box<dyn Draw>>,
# }
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

*Listing 18-5: `Screen` par ek `run` method jo har component par `draw` method call karta hai*

Yah ek aise struct ko define karne se alag kaam karta hai jo generic type parameter ka upyog karta hai. Ek generic type parameter ek samay mein keval ek hi concrete type se substitute kiya ja sakta hai, jabki trait objects runtime par kai concrete types ko trait object ke liye fill in karne ki anumati dete hain. Generics aur trait bounds ka upyog karna behtar hai agar aapke paas hamesha **homogeneous collections** (ek hi type ke elements) honge.

Doosri or, trait objects ke saath, ek `Screen` instance ek `Vec` hold kar sakta hai jismein `Box<Button>` ke saath-saath `Box<TextField>` bhi ho (ek **heterogeneous collection**).

-----

### Trait ko Implement Karna

Ab hum kuch aise types jodenge jo `Draw` trait ko implement karte hain. Hum `Button` type pradan karenge.

```rust
// Filename: src/lib.rs
# pub trait Draw {
#     fn draw(&self);
# }
#[derive(Debug)]
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // Code to actually draw a button
        println!("Drawing a button: {:?}", self);
    }
}
```

*Listing 18-7: Ek `Button` struct jo `Draw` trait ko implement karta hai*

Agar hamari library ka istemal karne wala koi vyakti `SelectBox` struct implement karne ka faisla karta hai, to vah bhi `SelectBox` type par `Draw` trait ko implement karega.

```rust
// Filename: src/main.rs (in a separate crate)
use gui::Draw;

#[derive(Debug)]
struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // Code to actually draw a select box
        println!("Drawing a select box: {:?}", self);
    }
}
```

*Listing 18-8: `gui` ka istemal karne wala ek aur crate jo `SelectBox` struct par `Draw` trait implement karta hai*

Hamari library ka user ab `main` function likh sakta hai. Vah `Screen` instance mein `SelectBox` aur `Button` dono ko `Box<T>` mein daalkar jod sakta hai.

```rust
// Filename: src/main.rs
use gui::{Button, Screen, Draw};

// ... SelectBox definition and impl Draw for SelectBox ...

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

*Listing 18-9: Ek hi trait ko implement karne wale alag-alag types ki values ko store karne ke liye trait objects ka istemal karna*

Jab humne library likhi, humein nahi pata tha ki koi `SelectBox` type jodega, lekin hamara `Screen` implementation naye type par kaam kar saka aur use draw kar saka kyunki `SelectBox` `Draw` trait ko implement karta hai.

Yah concept—keval un messages se chintit hona jinka ek value jawab deta hai, na ki value ke concrete type se—dynamically typed languages mein **duck typing** ke concept ke samaan hai: agar yah ek battakh ki tarah chalta hai aur ek battakh ki tarah quack karta hai, to yah ek battakh hi hona chahiye\! 🦆

Trait objects ka upyog karne ka fayda yah hai ki humein kabhi bhi runtime par yah jaanchne ki zaroorat nahi hoti ki koi value ek vishesh method ko implement karti hai ya nahi. Rust hamare code ko compile nahi karega agar values un traits ko implement nahi karti hain jinki trait objects ko zaroorat hai.

-----

### Trait Objects Dynamic Dispatch Karte Hain

Chapter 10 mein generics par hamari charcha mein **monomorphization** prakriya ko yaad karein: compiler har concrete type ke liye functions aur methods ke non-generic implementations generate karta hai. Isse utpann code **static dispatch** kar raha hai, jo tab hota hai jab compiler jaanta hai ki aap compile time par kaun sa method call kar rahe hain. Iske vipreet **dynamic dispatch** hai, jo tab hota hai jab compiler compile time par nahi bata sakta ki aap kaun sa method call kar rahe hain. Dynamic dispatch ke maamlon mein, compiler aisa code emit karta hai jo runtime par jaanta hoga ki kaun sa method call karna hai.

Jab hum trait objects ka istemal karte hain, to Rust ko dynamic dispatch ka istemal karna chahiye. Compiler ko un sabhi types ka pata nahi hota jo trait objects ka istemal karne wale code ke saath istemal kiye ja sakte hain. Iske bajaye, runtime par, Rust yah jaanne ke liye trait object ke andar ke pointers ka istemal karta hai ki kaun sa method call karna hai. Is lookup mein ek runtime laagat (cost) aati hai jo static dispatch ke saath nahi hoti. Dynamic dispatch compiler ko ek method ke code ko inline karne se bhi rokta hai, jo badle mein kuch optimizations ko rokta hai. Halaanki, humein code mein atirikt lacheelapan (flexibility) mila, isliye yah ek trade-off hai jis par vichaar karna chahiye.