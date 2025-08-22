# Environment Variables ke saath Kaam karna 🌐

Hum `minigrep` mein ek extra feature add karenge: **case-insensitive searching** ka option jise user ek **environment variable** ke zariye on kar sakta hai. Hum TDD process ko follow karte hue aage badhenge.

### Case-Insensitive `search` Function ke liye Ek Failing Test Likhna ✍️

Hum ek naya `search_case_insensitive` function add karna chahte hain jise hum tab call karenge jab environment variable on ho. Pehla step phir se ek failing test likhna hai. Hum `src/lib.rs` mein `one_result` test ko `case_sensitive` rename karenge aur ek naya test **`case_insensitive`** add karenge, jaisa **Listing 12-20** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn case_sensitive() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Duct tape.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }

    #[test]
    fn case_insensitive() {
        let query = "rUsT";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Trust me.";

        assert_eq!(
            vec!["Rust:", "Trust me."],
            search_case_insensitive(query, contents)
        );
    }
}
```

\<span class="caption"\>Listing 12-20: Naye case-insensitive function ke liye ek failing test add karna\</span\>

`case_sensitive` test ko is tarah change kiya gaya hai taaki hum ensure kar saken ki hum existing functionality ko accidental break na karein. Naya `case_insensitive` test `"rUsT"` ko query ke roop mein use karta hai aur `"Rust:"` aur `"Trust me."` dono lines ko match karna chahiye. Ye hamara **failing test** hai, aur ye compile nahi hoga kyunki humne `search_case_insensitive` function ko abhi tak define nahi kiya hai.

-----

### `search_case_insensitive` Function ko Implement karna 🛠️

`search_case_insensitive` function, jaisa **Listing 12-21** mein dikhaya gaya hai, `search` function ke jaisa hi hoga. Fark sirf itna hai ki hum `query` aur har `line` ko **lowercase** kar denge taaki input arguments ka case kuch bhi ho, comparison ke samay wo same case mein honge.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}
```

\<span class="caption"\>Listing 12-21: `search_case_insensitive` function ko define karna taaki comparison se pehle query aur line ko lowercase kiya ja sake\</span\>

Pehle, hum `query` string ko lowercase karte hain. `to_lowercase` ek nayi `String` banata hai, isliye humein `contains` method ko call karte samay **ampersand (`&`)** add karne ki zaroorat hai, kyunki `contains` ka signature ek string slice leta hai. Phir, hum har `line` ko lowercase karte hain. Ab tests pass hone chahiye:

```text
running 2 tests
test test::case_insensitive ... ok
test test::case_sensitive ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

-----

### Environment Variable ke liye Check karna 🌐

Ab hum `Config` struct mein ek naya field `case_sensitive` add karte hain, jismein ek **Boolean** value hogi. `run` function is field ko check karke decide karega ki `search` ya `search_case_insensitive` ko call karna hai, jaisa **Listing 12-22** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# use std::error::Error;
# use std::fs::File;
# use std::io::prelude::*;
#
# fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#      vec![]
# }
#
# pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#      vec![]
# }
#
# struct Config {
#     query: String,
#     filename: String,
#     case_sensitive: bool,
# }
#
pub fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    let results = if config.case_sensitive {
        search(&config.query, &contents)
    } else {
        search_case_insensitive(&config.query, &contents)
    };

    for line in results {
        println!("{}", line);
    }

    Ok(())
}
```

\<span class="caption"\>Listing 12-22: `config.case_sensitive` ki value ke aadhar par `search` ya `search_case_insensitive` ko call karna\</span\>

Environment variable ke liye check karne ke liye, hum `Config::new` mein **`std::env::var`** function ka use karenge. Ye function ek `Result` return karta hai jo environment variable set hone par `Ok` variant ya set na hone par `Err` variant hota hai. Hum **`is_err` method** ka upyog karte hain taaki ye check kar saken ki variable set nahi hai, jiska matlab case-sensitive search hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
use std::env;
# struct Config {
#     query: String,
#     filename: String,
#     case_sensitive: bool,
# }

// --snip--

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

\<span class="caption"\>Listing 12-23: `CASE_INSENSITIVE` naam ke environment variable ke liye check karna\</span\>

Ab hum `minigrep` ko try karte hain. Pehle, bina environment variable set kiye:

```text
$ cargo run to poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep to poem.txt`
Are you nobody, too?
How dreary to be somebody!
```

Ab, `CASE_INSENSITIVE` ko set karke:

```text
$ CASE_INSENSITIVE=1 cargo run to poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep to poem.txt`
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Bahut badhiya\! Ab hamara `minigrep` program case-insensitive searching bhi kar sakta hai. Ab aap jaante hain ki command line arguments ya environment variables ka use karke options ko kaise manage kiya jaata hai.

-----
