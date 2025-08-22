## Advanced Functions and Closures

Yeh section functions aur closures se jude kuch advanced features ko explore karta hai, jismein function pointers aur returning closures shaamil hain.

-----

### Function Pointers

Humne functions mein closures pass karne ke baare mein baat ki hai; aap functions mein regular functions bhi pass kar sakte hain\! Yeh technique tab upyogi hoti hai jab aap ek naya closure define karne ke bajaye pehle se define kiye gaye function ko pass karna chahte hain. Functions `fn` type (chote *f* ke saath) mein coerce ho jaate hain, isey `Fn` closure trait ke saath confuse na karein. `fn` type ko **function pointer** kaha jaata hai. Function pointers ke saath functions pass karna aapko functions ko doosre functions ke arguments ke roop mein istemal karne dega.

Ek parameter ko function pointer ke roop mein specify karne ka syntax closures jaisa hi hai, jaisa ki Listing 20-28 mein dikhaya gaya hai. Yahan humne ek function `add_one` define kiya hai jo apne parameter mein 1 jodta hai. Function `do_twice` do parameters leta hai: ek function pointer jo kisi bhi aise function ko point karta hai jo ek `i32` parameter leta hai aur ek `i32` return karta hai, aur ek `i32` value. `do_twice` function `f` ko do baar call karta hai, use `arg` value pass karke, phir dono function call ke results ko jod deta hai. `main` function `do_twice` ko `add_one` aur `5` arguments ke saath call karta hai.

\<Listing number="20-28" file-name="src/main.rs" caption="`fn` type ka istemal karke ek function pointer ko argument ke roop mein accept karna"\>

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);
    println!("The answer is: {}", answer);
}
```

\</Listing\>

Yeh code `The answer is: 12` print karta hai. Humne specify kiya ki `do_twice` mein parameter `f` ek `fn` hai jo `i32` type ka ek parameter leta hai aur `i32` return karta hai. Phir hum `do_twice` ki body mein `f` ko call kar sakte hain. `main` mein, hum function ka naam `add_one` `do_twice` ke pehle argument ke roop mein pass kar sakte hain.

Closures ke vipreet, `fn` ek **type** hai na ki ek trait, isliye hum `fn` ko seedhe parameter type ke roop mein specify karte hain, bajaye iske ki hum ek generic type parameter declare karein jismein `Fn` traits mein se ek trait bound ho.

Function pointers teeno closure traits (`Fn`, `FnMut`, aur `FnOnce`) ko implement karte hain, jiska matlab hai ki aap hamesha ek function pointer ko ek aise function ke liye argument ke roop mein pass kar sakte hain jo ek closure expect karta hai. Behtar yahi hai ki aap functions ko ek generic type aur closure traits mein se ek ka istemal karke likhein taaki aapke functions functions ya closures dono ko accept kar sakein.

Iske bawajood, ek aisi jagah jahan aap sirf `fn` accept karna chahenge na ki closures, woh hai jab aap external code ke saath interface kar rahe hon jismein closures nahi hote: C functions arguments ke roop mein functions accept kar sakte hain, lekin C mein closures nahi hote.

Ek example ke liye jahan aap ya toh inline-defined closure ya ek named function ka istemal kar sakte hain, chaliye standard library mein `Iterator` trait dwara pradan kiye gaye `map` method ka istemal dekhte hain. Numbers ke vector ko strings ke vector mein badalne ke liye `map` method ka istemal karne ke liye, hum ek closure ka istemal kar sakte hain, jaisa ki Listing 20-29 mein hai.

\<Listing number="20-29" caption="Numbers ko strings mein convert karne ke liye `map` method ke saath ek closure ka istemal"\>

```rust
fn main() {
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(|i| i.to_string()).collect();
    println!("{list_of_strings:?}");
}
```

\</Listing\>

Ya hum closure ke bajaye `map` ke argument ke roop mein ek function ka naam de sakte hain, jaisa ki Listing 20-30 dikhata hai.

\<Listing number="20-30" caption="Numbers ko strings mein convert karne ke liye `map` method ke saath `String::to_string` function ka istemal"\>

```rust
fn main() {
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(ToString::to_string).collect();
    println!("{list_of_strings:?}");
}
```

\</Listing\>

Dhyan dein ki humein yahan fully qualified syntax ka istemal karna hoga kyunki `to_string` naam ke kai functions uplabdh hain.

Enum values ke baare mein yaad karein, har enum variant ka naam ek initializer function bhi ban jaata hai. Hum in initializer functions ko function pointers ke roop mein istemal kar sakte hain jo closure traits ko implement karte hain. Iska matlab hai ki hum initializer functions ko un methods ke liye arguments ke roop mein specify kar sakte hain jo closures lete hain, jaisa ki Listing 20-31 mein dekha ja sakta hai.

\<Listing number="20-31" caption="Numbers se `Status` instance banane ke liye `map` method ke saath ek enum initializer ka istemal"\>

```rust
#[derive(Debug)]
enum Status {
    Value(u32),
    Stop,
}

fn main() {
    let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
    println!("{list_of_statuses:?}");
}
```

\</Listing\>

Yahan, hum `Status::Value` ke initializer function ka istemal karke `map` call kiye gaye range ke har `u32` value se `Status::Value` instances banate hain. Kuch log is style ko pasand karte hain aur kuch log closures ka istemal karna pasand karte hain. Dono ek hi code mein compile hote hain, isliye jo style aapko zyada saaf lage, uska istemal karein.

-----

### Closures ko Return Karna

Closures traits dwara represent kiye jaate hain, jiska matlab hai ki aap closures ko seedhe return nahi kar sakte. Zyada-tar cases mein jahan aap ek trait return karna chahte hain, aap us trait ko implement karne wale concrete type ko function ke return value ke roop mein istemal kar sakte hain. Lekin, aap aam taur par closures ke saath aisa nahi kar sakte kyunki unka koi concrete type nahi hota jo returnable ho; agar closure apne scope se koi value capture karta hai, toh aapko function pointer `fn` ko return type ke roop mein istemal karne ki anumati nahi hai.

Iske bajaye, aap aam taur par `impl Trait` syntax ka istemal karenge. Aap `Fn`, `FnOnce` aur `FnMut` ka istemal karke koi bhi function type return kar sakte hain. Udaharan ke liye, Listing 20-32 ka code aaram se compile ho jaayega.

\<Listing number="20-32" caption="`impl Trait` syntax ka istemal karke ek function se closure return karna"\>

```rust
fn returns_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}
```

\</Listing\>

Lekin, jaisa ki humne Chapter 13 mein dekha tha, har closure ka apna alag type hota hai. Agar aapko ek hi signature wale lekin alag-alag implementation wale multiple functions ke saath kaam karna hai, toh aapko unke liye ek **trait object** ka istemal karna hoga. Sochiye ki kya hota hai agar aap Listing 20-33 jaisa code likhte hain.

\<Listing file-name="src/main.rs" number="20-33" caption="`impl Fn` types return karne wale functions dwara define kiye gaye closures ka `Vec&lt;T&gt;` banana"\>

```rust,ignore,does_not_compile
fn returns_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}

fn returns_initialized_closure() -> impl Fn(i32) -> i32 {
    let y = 10;
    move |x| x + y
}

fn main() {
    let mut closures = Vec::new();

    closures.push(returns_closure());
    closures.push(returns_initialized_closure());
}
```

\</Listing\>

Yahan hamare paas do functions hain, `returns_closure` aur `returns_initialized_closure`, jo dono `$impl Fn(i32) -> i32$` return karte hain. Dhyan dein ki unke dwara return kiye gaye closures alag-alag hain, bhale hi woh ek hi type implement karte hain. Agar hum ise compile karne ki koshish karte hain, toh Rust humein batata hai ki yeh kaam nahi karega:

```text
error[E0308]: `if` and `else` have incompatible types
  --> src/main.rs:14:5
   |
14 |     closures.push(returns_initialized_closure());
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected `()`, found a different `()`
   |
   = note: expected closure `[closure@src/main.rs:2:29: 2:38]`
              found closure `[closure@src/main.rs:7:22: 7:33]`
   = note: closures are anonymous types, and so have distinct types
```

Error message batata hai ki jab bhi hum `impl Trait` return karte hain, Rust ek unique **opaque type** banata hai—ek aisa type jiske details hum dekh nahi sakte. Isliye bhale hi yeh functions aise closures return karte hain jo ek hi trait, `$Fn(i32) -> i32$`, implement karte hain, har ek ke liye Rust jo opaque types generate karta hai, woh alag-alag hote hain. Is samasya ka ek samadhan humne kai baar dekha hai: hum ek trait object ka istemal kar sakte hain, jaisa ki Listing 20-34 mein hai.

\<Listing number="20-34" caption="`Box&lt;dyn Fn&gt;` return karne wale functions dwara define kiye gaye closures ka `Vec&lt;T&gt;` banana taaki unka type ek hi ho"\>

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}

fn returns_initialized_closure() -> Box<dyn Fn(i32) -> i32> {
    let y = 10;
    Box::new(move |x| x + y)
}

fn main() {
    let mut closures = Vec::new();

    closures.push(returns_closure());
    closures.push(returns_initialized_closure());
}
```

\</Listing\>

Yeh code aaram se compile ho jaayega. Trait objects ke baare mein aur jaankari ke liye, Chapter 18 ka section "Using Trait Objects That Allow for Values of Different Types" dekhein.

Ab, chaliye macros par nazar daalte hain\!