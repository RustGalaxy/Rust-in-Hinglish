## Recoverable Errors with `Result`

Most errors itne serious nahi hote ki program poori tarah se ruk jaaye. Jab ek function fail hota hai, to uska reason aksar aisa hota hai jise aap aasaani se samajh sakte hain aur respond kar sakte hain. Jaise, agar aap ek file kholne ki koshish karte hain aur wo file exist nahi karti, to aap process ko terminate karne ke bajaye file create karna chahenge.

-----

### `Result` Enum ka Upyog ⚙️

**Chapter 2** mein, humne dekha tha ki `Result` enum do variants ke saath define kiya gaya hai, `Ok` aur `Err`:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` aur `E` generic type parameters hain. Abhi aapko itna jaanna hai ki `T` success case mein `Ok` variant ke andar return hone wali value ke type ko represent karta hai, aur `E` failure case mein `Err` variant ke andar return hone wali error ke type ko represent karta hai.

Chaliye, ek aise function ko call karte hain jo `Result` value return karta hai, kyunki wo fail ho sakta hai. **Listing 9-3** mein, hum ek file kholne ki koshish kar rahe hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```

\<span class="caption"\>Listing 9-3: Ek file kholna\</span\>

Agar hum `let f: u32 = File::open("hello.txt");` likhkar compile karein, to compiler humein batayega ki types match nahi kar rahe hain. Error message ye hoga:

```text
error[E0308]: mismatched types
 --> src/main.rs:4:18
  |
4 |     let f: u32 = File::open("hello.txt");
  |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
`std::result::Result`
  |
  = note: expected type `u32`
             found type `std::result::Result<std::fs::File, std::io::Error>`
```

Ye humein batata hai ki `File::open` function ka return type **`Result<std::fs::File, std::io::Error>`** hai. Iska matlab hai ki call `Ok` ho sakta hai aur ek `File` handle return kar sakta hai, ya fail ho sakta hai aur ek `io::Error` return kar sakta hai.

**Listing 9-4** ek tarika dikhata hai ki `Result` ko **`match` expression** se kaise handle kiya jaaye.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
}
```

\<span class="caption"\>Listing 9-4: `Result` variants ko handle karne ke liye `match` expression ka upyog\</span\>

Yahan hum Rust ko batate hain ki jab result `Ok` ho, to inner `file` value return kare aur us file handle ko variable `f` ko assign karein. Agar `Err` ho, to hum `panic!` macro ko call kar dete hain.

### Alag-alag Errors par Matching 🧩

**Listing 9-5** mein, hum different failure reasons ke liye alag actions lete hain. Agar file exist nahi karti, to hum use create karna chahte hain. Doosre reasons ke liye, hum `panic!` hi karte hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!(
                        "Tried to create file but there was a problem: {:?}",
                        e
                    )
                },
            }
        },
        Err(error) => {
            panic!(
                "There was a problem opening the file: {:?}",
                error
            )
        },
    };
}
```

\<span class="caption"\>Listing 9-5: Alag-alag tarah ki errors ko handle karna\</span\>

Yahan, `if error.kind() == ErrorKind::NotFound` ek **match guard** hai. Ye ek extra condition hai jo `match` arm ke pattern ko aur refine karti hai. Hum **`ref`** ka upyog karte hain taaki `error` ko guard condition mein **move** na kiya jaaye.

### Error par Panic ke liye Shortcuts: `unwrap` aur `expect` 💥

`match` ka upyog karna kaafi lamba ho sakta hai. `Result<T, E>` type mein kayi helper methods hote hain. Unmein se ek **`unwrap`** hai, jo bilkul **Listing 9-4** mein likhe gaye `match` expression ki tarah kaam karta hai. Agar `Result` `Ok` variant hai, to `unwrap` `Ok` ke andar ki value return karega. Agar `Result` `Err` hai, to `unwrap` hamare liye `panic!` macro ko call kar dega.

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

Iske jaisa hi ek aur method **`expect`** hai, jo humein `panic!` error message choose karne deta hai. `expect` ka use karne se intention clear ho jaati hai aur panic ke source ko track karna aasaan ho jaata hai.

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

Isse error message mein `Failed to open hello.txt` shuruat mein dikhega.

### Errors ko Propagate karna ➡️

Jab aap ek function likh rahe hote hain jismein kuch aisa call hota hai jo fail ho sakta hai, to error ko us function ke andar handle karne ke bajaye, aap usse **calling code** ko return kar sakte hain, jise **propagating the error** kehte hain. Ye calling code ko zyaada control deta hai. **Listing 9-6** mein, hum ek function banate hain jo file se username padhta hai. Agar file exist nahi karti ya read nahi ho sakti, to ye function un errors ko call karne wale code ko return kar deta hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

\<span class="caption"\>Listing 9-6: `match` ka upyog karke errors ko calling code ko return karna\</span\>

Ye pattern Rust mein itna common hai ki Rust isko aasan banane ke liye **question mark operator `?`** provide karta hai.

### `?` Operator se Errors Propagate karne ka Shortcut ⚡

**Listing 9-7** `read_username_from_file` ka implementation dikhata hai jo `?` operator ka use karta hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

\<span class="caption"\>Listing 9-7: `?` operator ka upyog karke errors ko calling code ko return karna\</span\>

`Result` value ke baad `?` `match` expression ki tarah kaam karta hai. Agar value `Ok` hai, to inner value return ho jaati hai. Agar value `Err` hai, to `Err` ke andar ki value poore function se return ho jaati hai. `?` operator bahut saare **boilerplate** ko hatata hai aur code ko simpler banata hai.

Aap method calls ko `?` ke baad chain bhi kar sakte hain, jaisa **Listing 9-8** mein hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

\<span class="caption"\>Listing 9-8: `?` operator ke baad method calls ko chain karna\</span\>

-----

### `?` Operator sirf un Functions mein use ho sakta hai jo `Result` return karte hain ❗

`?` operator ka upyog sirf un functions mein ho sakta hai jinka return type **`Result`** ho. Kyunki ye `match` expression ki tarah kaam karta hai jismein `return Err(e)` hota hai. Agar aap `main` function jaise functions mein `?` ka upyog karte hain, to aapko error milegi.

```rust,ignore
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
```

Compile karne par, error `E0277` aayegi jo batati hai ki `?` operator sirf `Result` return karne wale functions mein use ho sakta hai.

Ab jab humne `panic!` call karne ya `Result` return karne ke baare mein baat kar li hai, to aage hum decide karenge ki kis case mein kaunsa tareeka sahi hai.
