# Modularity aur Error Handling ko Improve karne ke liye Refactoring 🛠️

Apne program ko behtar banane ke liye, hum uske structure aur error handling se related char problems ko theek karenge.

1.  **`main` function ki zyaada zimmedariyan:** Hamara `main` function ab do kaam kar raha hai: arguments ko parse karna aur files ko kholna. Agar hum `main` mein code badhate rahein, to iski zimmedariyan aur badh jaayengi, jisse iske baare mein sochna, test karna, aur change karna mushkil ho jaayega. Ye behtar hai ki har function sirf ek kaam ke liye responsible ho.
2.  **Configuration variables ki clarity:** `query` aur `filename` jaise variables hamare program ke liye configuration variables hain. Jabki `f` aur `contents` jaise variables program ke logic ke liye hain. `main` ke lamba hone par, zyaada variables scope mein aayenge, jisse unke maqsad ko yaad rakhna mushkil hoga. Behtar hai ki configuration variables ko ek hi structure mein group kiya jaaye.
3.  **`expect` ke error messages:** Humne file kholne mein failure par `expect` ka upyog kiya hai, lekin error message sirf `file not found` print karta hai. File kholna kayi tareekon se fail ho sakta hai. Abhi, agar file exist karti hai lekin hamare paas use kholne ki permission nahi hai, to bhi hum `file not found` print karenge, jo user ko galat jaankari dega.
4.  **Poor error handling:** Hum `expect` ka upyog baar-baar kar rahe hain, aur agar user enough arguments nahi deta, to unhein `index out of bounds` error milegi jo problem ko theek se explain nahi karti. Ye behtar hoga ki saara error-handling code ek hi jagah ho taaki future maintainers ko sirf ek hi jagah dekhna pade aur hum users ke liye meaningful messages print kar saken.

Chaliye in char problems ko **refactoring** karke theek karte hain.

-----

### Binary Projects ke liye Concerns ka Separation 🎯

`main` function ko zyaada zimmedariyan dene ki organizational problem kayi binary projects mein common hai. Isliye, Rust community ne ek process develop kiya hai jo ek guideline ke roop mein kaam karta hai jab `main` bada hone lagta hai. Is process ke steps is tarah hain:

  * Apne program ko **`main.rs` aur `lib.rs`** mein split karein aur program ke logic ko **`lib.rs`** mein move karein.
  * Jab tak aapka command line parsing logic chota hai, wo `main.rs` mein reh sakta hai.
  * Jab command line parsing logic complicated hone lage, to usse `main.rs` se extract karke `lib.rs` mein move kar dein.

Is process ke baad `main` function ki zimmedariyan in tak limited honi chahiye:

  * Command line parsing logic ko arguments ke saath call karna.
  * Koi aur configuration set up karna.
  * `lib.rs` mein `run` function ko call karna.
  * Agar `run` error return karta hai to usko handle karna.

Is pattern ka maqsad **concerns ko separate** karna hai: `main.rs` program ko run karne ka kaam sambhalta hai, aur `lib.rs` saare logic ko handle karta hai. Kyunki aap `main` function ko directly test nahi kar sakte, ye structure aapko apne saare program logic ko `lib.rs` mein move karke test karne deta hai.

#### Argument Parser ko Extract karna ✂️

Hum arguments ko parse karne ki functionality ko ek function mein extract karenge jise `main` call karega, taaki hum command line parsing logic ko `src/lib.rs` mein move karne ke liye taiyar ho saken. **Listing 12-5** `main` ki nayi shuruat dikhati hai jo ek naye function `parse_config` ko call karta hai, jise hum filhaal *src/main.rs* mein hi define karenge.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let (query, filename) = parse_config(&args);

    // --snip--
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let filename = &args[2];

    (query, filename)
}
```

\<span class="caption"\>Listing 12-5: `main` se `parse_config` function ko extract karna\</span\>

Humne abhi bhi command line arguments ko ek vector mein collect kiya hai, lekin ab hum poore vector ko `parse_config` function ko pass kar rahe hain. `parse_config` function ye logic hold karta hai ki kaunsa argument kis variable mein jaayega aur values ko `main` ko wapas bhejta hai.

#### Configuration Values ko Group karna 📦

Ab hum `parse_config` function ko aur behtar banate hain. Abhi hum ek **tuple** return kar rahe hain, lekin ye is baat ka sign hai ki shayad hamare paas sahi **abstraction** nahi hai. `config` naam se pata chalta hai ki ye do values related hain. Hum in do values ko ek **struct** mein rakh sakte hain aur har field ko ek meaningful naam de sakte hain. Aisa karne se is code ke future maintainers ko ye samajhna aasan ho jaayega ki alag-alag values ek doosre se kaise related hain aur unka maqsad kya hai.

**Listing 12-6** `parse_config` function mein improvements dikhati hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,should_panic
# use std::env;
# use std::fs::File;
#
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = parse_config(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    let mut f = File::open(config.filename).expect("file not found");

    // --snip--
}

struct Config {
    query: String,
    filename: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let filename = args[2].clone();

    Config { query, filename }
}
```

\<span class="caption"\>Listing 12-6: `parse_config` ko `Config` struct ka instance return karne ke liye refactor karna\</span\>

Humne `Config` naam ka ek struct add kiya hai. `parse_config` ka signature ab indicate karta hai ki ye ek `Config` value return karta hai. Is case mein, hum **`clone` method** ko call karte hain, jisse `Config` instance ke liye data ki ek full copy ban jaati hai. `clone` ka upyog karna thoda inefficient ho sakta hai, lekin ye hamare code ko bahut straightforward banata hai.

-----

#### `Config` ke liye ek Constructor banana 🏗️

Ab jab `parse_config` function ka maqsad ek `Config` instance banana hai, to hum `parse_config` ko ek plain function se `new` naam ke function mein badal sakte hain jo `Config` struct se associated hai. **Listing 12-7** mein zaroori changes dikhaye gaye hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,should_panic
# use std::env;
#
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args);

    // --snip--
}

# struct Config {
#     query: String,
#     filename: String,
# }
#
// --snip--

impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let filename = args[2].clone();

        Config { query, filename }
    }
}
```

\<span class="caption"\>Listing 12-7: `parse_config` ko `Config::new` mein badalna\</span\>

Humne `parse_config` ka naam `new` kar diya hai aur ise ek `impl` block ke andar move kar diya hai, jo `new` function ko `Config` se associate karta hai. Ab hum `main` mein `Config::new` ko call kar sakte hain.

-----

### Error Handling ko Theek karna 🐛

Ab hum error handling ko theek karenge. `args` vector ke index 1 ya 2 ko access karne ki koshish karne par program panic ho jaata hai. **Listing 12-8** mein, hum `new` function mein ek check add karte hain jo verify karega ki slice enough lamba hai ya nahi.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
// --snip--
fn new(args: &[String]) -> Config {
    if args.len() < 3 {
        panic!("not enough arguments");
    }
    // --snip--
```

\<span class="caption"\>Listing 12-8: Arguments ki sankhya ke liye check add karna\</span\>

Ye output behtar hai, lekin `panic!` user ke liye zyada friendly nahi hai. Iski jagah, hum **`Result`** return kar sakte hain.

#### `panic!` ke bajaye `new` se `Result` return karna 🔄

**Listing 12-9** `Config::new` ka return value `Result` mein change kar diya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        Ok(Config { query, filename })
    }
}
```

\<span class="caption"\>Listing 12-9: `Config::new` se `Result` return karna\</span\>

Hamara `new` function ab `Ok` case mein `Config` ka instance aur `Err` case mein `&'static str` return karta hai. Humne `panic!` ke bajaye `Err` value return ki hai.

#### `Config::new` ko call karna aur Errors ko Handle karna 🚨

Error case ko handle karne aur user-friendly message print karne ke liye, humein `main` ko update karna hoga, jaisa **Listing 12-10** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
```

\<span class="caption"\>Listing 12-10: Naya `Config` banane mein fail hone par error code ke saath exit karna\</span\>

Humne `unwrap_or_else` method ka upyog kiya hai. Agar `Result` ek `Ok` value hai, to ye method `Ok` ke andar ki value return kar deta hai. Lekin agar value `Err` hai, to ye method `closure` ke code ko call karta hai, jismein hum error ko print karte hain aur **`process::exit(1)`** ko call karte hain. `process::exit` program ko turant rok deta hai aur ek non-zero exit status code return karta hai jo error state ko indicate karta hai.

-----

### Logic ko `main` se Extract karna ➡️

Ab hum program ke logic ko `run` naam ke function mein extract karenge. **Listing 12-11** mein extracted `run` function dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    run(config);
}

fn run(config: Config) {
    let mut f = File::open(config.filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");

    println!("With text:\n{}", contents);
}

// --snip--
```

\<span class="caption"\>Listing 12-11: Program ke baki logic ko contain karne wale `run` function ko extract karna\</span\>

#### `run` Function se Errors ko Return karna ↩️

`run` function ab `Result` return karega jab kuch galat hoga. **Listing 12-12** `run` ke signature aur body mein changes dikhati hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
use std::error::Error;

// --snip--

fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    println!("With text:\n{}", contents);

    Ok(())
}
```

\<span class="caption"\>Listing 12-12: `run` function ko `Result` return karne ke liye badalna\</span\>

Humne `run` ka return type **`Result<(), Box<Error>>`** kar diya hai. `Box<Error>` ek **trait object** hai jo batata hai ki function ek aisa type return karega jo `Error` trait ko implement karta hai. Humne `expect` calls ko `?` operator se replace kar diya hai.

`main` mein hum `if let` ka upyog karke `run` se return hone wali error ko handle karte hain.

```rust,ignore
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    if let Err(e) = minigrep::run(config) {
        println!("Application error: {}", e);

        process::exit(1);
    }
}
```

-----

### Code ko Library Crate mein Split karna 📚

Ab hum `minigrep` project ko *src/main.rs* file se split karke kuch code ko **`src/lib.rs`** file mein daalenge. Hum `run` function definition, relevant `use` statements, `Config` ki definition, aur `Config::new` function definition ko `src/lib.rs` mein move karenge. Hamein `Config` aur `run` ko **`pub`** banana hoga.

`src/main.rs` mein, hum library crate ko `extern crate minigrep` se import karenge aur `use minigrep::Config` se `Config` type ko scope mein layenge. `run` function ko ab `minigrep::run(config)` call kiya jaayega.

Ab hamara code zyaada modular hai aur errors ko handle karna aasan hai. Ab hum tests likhne ke liye taiyar hain\!
