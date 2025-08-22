# File ko Padhna 📖

Ab hum `filename` command line argument mein specify ki gayi file ko padhne ki functionality add karenge. Sabse pehle, humein test karne ke liye ek sample file chahiye. **Listing 12-3** mein Emily Dickinson ki ek poem hai, jo hamare liye ek accha test case rahegi. Apne project ke root level par **`poem.txt`** naam ki ek file banayein, aur usmein ye poem daalein.

\<span class="filename"\>Filename: poem.txt\</span\>

```text
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us - don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

\<span class="caption"\>Listing 12-3: Emily Dickinson ki ek poem ek accha test case hai\</span\>

Text daalne ke baad, *src/main.rs* ko edit karein aur file ko kholne ke liye code add karein, jaisa **Listing 12-4** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,should_panic
use std::env;
use std::fs::File;
use std::io::prelude::*;

fn main() {
#     let args: Vec<String> = env::args().collect();
#
#     let query = &args[1];
#     let filename = &args[2];
#
#     println!("Searching for {}", query);
    // --snip--
    println!("In file {}", filename);

    let mut f = File::open(filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");

    println!("With text:\n{}", contents);
}
```

\<span class="caption"\>Listing 12-4: Second argument dwara specify ki gayi file ke contents ko padhna\</span\>

Sabse pehle, hum standard library ke relevant parts ko scope mein laane ke liye kuch aur `use` statements add karte hain: files handle karne ke liye `std::fs::File`, aur `std::io::prelude::*` mein I/O ke liye useful traits hote hain. `std::io` module ka apna ek prelude hai, jise humein explicitly `use` statement se add karna padta hai.

`main` mein, humne teen statements add kiye hain: pehle, hum `File::open` function ko call karke aur `filename` variable ki value pass karke file ka ek **mutable handle** paate hain. Doosra, hum `contents` naam ka ek variable banate hain aur use ek mutable, khali `String` par set karte hain. Ye file ka content hold karega. Teesra, hum apne file handle par `read_to_string` call karte hain aur `contents` ka ek mutable reference as an argument pass karte hain.

In lines ke baad, humne ek temporary `println!` statement add kiya hai jo file ko padhne ke baad `contents` ki value ko print karta hai.

Chaliye is code ko run karte hain. Pehla command line argument koi bhi string (kyunki humne abhi searching part implement nahi kiya hai) aur doosra argument *poem.txt* file hoga:

```text
$ cargo run the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us - don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

Badhiya\! Code ne file ke contents ko padha aur print kiya. Lekin is code mein kuch flaws hain. `main` function ki multiple zimmedariyan hain: generally, functions zyaada clear aur maintain karne mein aasan hote hain agar har function sirf ek hi idea ke liye responsible ho. Doosri problem ye hai ki hum errors ko theek se handle nahi kar rahe hain. Abhi program chota hai, to ye flaws ek badi problem nahi hain, lekin program ke badhne par, unhein theek karna mushkil hoga. Program develop karte samay shuruat mein hi **refactoring** shuru karna ek acchi practice hai. Hum aage wahi karenge.
