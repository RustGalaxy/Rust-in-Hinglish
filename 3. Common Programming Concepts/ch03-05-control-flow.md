## Control Flow

Zyadatar programming languages mein, yeh decide karna ki koi code chalana hai ya nahi, aur us code ko bar-bar chalate rehna jab tak ek condition true hai, ek basic building block hai. Rust code mein execution ke flow ko control karne ke liye sabse common constructs **`if` expressions** aur **loops** hain.

-----

### `if` Expressions 

Ek `if` expression aapko conditions ke hisab se apne code ko branch karne ki permission deta hai. Aap ek condition dete hain aur kehte hain, "Agar yeh condition meet hoti hai, toh yeh code block chalao. Agar condition meet nahi hoti, toh mat chalao."

`if` expressions ko explore karne ke liye, *branches* naam ka ek naya project banayein. `src/main.rs` file mein, neeche diya gaya code likhein:

Filename: src/main.rs

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

Sabhi `if` expressions **`if` keyword** se shuru hote hain, jiske baad ek condition hoti hai. Is case mein, condition check karti hai ki `number` variable ki value 5 se kam hai ya nahi. Jo code block hum execute karna chahte hain agar condition true hai, woh turant condition ke baad curly brackets ke andar rakha jata hai. `if` expressions mein conditions se jude code blocks ko kabhi-kabhi **arms** bhi kehte hain, jaise Chapter 2 mein `match` expressions ke arms.

Optionally, hum ek **`else` expression** bhi shamil kar sakte hain, jaisa humne yahaan kiya hai, taki program ko ek alternative code block de sakein, agar condition false ho. Agar aap `else` expression nahi dete hain aur condition false hoti hai, toh program bas `if` block ko chhod dega aur agle code par chala jayega.

Is code ko run karke dekhein; aapko neeche diya gaya output milna chahiye:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
condition was true
```

Chaliye `number` ki value ko badalkar ek aisi value karte hain jo condition ko `false` bana de, phir dekhte hain kya hota hai:

```rust,ignore
let number = 7;
```

Program ko dobara run karein, aur output dekhein:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
condition was false
```

Yeh dhyan rakhna bhi zaroori hai ki is code mein condition ek **`bool`** honi chahiye. Agar condition `bool` nahi hai, toh humein error milegi. Jaise, neeche diye gaye code ko run karne ki koshish karein:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let number = 3;

    if number {
        println!("number was three");
    }
}
```

Is baar `if` condition `3` ki value mein evaluate hoti hai, aur Rust error throw karta hai:

```text
error[E0308]: mismatched types
 --> src/main.rs:4:8
  |
4 |       if number {
  |          ^^^^^^ expected bool, found integral variable
  |
  = note: expected type `bool`
              found type `{integer}`
```

Error batata hai ki Rust ko ek `bool` chahiye tha, lekin usse ek integer mila. Ruby aur JavaScript jaisi languages se alag, Rust non-Boolean types ko automatically Boolean mein convert karne ki koshish nahi karega. Aapko explicit hona padega aur hamesha `if` ko uski condition ke liye ek Boolean hi provide karna hoga. Agar hum chahte hain ki `if` code block tabhi chale jab number `0` ke barabar na ho, toh hum `if` expression ko is tarah badal sakte hain:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let number = 3;

    if number != 0 {
        println!("number was something other than zero");
    }
}
```

Is code ko run karne par `number was something other than zero` print hoga.

#### Handling Multiple Conditions with `else if` (Multiple Conditions ko `else if` se Handle karna)

Aap `if` aur `else` ko `else if` expression mein jodkar multiple conditions rakh sakte hain. Jaise:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

Is program ke char possible paths hain. Isse run karne ke baad, aapko neeche diya gaya output dekhna chahiye:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
number is divisible by 3
```

Jab yeh program execute hota hai, toh yeh har `if` expression ko ek-ek karke check karta hai aur pehle body ko execute karta hai jiske liye condition true hoti hai. Dhyan dein ki 6, 2 se divisible hone ke bawajood, humein `number is divisible by 2` ka output nahi dikhta, na hi `else` block se `number is not divisible by 4, 3, or 2` text dikhta hai. Iska reason yeh hai ki Rust sirf pehli true condition ke liye block ko execute karta hai, aur jab ek mil jata hai, toh baaki ko check bhi nahi karta.

Bahut zyada `else if` expressions ka use karne se aapka code messy ho sakta hai, isliye agar aapke paas ek se zyada hain, toh aapko apne code ko refactor karna pad sakta hai. Chapter 6 mein, hum in cases ke liye Rust ke ek powerful branching construct, `match`, ke bare mein discuss karenge.

-----

#### Using `if` in a `let` Statement (if ko `let` Statement mein Upyog karna)

Kyunki `if` ek expression hai, hum isse `let` statement ke right side mein use kar sakte hain, jaisa Listing 3-2 mein hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

\<span class="caption"\>Listing 3-2: `if` expression ke result ko ek variable mein assign karna\</span\>

`number` variable ko `if` expression ke outcome ke aadhar par ek value assign ki jayegi. Is code ko run karke dekhein kya hota hai:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/branches`
The value of number is: 5
```

Yaad rakhein ki code blocks un mein aakhri expression mein evaluate hote hain, aur numbers khud bhi expressions hote hain. Is case mein, poore `if` expression ki value is baat par depend karti hai ki kaun sa block execute hota hai. Iska matlab hai ki har arm se jo values result ho sakti hain, unka type same hona chahiye; Listing 3-2 mein, `if` arm aur `else` arm dono ke results `i32` integers the. Agar types mismatched hain, jaisa neeche diye gaye example mein hai, toh humein error milegi:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let condition = true;

    let number = if condition {
        5
    } else {
        "six"
    };

    println!("The value of number is: {}", number);
}
```

Jab hum is code ko compile karne ki koshish karte hain, toh humein error milegi. `if` aur `else` arms ke value types incompatible hain, aur Rust program mein exactly batata hai ki problem kahan hai:

```text
error[E0308]: if and else have incompatible types
 --> src/main.rs:4:18
  |
4 |         let number = if condition {
  | __________________^
5 | |          5
6 | |      } else {
7 | |          "six"
8 | |      };
  | |_____^ expected integral variable, found &str
  |
  = note: expected type `{integer}`
              found type `&str`
```

`if` block ka expression ek integer mein evaluate hota hai, aur `else` block ka expression ek string mein evaluate hota hai. Yeh kaam nahi karega kyunki variables ka ek hi type hona chahiye. Rust ko compile time par pata hona chahiye ki `number` variable ka type kya hai, taaki woh compile time par verify kar sake ki uske type har jagah valid hai jahan hum `number` ka upyog karte hain. Agar `number` ka type sirf runtime par determine hota, toh Rust aisa nahi kar pata; compiler aur bhi complex ho jata aur code ke baare mein kam guarantees deta.

-----

### Repetition with Loops (Loops ke saath Repetition)

Ek code block ko ek se zyada baar execute karna aksar useful hota hai. Is kaam ke liye, Rust kai **loops** provide karta hai. Ek loop loop body ke andar ke code ko aakhir tak chalata hai aur phir turant shuruaat se wapas shuru ho jata hai. Loops ke saath experiment karne ke liye, aaiye *loops* naam ka ek naya project banate hain.

Rust mein teen tarah ke loops hain: **`loop`**, **`while`**, aur **`for`**. Aaiye har ek ko try karte hain.

#### Repeating Code with `loop` (`loop` se Code Repeat karna)

`loop` keyword Rust ko batata hai ki ek code block ko bar-bar chalao, hamesha ke liye ya jab tak aap use explicitly rokte nahi hain.

Example ke liye, apne *loops* directory mein `src/main.rs` file ko is tarah badlein:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    loop {
        println!("again!");
    }
}
```

Jab hum is program ko run karte hain, toh humein `again!` bar-bar print hota hua dikhega jab tak hum program ko manually nahi rokte. Zyadatar terminals ek keyboard shortcut, **`ctrl-c`**, support karte hain, ek program ko rokne ke liye jo ek continual loop mein phasa ho. Isse try karke dekhein:

```text
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29 secs
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

`^C` symbol wahan hai jahan aapne `ctrl-c` dabaya. Aapko `^C` ke baad `again!` word print hota hua dikh bhi sakta hai aur nahi bhi, is baat par depend karta hai ki jab use halt signal mila tab code loop mein kahan tha.

Fortunately, Rust ek aur, zyada reliable tarika provide karta hai loop se bahar nikalne ka. Aap loop ke andar **`break` keyword** rakh sakte hain, taaki program ko pata chal sake ki loop ko kab rokna hai. Yaad karein ki humne yeh guessing game mein kiya tha (Chapter 2 ke "Quitting After a Correct Guess" section mein) program se bahar nikalne ke liye jab user sahi number guess karke game jeet gaya tha.

-----

#### Conditional Loops with `while` (`while` se Conditional Loops)

Ek loop ke andar ek condition ko evaluate karna aksar useful hota hai. Jab tak condition true hai, loop chalta hai. Jab condition true hona band ho jati hai, program `break` call karta hai, loop ko rok deta hai. Is tarah ke loop ko `loop`, `if`, `else`, aur `break` ke combination se implement kiya jaa sakta hai.

Lekin, yeh pattern itna common hai ki Rust mein iske liye ek built-in language construct hai, jise **`while` loop** kehte hain. Listing 3-3 `while` ka upyog karta hai: program teen baar loop karta hai, har baar ginti kam karta hai, aur phir, loop ke baad, ek aur message print karta hai aur exit ho jata hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number = number - 1;
    }

    println!("LIFTOFF!!!");
}
```

\<span class="caption"\>Listing 3-3: Ek `while` loop ka upyog code chalane ke liye jab tak ek condition true rehti hai\</span\>

Yeh construct us nesting ko eliminate karta hai jo `loop`, `if`, `else`, aur `break` ka upyog karne par zaroori hoti, aur yeh zyada clear hai. Jab tak ek condition true rehti hai, code chalta hai; otherwise, yeh loop se bahar nikal jata hai.

-----

#### Looping Through a Collection with `for` (`for` se Collection mein Loop karna)

Aap `while` construct ka upyog kisi collection, jaise ki ek array, ke elements par loop karne ke liye kar sakte hain. Jaise, aaiye Listing 3-4 ko dekhte hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index = index + 1;
    }
}
```

\<span class="caption"\>Listing 3-4: `while` loop ka upyog karke ek collection ke har element par loop karna\</span\>

Yahaan, code array ke elements ke through ginti karta hai. Yeh index `0` se shuru hota hai, aur phir tab tak loop karta hai jab tak yeh array mein final index tak nahi pahunch jata (yaani, jab `index < 5` ab true nahi rehta). Is code ko run karne par array ka har element print hoga:

```text
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
     Running `target/debug/loops`
the value is: 10
the value is: 20
the value is: 30
the value is: 40
the value is: 50
```

Array ki sabhi paanch values terminal mein appear hoti hain, jaisa expected tha. Halanki `index` ek point par `5` ki value tak pahunch jayega, loop array se chhati value fetch karne ki koshish karne se pehle hi ruk jata hai.

Lekin yeh approach error prone hai; agar index length galat ho toh hum program ko panic karwa sakte hain. Yeh slow bhi hai, kyunki compiler har iteration par har element par conditional check karne ke liye runtime code add karta hai.

Ek aur concise alternative ke roop mein, aap ek **`for` loop** ka upyog kar sakte hain aur ek collection mein har item ke liye kuch code execute kar sakte hain. Ek `for` loop Listing 3-5 mein diye gaye code jaisa dikhta hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```

\<span class="caption"\>Listing 3-5: `for` loop ka upyog karke ek collection ke har element par loop karna\</span\>

Jab hum is code ko run karte hain, toh humein Listing 3-4 jaisa hi output dikhega. Zyada important baat yeh hai ki humne ab code ki safety badha di hai aur un bugs ke chance ko eliminate kar diya hai jo array ke aakhir se aage jane ya itna aage na jane se ho sakte hain ki kuch items chhoot jayein.

Example ke liye, Listing 3-4 ke code mein, agar aapne `a` array se ek item hata diya, lekin `while index < 4` karne ke liye condition ko update karna bhool gaye, toh code panic kar jayega. `for` loop ka upyog karke, aapko kisi aur code ko badalne ki zaroorat nahi padegi agar aap array mein values ki sankhya badalte hain.

`for` loops ki safety aur conciseness unhe Rust mein sabse commonly used loop construct banati hai. Yahan tak ki un situations mein bhi jahan aap kisi code ko ek certain number of times chalana chahte hain, jaise countdown example jisme Listing 3-3 mein `while` loop ka upyog kiya gaya tha, zyadatar Rustaceans ek `for` loop ka upyog karenge. Aisa karne ka tarika ek **`Range`** ka upyog karna hoga, jo ek standard library dwara provide kiya gaya type hai jo ek number se shuru hone wali aur doosre number se pehle khatam hone wali sequence mein sabhi numbers generate karta hai.

Yahaan dekhein ki countdown ek `for` loop aur ek aur method, **`rev`** (jo range ko reverse karta hai), ka upyog karke kaisa dikhega:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

Yeh code thoda behtar hai, hai na?

-----

### Summary 

Aapne kar dikhaya\! Yeh ek bada chapter tha: aapne variables, scalar aur compound data types, functions, comments, `if` expressions, aur loops ke bare mein sikha\! Agar aap is chapter mein discuss kiye gaye concepts ke saath practice karna chahte hain, toh neeche diye gaye programs banane ki koshish karein:

  * Fahrenheit aur Celsius ke beech temperatures ko convert karein.
  * nth Fibonacci number generate karein.
  * Christmas carol "The Twelve Days of Christmas" ke lyrics print karein, gaane mein repetition ka fayda uthate hue.

Jab aap aage badhne ke liye ready honge, hum Rust mein ek aise concept ke baare mein baat karenge jo aam taur par doosri programming languages mein nahi hota: ownership.

You can learn more about if-else-if structures in programming with this video: [Multiple (If-else-If) Structure with Example](https://www.youtube.com/watch?v=EzdEuOvutu0).
http://googleusercontent.com/youtube_content/0
