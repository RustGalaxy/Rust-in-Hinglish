Bilkul, yahan Hinglish mein anuvaad hai:

## Advanced Types

Rust ke type system mein kuch aise features hain jinka zikr humne ab tak kiya hai, lekin unhe discuss nahi kiya hai. Hum pehle newtypes ke baare mein samany roop se charcha karke shuruwat karenge, yeh dekhte hue ki newtypes types ke roop mein kyun upyogi hain. Phir hum type aliases par aage badhenge, jo newtypes jaisa hi ek feature hai lekin thoda alag semantics ke saath. Hum `!` type aur dynamically sized types par bhi charcha karenge.

-----

### Type Safety aur Abstraction ke liye Newtype Pattern ka Istemal

Newtype pattern un kaamo ke liye bhi upyogi hai jinke baare mein humne ab tak charcha nahi ki hai, jaise ki statically yeh enforce karna ki values kabhi confuse na hon aur ek value ki units batana. Aapne Listing 20-16 mein units batane ke liye newtypes ka istemal dekha tha: yaad karein ki `Millimeters` aur `Meters` structs ne `u32` values ko ek newtype mein wrap kiya tha. Agar hum ek function likhte jiska parameter `Millimeters` type ka hota, toh hum galti se us function ko `Meters` type ki value ya ek sadharan `u32` ke saath call karne wala program compile nahi kar paate.

Hum newtype pattern ka istemal ek type ke kuch implementation details ko abstract karne ke liye bhi kar sakte hain: naya type ek public API expose kar sakta hai jo private inner type ke API se alag ho.

Newtypes internal implementation ko bhi chhipa sakte hain. Udaharan ke liye, hum ek `People` type pradan kar sakte hain jo ek `$HashMap<i32, String>$` ko wrap karta hai, jo ek vyakti ki ID ko unke naam ke saath store karta hai. `People` ka istemal karne wala code sirf hamare dwara pradan kiye gaye public API ke saath interact karega, jaise ki `People` collection mein ek naam string add karne ka method; us code ko yeh janne ki zaroorat nahi hogi ki hum aantarik roop se naamon ko ek `i32` ID assign karte hain. Newtype pattern encapsulation haasil karne ka ek halka tareeka hai implementation details ko chhipane ke liye.

-----

### Type Aliases se Type Synonyms Banana

Rust ek maujooda type ko doosra naam dene ke liye ek **type alias** declare karne ki kshamta pradan karta hai. Iske liye hum `type` keyword ka istemal karte hain. Udaharan ke liye, hum `i32` ke liye `Kilometers` alias is tarah bana sakte hain:

```rust
type Kilometers = i32;
```

Ab alias `Kilometers`, `i32` ke liye ek **synonym** (paryayvachi) hai; Listing 20-16 mein banaye gaye `Millimeters` aur `Meters` types ke vipreet, `Kilometers` ek alag, naya type nahi hai. `Kilometers` type wali values ko `i32` type ki values ki tarah hi treat kiya jaayega:

```rust
let x: i32 = 5;
let y: Kilometers = 5;

println!("x + y = {}", x + y);
```

Kyunki `Kilometers` aur `i32` ek hi type hain, hum dono types ki values ko jod sakte hain aur hum `Kilometers` values ko un functions mein pass kar sakte hain jo `i32` parameters lete hain. Lekin, is method ka istemal karke, humein woh type-checking ke fayde nahi milte jo humein pehle discuss kiye gaye newtype pattern se milte hain. Doosre shabdon mein, agar hum kahin `Kilometers` aur `i32` values ko mix kar dete hain, toh compiler humein error nahi dega.

Type synonyms ka mukhya use case **repetition kam karna** hai. Udaharan ke liye, hamare paas ek lamba type ho sakta hai jaise:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Is lambe type ko function signatures mein aur code mein har jagah type annotations ke roop mein likhna thakaau aur galti-prone ho sakta hai. Listing 20-25 mein is tarah ke code se bhare project ki kalpana karein.

\<Listing number="20-25" caption="Ek lambe type ka kai jagahon par istemal"\>

```rust
let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("hi"));

fn takes_long_type(f: Box<dyn Fn() + Send + 'static>) {
    // --snip--
}

fn returns_long_type() -> Box<dyn Fn() + Send + 'static> {
    // --snip--
    Box::new(|| ())
}
```

\</Listing\>

Ek type alias is code ko repetition kam karke adhik manageable banata hai. Listing 20-26 mein, humne verbose type ke liye `Thunk` naam ka ek alias introduce kiya hai aur ab hum type ke sabhi istemal ko chote alias `Thunk` se badal sakte hain.

\<Listing number="20-26" caption="Repetition kam karne ke liye `Thunk` naam ka ek type alias introduce karna"\>

```rust
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));

fn takes_long_type(f: Thunk) {
    // --snip--
}

fn returns_long_type() -> Thunk {
    // --snip--
    Box::new(|| ())
}
```

\</Listing\>

Yeh code padhne aur likhne mein bahut aasan hai\! Ek type alias ke liye ek arthpurn naam chunna aapke iraade ko communicate karne mein bhi madad kar sakta hai.

Type aliases aam taur par `$Result<T, E>$` type ke saath bhi repetition kam karne ke liye istemal ki jaati hain. Standard library ke `std::io` module par gaur karein. I/O operations aksar `$Result<T, E>$` return karte hain. Is library mein ek `std::io::Error` struct hai jo sabhi sambhav I/O errors ko represent karta hai. `std::io` mein bahut se functions `$Result<T, E>$` return karenge jahan `E` `std::io::Error` hai.

`$Result<..., Error>$` bahut baar repeat hota hai. Isliye, `std::io` mein yeh type alias declaration hai:

```rust,noplayground
type Result<T> = std::result::Result<T, std::io::Error>;
```

Kyunki yeh declaration `std::io` module mein hai, hum fully qualified alias `$std::io::Result<T>$` ka istemal kar sakte hain; yaani, ek `$Result<T, E>$` jismein `E` ko `$std::io::Error$` se bhar diya gaya hai. Type alias do tarah se madad karta hai: yeh code likhna aasan banata hai *aur* yeh humein `std::io` ke sabhi hisson mein ek consistent interface deta hai.

-----

### The Never Type jo Kabhi Return Nahi Karta

Rust mein ek khaas type hai jiska naam `!` hai, jise type theory ki bhasha mein **empty type** kaha jaata hai kyunki iski koi values nahi hoti. Hum ise **never type** kehna pasand karte hain kyunki yeh return type ki jagah leta hai jab ek function kabhi return nahi karega. Yahan ek example hai:

```rust,noplayground
fn bar() -> ! {
    // --snip--
    panic!();
}
```

Is code ko is tarah padha jaata hai "function `bar` kabhi return nahi karta." Jo functions kabhi return nahi karte, unhe **diverging functions** kaha jaata hai. Hum `!` type ki values nahi bana sakte, isliye `bar` kabhi bhi return nahi kar sakta.

Lekin ek aise type ka kya fayda jiski aap kabhi values nahi bana sakte? Number-guessing game ke code ko yaad karein. Usmein ek `match` tha jiski ek arm `continue` mein khatm hoti thi. Us samay, humne is code ke kuch details ko chhod diya tha. Humne `match` ke baare mein charcha karte samay kaha tha ki `match` ki sabhi arms ko ek hi type return karna chahiye. Toh, `continue` kya return karta hai?

Jaisa ki aapne anumaan lagaya hoga, `continue` ka type `!` hota hai. Yaani, jab Rust `guess` ka type compute karta hai, toh woh dono match arms ko dekhta hai, pehli `u32` value ke saath aur doosri `!` value ke saath. Kyunki `!` ki kabhi koi value nahi ho sakti, Rust yeh faisla karta hai ki `guess` ka type `u32` hai.

Is behavior ko formal tareeke se describe karne ka tareeka yeh hai ki `!` type ke expressions ko kisi bhi doosre type mein **coerced** (roopantarit) kiya ja sakta hai. Humein is `match` arm ko `continue` ke saath khatm karne ki anumati hai kyunki `continue` koi value return nahi karta; balki, yeh control ko loop ke top par wapas le jaata hai, isliye `Err` case mein, hum kabhi `guess` ko koi value assign nahi karte.

Never type `panic!` macro ke saath bhi upyogi hai. Iske alawa, ek aur expression jiska type `!` hai, woh hai `loop` (bina `break` ke), kyunki woh loop kabhi khatm nahi hota.

-----

### Dynamically Sized Types aur `Sized` Trait

Rust ko apne types ke baare mein kuch details janne ki zaroorat hoti hai, jaise ki ek particular type ki value ke liye kitni space allocate karni hai. Yeh uske type system ke ek kone ko thoda confusing banata hai: **dynamically sized types** ka concept. Inhe kabhi-kabhi **DSTs** ya **unsized types** bhi kaha jaata hai, yeh types humein aisi values ka istemal karke code likhne dete hain jinka size hum sirf runtime par hi jaan sakte hain.

Chaliye ek dynamically sized type `str` ke details mein jaate hain, jiska istemal hum poori kitaab mein karte aa rahe hain. Ji haan, `&str` nahi, balki `str` apne aap mein ek DST hai. Hum runtime tak nahi jaan sakte ki ek string kitni lambi hai. Iska matlab hai ki hum `str` type ka variable nahi bana sakte, na hi hum `str` type ka argument le sakte hain. Yeh code kaam nahi karega:

```rust,ignore,does_not_compile
let s1: str = "Hello there!";
let s2: str = "How's it going?";
```

Toh hum kya karein? Is case mein, aap jawab pehle se jaante hain: hum `s1` aur `s2` ke types ko `str` ke bajaye `&str` bana dete hain. Yaad karein ki slice data structure sirf starting position aur slice ki length store karta hai. Isliye, jabki `&T` ek single value hai jo `T` ke memory address ko store karta hai, `&str` **do** values hai: `str` ka address aur uski length. Aise mein, hum compile time par `&str` value ka size jaan sakte hain: yeh ek `usize` ki lambai ka dugna hai.

Samany roop se, yahi tareeka hai jisse dynamically sized types Rust mein istemal kiye jaate hain: unke paas ek extra metadata hota hai jo dynamic information ka size store karta hai. **Dynamically sized types ka sunhara niyam yeh hai ki humein hamesha dynamically sized types ki values ko kisi na kisi pointer ke peeche rakhna chahiye.**

DSTs ke saath kaam karne ke liye, Rust **`Sized` trait** pradan karta hai yeh determine karne ke liye ki kya kisi type ka size compile time par jaana ja sakta hai ya nahi. Yeh trait un sabhi cheezon ke liye automatically implement kiya jaata hai jinka size compile time par pata hota hai. Iske alawa, Rust har generic function par `Sized` ka ek bound implicitly jod deta hai. Yaani, ek aisi generic function definition:

```rust,ignore
fn generic<T>(t: T) {
    // --snip--
}
```

Vastav mein is tarah treat ki jaati hai jaise humne yeh likha ho:

```rust,ignore
fn generic<T: Sized>(t: T) {
    // --snip--
}
```

Default roop se, generic functions sirf un types par kaam karenge jinka size compile time par pata hota hai. Lekin, aap is pabandi ko hatane ke liye nimnalikhit special syntax ka istemal kar sakte hain:

```rust,ignore
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

`?Sized` par ek trait bound ka matlab hai "`T` `Sized` ho bhi sakta hai aur nahi bhi" aur yeh notation us default ko override karta hai ki generic types ka size compile time par pata hona chahiye. `?Trait` syntax is arth ke saath sirf `Sized` ke liye uplabdh hai, kisi aur trait ke liye nahi.

Yeh bhi dhyan dein ki humne `t` parameter ke type ko `T` se `&T` mein badal diya. Kyunki type `Sized` nahi ho sakta, humein ise kisi pointer ke peeche istemal karna hoga. Is case mein, humne ek reference chuna hai.