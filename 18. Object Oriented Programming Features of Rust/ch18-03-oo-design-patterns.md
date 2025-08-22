## Ek Object-Oriented Design Pattern Implement Karna

**State pattern** ek object-oriented design pattern hai. Is pattern ka mukhya bindu yah hai ki hum ek value ke liye states ka ek set define karte hain jo vah aantarik (internally) roop se ho sakti hai. States ko **state objects** ke ek set dwara represent kiya jaata hai, aur value ka vyavhaar (behavior) uske state ke aadhar par badalta hai. Hum ek blog post `struct` ka udaharan lenge jismein uske state ko hold karne ke liye ek field hoga, jo "draft," "review," ya "published" set se ek state object hoga.

State objects functionality share karte hain: Rust mein, hum objects aur inheritance ke bajaye structs aur traits ka istemal karte hain. Har state object apne khud ke behavior ke liye aur yah tay karne ke liye zimmedaar hai ki use kab doosre state mein badalna chahiye. Vah value jo ek state object ko hold karti hai, use states ke alag-alag behavior ya states ke beech kab transition karna hai, iske baare mein kuch nahi pata hota.

State pattern ka istemal karne ka fayda yah hai ki, jab program ki business requirements badalti hain, to humein state hold karne wali value ke code ya us value ka istemal karne wale code ko badalne ki zaroorat nahi hogi. Humein keval ek state object ke andar ke code ko uske niyamon ko badalne ke liye update karna hoga ya shayad aur state objects jodne honge.

Pehle hum state pattern ko ek zyada traditional object-oriented tareeke se implement karenge, phir hum ek aisa approach istemal karenge jo Rust mein thoda zyada natural hai.

Antim functionality is tarah dikhegi:

1.  Ek blog post ek khaali draft ke roop mein shuru hota hai.
2.  Jab draft poora ho jaata hai, to post ki review ke liye request ki jaati hai.
3.  Jab post approve ho jaata hai, to vah publish ho jaata hai.
4.  Keval published blog posts hi print karne ke liye content return karte hain, taaki unapproved posts galti se publish na ho sakein.

-----

### Ek Traditional Object-oriented Prayas

Listing 18-11 is workflow ko code ke roop mein dikhata hai: yah us API ke upyog ka ek udaharan hai jise hum `blog` naamak ek library crate mein implement karenge. Yah abhi compile nahi hoga.

```rust,ignore,does_not_compile
// Filename: src/main.rs
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```

*Listing 18-11: Code jo us desired behavior ko darshata hai jo hum chahte hain ki hamara `blog` crate rakhe*

Dhyan dein ki hum crate se keval `Post` type ke saath interact kar rahe hain. Yah type state pattern ka istemal karega aur ek aisi value hold karega jo teen state objects mein se ek hogi. Ek state se doosre mein badalna `Post` type ke andar aantarik roop se manage kiya jayega.

#### `Post` Define Karna aur Draft State mein Naya Instance Banana

Chaliye library ke implementation par shuruaat karte hain\! Humein ek public `Post` struct ki zaroorat hai, ek `new` function ki, aur ek private `State` trait ki jo sabhi state objects ke liye behavior define karega.

`Post` `state` naamak ek private field mein `Box<dyn State>` ka ek trait object hold karega.

```rust
// Filename: src/lib.rs
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }
}

trait State {}

struct Draft {}

impl State for Draft {}
```

*Listing 18-12: `Post` struct ki definition, ek `new` function, `State` trait, aur `Draft` struct*

Jab hum ek naya `Post` banate hain, to hum uske `state` field ko `Some(Box::new(Draft {}))` par set karte hain. Yah sunishchit karta hai ki jab bhi hum ek naya `Post` banate hain, to vah ek draft ke roop mein shuru hoga.

#### Post Content ka Text Store Karna

Hum ek `add_text` method implement karte hain. Yah method `self` ka ek mutable reference leta hai kyunki hum `Post` instance ko badal rahe hain. Yah `content` field mein text jodta hai. Yah behavior post ke state par nirbhar nahi karta.

```rust
// Filename: src/lib.rs
# pub struct Post {
#     content: String,
# }
impl Post {
    // ...
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

*Listing 18-13: Post ke `content` mein text jodne ke liye `add_text` method ka implementation*

#### Yah Sunishchit Karna ki Draft Post ka Content Khaali Ho

Bhale hi humne `add_text` call kiya ho, hum chahte hain ki `content` method ek khaali string slice return kare kyunki post abhi bhi draft state mein hai. Hum `content` method ko ek placeholder implementation ke saath jodte hain jo hamesha ek khaali string slice return karta hai.

```rust
// Filename: src/lib.rs
# pub struct Post {
#     content: String,
# }
impl Post {
    // ...
    pub fn content(&self) -> &str {
        ""
    }
}
```

*Listing 18-14: `Post` par `content` method ke liye ek placeholder implementation jodna*

#### Review Request Karna Post ke State ko Badal Deta Hai

Ab, humein ek post ki review request karne ke liye functionality jodni hai, jisse uska state `Draft` se `PendingReview` mein badal jana chahiye.

```rust
// Filename: src/lib.rs
// ...
impl Post {
    // ...
    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}
impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
}

struct PendingReview {}
impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
```

*Listing 18-15: `Post` aur `State` trait par `request_review` methods ka implementation*

Hum `Post` ko `request_review` naamak ek public method dete hain. Phir hum `Post` ke vartaman state par ek internal `request_review` method call karte hain.

Hum `State` trait mein `request_review` method jodte hain. Dhyan dein ki method ka pehla parameter `self: Box<Self>` hai. Is syntax ka matlab hai ki method keval `Box` hold karne wale type par call kiye jaane par hi valid hai. Yah syntax `Box<Self>` ka ownership le leta hai, purane state ko anvaidh (invalidating) kar deta hai taaki `Post` ka state value ek naye state mein badal sake.

Purane state ko consume karne ke liye, `request_review` method ko state value ka ownership lena hoga. Yahin par `Post` ke `state` field mein `Option` kaam aata hai: hum `state` field se `Some` value nikalne ke liye `take` method call karte hain aur uski jagah `None` chhod dete hain.

Ab hum state pattern ke fayde dekhna shuru kar sakte hain: `Post` par `request_review` method uske `state` value se anapekshit (irrespective) hai. Har state apne khud ke niyamon ke liye zimmedaar hai.

#### `approve` Method Jodna jo `content` ke Behavior ko Badalta hai

`approve` method `request_review` method ke samaan hoga: yah `state` ko us value par set karega jo vartaman state kehta hai ki jab vah state approve ho to hona chahiye.

```rust
// Filename: src/lib.rs
# trait State {
#     fn request_review(self: Box<Self>) -> Box<dyn State>;
#     fn approve(self: Box<Self>) -> Box<dyn State>;
# }
# struct PendingReview {}
# impl State for PendingReview {
#     fn request_review(self: Box<Self>) -> Box<dyn State> { self }
#     fn approve(self: Box<Self>) -> Box<dyn State> {
#         Box::new(Published {})
#     }
# }
# struct Published {}
# impl State for Published {
#     fn request_review(self: Box<Self>) -> Box<dyn State> { self }
#     fn approve(self: Box<Self>) -> Box<dyn State> { self }
# }
# pub struct Post {
#    state: Option<Box<dyn State>>,
# }
impl Post {
    // ...
    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}
// ... (Draft and PendingReview structs and their impls)
```

*Listing 18-16: `Post` aur `State` trait par `approve` method ka implementation*

Hum `State` trait mein `approve` method jodte hain aur ek naya struct `Published` jodte hain jo `State` implement karta hai. Jab hum `PendingReview` par `approve` call karte hain, to yah `Published` struct ka ek naya, boxed instance return karta hai.

Ab humein `Post` par `content` method ko update karne ki zaroorat hai. Hum `Post` ko apne `state` par define kiye gaye `content` method ko delegate karvayenge.

```rust
// Filename: src/lib.rs (updated content method on Post)
impl Post {
    // ...
    pub fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(self)
    }
    // ...
}
```

*Listing 18-17: `Post` par `content` method ko `State` par ek `content` method ko delegate karne ke liye update karna*

Iska matlab hai ki humein `State` trait definition mein `content` jodna hoga.

```rust
// Filename: src/lib.rs
trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    fn approve(self: Box<Self>) -> Box<dyn State>;
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

// ...

struct Published {}
impl State for Published {
    // ...
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}
```

*Listing 18-18: `State` trait mein `content` method jodna*

Hum `content` method ke liye ek default implementation jodte hain jo ek khaali string slice return karta hai. Iska matlab hai ki humein `Draft` aur `PendingReview` structs par `content` implement karne ki zaroorat nahi hai. `Published` struct `content` method ko override karega aur `post.content` mein value return karega.

Aur humara kaam ho gaya—Listing 18-11 ab poori tarah se kaam karta hai\!

#### State Pattern ke Trade-offs

Rust object-oriented state pattern ko implement karne mein saksham hai. Is pattern ka ek nuksan yah hai ki kuch states ek doosre se jude (coupled) hote hain. Ek aur nuksan yah hai ki humne kuch logic ko duplicate kiya hai.

Object-oriented languages ke liye theek vaise hi state pattern ko implement karke, hum Rust ki taakaton ka poora fayda nahi utha rahe hain. Chaliye kuch aise badlav dekhte hain jo hum kar sakte hain jo anvaidh states aur transitions ko compile-time errors mein badal sakte hain.

-----

### States aur Behavior ko Types ke Roop Mein Encode Karna

Ab hum state pattern ko alag tarike se sochenge. States aur transitions ko poori tarah se encapsulate karne ke bajaye, hum states ko alag-alag **types** mein encode karenge. Parinaamsvaroop, Rust ka type checking system draft posts ka istemal un jagahon par rok dega jahan keval published posts ki anumati hai, ek compiler error dekar.

Hum draft posts ke liye `content` method hi nahi rakhenge. Is tarah, agar hum ek draft post ka content prapt karne ki koshish karte hain, to humein ek compiler error milega jo batayega ki method मौजूद nahi hai.

```rust
// Filename: src/lib.rs
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

impl Post {
    pub fn new() -> DraftPost {
        DraftPost {
            content: String::new(),
        }
    }

    pub fn content(&self) -> &str {
        &self.content
    }
}

impl DraftPost {
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

*Listing 18-19: `content` method ke saath ek `Post` aur `content` method ke bina `DraftPost`*

`Post` aur `DraftPost` dono structs mein ek private `content` field hai. `Post::new` ab `Post` ka instance return karne ke bajaye `DraftPost` ka instance return karta hai. `DraftPost` struct mein `add_text` method hai, lekin `content` method define nahi hai\!

#### Transitions ko Alag-alag Types mein Transformations ke Roop Mein Implement Karna

To hum ek published post kaise prapt karte hain? Hum is niyam ko laagoo karna chahte hain ki ek draft post ko publish hone se pehle review aur approve karna hoga. Hum ek aur struct, `PendingReviewPost`, jodkar in badhyataon ko implement karenge.

```rust
// Filename: src/lib.rs
// ...
impl DraftPost {
    // ...
    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost {
            content: self.content,
        }
    }
}

pub struct PendingReviewPost {
    content: String,
}

impl PendingReviewPost {
    pub fn approve(self) -> Post {
        Post {
            content: self.content,
        }
    }
}
```

*Listing 18-20: `DraftPost` par `request_review` call karke `PendingReviewPost` banaya jaata hai aur ek `approve` method jo `PendingReviewPost` ko ek published `Post` mein badal deta hai*

`request_review` aur `approve` methods `self` ka ownership le lete hain, is prakaar `DraftPost` aur `PendingReviewPost` instances ko consume karke unhe kramashah `PendingReviewPost` aur ek published `Post` mein transform kar dete hain.

Iske liye humein `main` mein bhi kuch chhote badlav karne honge. `request_review` aur `approve` methods naye instances return karte hain, isliye humein return kiye gaye instances ko save karne ke liye `let post = ...` shadowing assignments jodne honge.

```rust
// Filename: src/main.rs (Updated)
use blog::Post;

fn main() {
    let mut post = Post::new();
    post.add_text("I ate a salad for lunch today");
    
    let post = post.request_review();
    let post = post.approve();
    
    assert_eq!("I ate a salad for lunch today", post.content());
}
```

*Listing 18-21: Blog post workflow ke naye implementation ka istemal karne ke liye `main` mein sanshodhan*

Is implementation ka fayda yah hai ki anvaidh states ab type system aur compile time par hone wali type checking ke kaaran asambhav hain\! Yah sunishchit karta hai ki kuch bugs, jaise ki ek anprakashit post ke content ka display, production mein jaane se pehle hi khoj liye jayenge.

Humne dekha hai ki bhale hi Rust object-oriented design patterns ko implement karne mein saksham hai, anya patterns, jaise ki state ko type system mein encode karna, Rust mein bhi uplabdh hain. In patterns ke alag-alag trade-offs hain. Object-oriented patterns hamesha Rust ki taakaton ka fayda uthane ka sabse accha tareeka nahi honge.