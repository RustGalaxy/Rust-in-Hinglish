# Command Line Arguments Accept karna ⌨️

Chaliye, sabse pehle, **`cargo new`** se ek naya project banate hain. Hum apne project ka naam `minigrep` rakhenge, taaki ye us `grep` tool se alag ho jo aapke system par pehle se ho sakta hai.

```text
$ cargo new --bin minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

Pehla kaam hai `minigrep` ko uske do **command line arguments** accept karwana: **filename** aur ek **string** jise search karna hai. Matlab, hum apne program ko `cargo run` ke saath ek search string aur ek file ka path dekar run karna chahte hain, is tarah:

```text
$ cargo run searchstring example-filename.txt
```

Filhaal, `cargo new` se bana program usse diye gaye arguments ko process nahi kar sakta. `Crates.io` par kuch existing libraries hain jo command line arguments accept karne wale program banane mein help kar sakti hain, lekin kyunki aap abhi ye concept seekh rahe hain, to chaliye hum is capability ko khud se implement karte hain.

-----

### Argument Values ko Padhna 📖

`minigrep` ko usse diye gaye command line arguments ki values padhne ke liye, humein Rust ki **standard library** ka ek function chahiye, jo **`std::env::args`** hai. Ye function `minigrep` ko diye gaye command line arguments ka ek **iterator** return karta hai. Humne abhi iterators discuss nahi kiye hain, lekin filhaal, aapko do details janna zaroori hai: iterators values ki ek series produce karte hain, aur hum ek iterator par **`collect` method** ko call karke use ek collection, jaise ki ek vector, mein badal sakte hain.

Apne `minigrep` program ko command line arguments padhne aur unhe ek **vector** mein collect karne ke liye **Listing 12-1** ke code ka upyog karein.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    println!("{:?}", args);
}
```

\<span class="caption"\>Listing 12-1: Command line arguments ko ek vector mein collect karna aur unhein print karna\</span\>

Sabse pehle, hum `use std::env;` se `std::env` module ko scope mein laate hain taaki hum uske `args` function ka upyog kar saken. Dhyan dein ki `std::env::args` function do levels ke modules mein nested hai. Jab desirable function ek se zyaada module mein nested hota hai, to **parent module** ko scope mein laana conventional hai.

> ### `args` Function aur Invalid Unicode ⚠️
>
> Note karein ki agar koi argument **invalid Unicode** contain karta hai to `std::env::args` **panic** ho jaayega. Agar aapke program ko invalid Unicode wale arguments accept karne ki zaroorat hai, to **`std::env::args_os`** ka upyog karein.

`main` ki pehli line par, hum `env::args` ko call karte hain aur turant `collect` ka upyog karke **iterator** ko ek vector mein badal dete hain. Hum `collect` function ka upyog kayi tarah ki collections banane ke liye kar sakte hain, isliye hum explicitly `args` ka type **`Vec<String>`** annotate karte hain.

Finally, hum vector ko **debug formatter** `:?` ka upyog karke print karte hain. Chaliye program ko pehle bina arguments ke aur fir do arguments ke saath run karke dekhte hain:

```text
$ cargo run
--snip--
["target/debug/minigrep"]

$ cargo run needle haystack
--snip--
["target/debug/minigrep", "needle", "haystack"]
```

Gaur karein ki vector mein pehli value **`"target/debug/minigrep"`** hai, jo hamare **binary** ka naam hai.

-----

### Argument Values ko Variables mein Save karna 💾

Arguments ke vector ki value print karne se ye pata chala ki program command line arguments ko access kar sakta hai. Ab humein do arguments ki values ko variables mein save karne ki zaroorat hai taaki hum un values ka upyog program mein aage kar saken. Hum ise **Listing 12-2** mein karte hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,should_panic
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let filename = &args[2];

    println!("Searching for {}", query);
    println!("In file {}", filename);
}
```

\<span class="caption"\>Listing 12-2: Query argument aur filename argument ko hold karne ke liye variables banana\</span\>

Jaisa ki humne dekha, program ka naam `args[0]` par vector mein pehli value leta hai, isliye hum **index `1`** se shuru kar rahe hain. `minigrep` ka pehla argument wo string hai jise hum search kar rahe hain, isliye hum pehle argument ka reference `query` variable mein rakhte hain. Doosra argument **filename** hoga, isliye hum doosre argument ka reference `filename` variable mein rakhte hain.

Hum in variables ki values ko temporarily print karte hain taaki ye prove ho sake ki code wahi kaam kar raha hai jo hum usse karwana chahte hain. Chaliye is program ko arguments `test` aur `sample.txt` ke saath dobara run karte hain:

```text
$ cargo run test sample.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep test sample.txt`
Searching for test
In file sample.txt
```

Great, program kaam kar raha hai\! Jin arguments ki values humein chahiye, wo sahi variables mein save ho rahi hain. Baad mein hum kuch **error handling** add karenge.
