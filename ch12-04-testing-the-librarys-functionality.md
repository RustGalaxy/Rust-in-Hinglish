# Test-Driven Development se Library ki Functionality Develop karna 📝

Ab jab humne logic ko **`src/lib.rs`** mein extract kar liya hai aur argument collecting aur error handling ko **`src/main.rs`** mein chhod diya hai, to hamare code ki core functionality ke liye tests likhna bahut aasan ho gaya hai. Hum alag-alag arguments ke saath functions ko directly call kar sakte hain aur return values ko check kar sakte hain, bina command line se apne binary ko call kiye.

Is section mein, hum **Test-Driven Development (TDD)** process ka upyog karke `minigrep` program mein searching logic add karenge. Ye software development technique in steps ko follow karti hai:

1.  Ek **failing test** likhein aur use run karke ensure karein ki wo us reason se fail ho raha hai jo aap expect karte hain.
2.  Naye test ko pass karne ke liye bas utna hi code likhein ya modify karein jitne ki zaroorat hai.
3.  Jo code abhi-abhi add ya change kiya hai usse **refactor** karein aur ensure karein ki tests abhi bhi pass ho rahe hain.
4.  Step 1 se repeat karein.

TDD code design ko bhi drive karne mein madad kar sakta hai. Code likhne se pehle test likhne se poore process mein high test coverage maintain karne mein madad milti hai. Hum us functionality ke implementation ko test drive karenge jo file contents mein **query string** ko search karegi aur matching lines ki list banayegi. Hum is functionality ko ek function mein add karenge jise **`search`** kehte hain.

-----

### Ek Failing Test Likhna ✍️

Ab humein `println!` statements ki zaroorat nahi hai, to chaliye unhein `src/lib.rs` aur `src/main.rs` se hata dete hain. Phir, `src/lib.rs` mein, hum ek `test` module banayenge jismein ek test function hoga. Test function us behavior ko specify karta hai jo hum `search` function se chahte hain: ye ek query aur text lega, aur text se sirf un lines ko return karega jismein query shamil hai. **Listing 12-15** mein ye test dikhaya gaya hai, jo abhi compile nahi hoga.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#      vec![]
# }
#
#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }
}
```

\<span class="caption"\>Listing 12-15: `search` function ke liye ek failing test banana jo hum chahte hain\</span\>

Hum is test ko run nahi kar sakte kyunki ye compile hi nahi hoga: `search` function abhi exist hi nahi karta\! To ab hum **Listing 12-16** mein `search` function ki ek definition add karenge jo hamesha ek khali vector return karti hai. Phir test compile hoga aur fail ho jaayega.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    vec![]
}
```

\<span class="caption"\>Listing 12-16: `search` function ki bas utni definition banana jisse hamara test compile ho sake\</span\>

Dhyan dein ki `search` ke signature mein humein ek explicit **lifetime `'a`** ki zaroorat hai, jise `contents` argument aur return value ke saath use kiya gaya hai. Hum Rust ko batate hain ki `search` function se return kiya gaya data `contents` argument mein pass kiye gaye data jitna lamba live karega.

Ab chaliye test ko run karte hain:

```text
$ cargo test
--snip--
running 1 test
test test::one_result ... FAILED

failures:

---- test::one_result stdout ----
        thread 'test::one_result' panicked at 'assertion failed: `(left ==
right)`
left: `["safe, fast, productive."]`,
right: `[]`)', src/lib.rs:48:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Bahut accha, test fail ho gaya, bilkul jaisa humne expect kiya tha. Chaliye ab test ko pass karne ke liye code likhte hain\!

-----

### Code Likhna taaki Test Pass ho ✅

Filhaal, hamara test fail ho raha hai kyunki hum hamesha ek khali vector return kar rahe hain. Isko theek karne aur `search` ko implement karne ke liye, hamare program ko ye steps follow karne honge:

  * Contents ki har line par **iterate** karein.
  * Check karein ki line mein hamari query string shamil hai ya nahi.
  * Agar haan, to use return kiye ja rahe values ki list mein add karein.
  * Agar nahi, to kuch na karein.
  * Matching results ki list return karein.

Rust mein strings ke liye line-by-line iteration ko handle karne ke liye ek helpful method hai jiska naam **`lines`** hai. **Listing 12-17** mein, `lines` method ka upyog dikhaya gaya hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        // do something with line
    }
}
```

\<span class="caption"\>Listing 12-17: `contents` ki har line par iterate karna\</span\>

Ab hum check karenge ki current line mein hamari query string hai ya nahi. Strings mein ek helpful method hai jiska naam **`contains`** hai. **Listing 12-18** mein `search` function mein `contains` method ka upyog add karte hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        if line.contains(query) {
            // do something with line
        }
    }
}
```

\<span class="caption"\>Listing 12-18: Ye dekhne ki functionality add karna ki line mein `query` mein di gayi string hai ya nahi\</span\>

Matching lines ko store karne ke liye, hum **`for` loop** se pehle ek mutable vector bana sakte hain aur `push` method ko call karke vector mein ek `line` store kar sakte hain. Loop ke baad, hum vector ko return kar denge, jaisa **Listing 12-19** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

\<span class="caption"\>Listing 12-19: Matching lines ko store karna taaki hum unhein return kar saken\</span\>

Ab `search` function sirf un lines ko return karega jismein `query` shamil hai, aur hamara test pass ho jaana chahiye. Chaliye test ko run karte hain:

```text
$ cargo test
--snip--
running 1 test
test test::one_result ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Hamara test pass ho gaya, to hum jaante hain ki ye kaam karta hai\!

-----

#### `run` Function mein `search` Function ka Upyog ⚙️

Ab jab `search` function kaam kar raha hai aur test ho chuka hai, humein `run` function se `search` ko call karne ki zaroorat hai. `run` function `search` se return kiye gaye har line ko print karega.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
pub fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    for line in search(&config.query, &contents) {
        println!("{}", line);
    }

    Ok(())
}
```

`run` se `search` ko call karne par, hum Emily Dickinson ki poem mein "frog" word ke liye search kar sakte hain.

```text
$ cargo run frog poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.38 secs
     Running `target/debug/minigrep frog poem.txt`
How public, like a frog
```

Badhiya\! Ab hum "body" word ke liye search karte hain.

```text
$ cargo run body poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep body poem.txt`
I’m nobody! Who are you?
Are you nobody, too?
How dreary to be somebody!
```

Shandar\! Humne ek classic tool ka apna mini version bana liya hai aur applications ko structure karne ke baare mein bahut kuch seekha hai. Ab hum is project ko poora karne ke liye, environment variables ke saath kaam karna aur standard error par print karna dikhayenge, jo command line programs likhte samay bahut useful hote hain.
