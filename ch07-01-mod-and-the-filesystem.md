# Module (Mod) aur Filesystem

Hum apna module ka example Cargo ke saath ek naya project banakar shuru karenge, lekin binary crate banane ke bajaye, hum ek **library crate** banayenge: ek aisa project jise dusre log apne project mein dependency ke roop mein shamil kar sakte hain. Example ke liye, Chapter 2 mein discuss kiya gaya `rand` crate ek library crate hai jise humne guessing game project mein ek dependency ke roop mein upyog kiya tha.

-----

Hum ek aisi library ka skeleton banayenge jo kuch general networking functionality provide karti hai; hum modules aur functions ke organization par dhyan denge, lekin hum function bodies mein kya code jayega uski chinta nahi karenge. Hum apni library ko **`communicator`** kahenge. Ek library banane ke liye, `--bin` option ke bajaye `--lib` option pass karein:

```text
$ cargo new communicator --lib
$ cd communicator
```

Dhyan dein ki Cargo ne **src/main.rs** ke bajaye **src/lib.rs** generate kiya. **src/lib.rs** ke andar humein neeche diya gaya code milega:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Cargo hamari library ko shuru karne mein madad karne ke liye ek example test create karta hai, na ki "Hello, world\!" binary jo hum `--bin` option upyog karte samay pate hain. Hum is chapter mein aage "Using `super` to Access a Parent Module" section mein `#[]` aur `mod tests` syntax ko dekhenge, lekin abhi ke liye, is code ko **src/lib.rs** ke bottom mein chhod dein.

Kyunki hamare paas **src/main.rs** file nahi hai, `cargo run` command ko execute karne ke liye kuch bhi nahi hai. Isliye, hum apni library crate ke code ko compile karne ke liye **`cargo build`** command ka upyog karenge.

Hum aapki library ke code ko organize karne ke alag-alag options ko dekhenge jo alag-alag situations mein suitable honge, jo code ke irade par nirbhar karta hai.

-----

### Module Definitions

Apni `communicator` networking library ke liye, hum sabse pehle **`network`** naam ka ek module define karenge jismein **`connect`** naam ke ek function ki definition shamil hai. Rust mein har module definition `mod` keyword se shuru hoti hai. Is code ko **src/lib.rs** file ke shuruat mein, test code ke upar add karein:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
mod network {
    fn connect() {
    }
}
```

`mod` keyword ke baad, hum module ka naam, `network`, aur phir curly brackets mein ek code block dalte hain. Is block ke andar sab kuch `network` namespace ke andar hai. Is case mein, hamare paas ek single function, `connect`, hai. Agar hum is function ko `network` module ke bahar code se call karna chahte the, toh humein module specify karna padta aur `::` jaisa namespace syntax upyog karna padta hai, jaise: `network::connect()`.

Hum ek hi **src/lib.rs** file mein ek se zyada modules, side by side, bhi rakh sakte hain. Example ke liye, `connect` naam ka function rakhne wale ek `client` module ko bhi rakhne ke liye, hum use Listing 7-1 mein dikhaye gaye tarike se add kar sakte hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
mod network {
    fn connect() {
    }
}

mod client {
    fn connect() {
    }
}
```

\<span class="caption"\>Listing 7-1: `network` module aur `client` module ko `src/lib.rs` mein side by side define kiya gaya hai\</span\>

Ab hamare paas ek `network::connect` function aur ek `client::connect` function hai. Inka functionality poori tarah se alag ho sakta hai, aur function names ek dusre se conflict nahi karte hain kyunki woh alag-alag modules mein hain.

Is case mein, kyunki hum ek library bana rahe hain, hamari library banane ke liye entry point ke roop mein kaam karne wali file **src/lib.rs** hai. Halanki, modules banane ke sandarbh mein, **src/lib.rs** ke bare mein kuch bhi special nahi hai. Hum binary crate ke liye bhi **src/main.rs** mein usi tarah modules bana sakte hain jaise hum library crate ke liye **src/lib.rs** mein bana rahe hain. Darasal, hum modules ko modules ke andar rakh sakte hain, jo aapke modules ke badhne par related functionality ko ek saath organize rakhne aur separate functionality ko alag karne ke liye useful ho sakta hai. Aap apne code ko organize karne ka jo tarika chunte hain, woh is baat par nirbhar karta hai ki aap apne code ke parts ke beech ke relationship ke bare mein kaise sochte hain. Udaharan ke liye, `client` code aur uska `connect` function hamari library ke users ke liye zyada samajhdari ka lag sakta hai agar woh `network` namespace ke andar hote, jaisa ki Listing 7-2 mein hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```

\<span class="caption"\>Listing 7-2: `client` module ko `network` module ke andar move karna\</span\>

Apni **src/lib.rs** file mein, maujooda `mod network` aur `mod client` definitions ko Listing 7-2 mein diye gaye se badal dein, jismein `client` module `network` ka ek inner module hai. `network::connect` aur `network::client::connect` dono functions ka naam `connect` hai, lekin woh ek dusre se conflict nahi karte hain kyunki woh alag-alag namespaces mein hain.

Is tarah, modules ek hierarchy banate hain. **src/lib.rs** ke contents sabse upari level par hain, aur submodules lower levels par hain. Yahaan yeh hierarchy hai ki hamare Listing 7-1 ka example hierarchy ke roop mein soche jane par kaisa dikhta hai:

```text
communicator
 ├── network
 └── client
```

Aur yahaan woh hierarchy hai jo Listing 7-2 ke example se milti hai:

```text
communicator
 └── network
     └── client
```

Hierarchy dikhati hai ki Listing 7-2 mein, `client` `network` module ka ek child hai, na ki sibling. Aur bhi complicated projects mein bahut se modules ho sakte hain, aur unhein logically organize karne ki zaroorat padegi taaki aap unka track rakh sakein. Aapke project mein "logically" ka kya matlab hai, yeh aap par nirbhar karta hai aur is baat par bhi ki aap aur aapki library ke users aapke project ke domain ke bare mein kaise sochte hain. Yahan dikhaye gaye techniques ka upyog karke aap side-by-side modules aur nested modules bana sakte hain jaisi bhi structure aap chahte hain.

-----

### Moving Modules to Other Files

Modules ek hierarchical structure banate hain, jaise computing mein ek aur structure jise aap jante hain: filesystems\! Hum Rust ke module system ka upyog multiple files ke saath Rust projects ko divide karne ke liye kar sakte hain taaki sab kuch **src/lib.rs** ya **src/main.rs** mein na rahe. Is example ke liye, chaliye Listing 7-3 mein diye gaye code se shuru karte hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
mod client {
    fn connect() {
    }
}

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

\<span class="caption"\>Listing 7-3: Teen modules, `client`, `network`, aur `network::server`, sabhi `src/lib.rs` mein define kiye gaye hain\</span\>

**src/lib.rs** file mein yeh module hierarchy hai:

```text
communicator
 ├── client
 └── network
     └── server
```

Agar in modules mein bahut se functions hote, aur woh functions lambe ho rahe hote, toh is file mein scroll karke us code ko dhundhna mushkil hota jiske saath hum kaam karna chahte the. Kyunki functions ek ya ek se zyada `mod` blocks ke andar nested hain, functions ke andar ki code ki lines bhi lambi hone lagengi. Yeh `client`, `network`, aur `server` modules ko **src/lib.rs** se alag karne aur unhein apni files mein rakhne ke liye achhe reasons honge.

Sabse pehle, chaliye `client` module code ko sirf `client` module ke declaration se badal dete hain taaki **src/lib.rs** Listing 7-4 mein dikhaye gaye code jaisa dikhe.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
mod client;

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

\<span class="caption"\>Listing 7-4: `client` module ke contents ko extract karna lekin declaration ko `src/lib.rs` mein chhod dena\</span\>

Hum ab bhi yahan `client` module **declare** kar rahe hain, lekin block ko ek semicolon se badalkar, hum Rust ko bata rahe hain ki `client` module ke scope ke andar define kiye gaye code ke liye kisi aur location mein dekhe. Dusre shabdon mein, line `mod client;` ka matlab yeh hai:

```rust,ignore
mod client {
    // contents of client.rs
}
```

Ab humein us module naam ke saath external file create karne ki zaroorat hai. Apni **src/** directory mein ek **client.rs** file banayein aur use open karein. Phir neeche diya gaya code enter karein, jo `client` module mein `connect` function hai jise humne pichle step mein hataya tha:

\<span class="filename"\>Filename: src/client.rs\</span\>

```rust
fn connect() {
}
```

Dhyan dein ki humein is file mein `mod` declaration ki zaroorat nahi hai kyunki humne `client` module ko **src/lib.rs** mein `mod` ke saath pehle hi declare kar diya hai. Yeh file sirf `client` module ke **contents** provide karta hai. Agar hum yahan `mod client` dalte hain, toh hum `client` module ko uska apna `client` naam ka submodule de rahe honge\!

Rust default roop se sirf **src/lib.rs** mein dekhna janta hai. Agar hum apne project mein aur files jodna chahte hain, toh humein **src/lib.rs** mein Rust ko batana padta hai ki dusri files mein dekhe; isliye `mod client` ko **src/lib.rs** mein define karne ki zaroorat hai aur **src/client.rs** mein define nahi kiya ja sakta.

Ab project successfully compile hona chahiye, halanki aapko kuch warnings milengi. Yaad rakhein ki `cargo run` ke bajaye `cargo build` ka upyog karein kyunki hamare paas binary crate ke bajaye ek library crate hai:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/lib.rs:4:5
  |
4 | /     fn connect() {
5 | |     }
  | |_____^

warning: function is never used: `connect`
 --> src/lib.rs:8:9
  |
8 | /         fn connect() {
9 | |         }
  | |_________^
```

Yeh warnings humein batati hain ki hamare paas aise functions hain jo kabhi upyog nahi hote. Abhi ke liye in warnings ki chinta na karein; hum is chapter mein baad mein "Controlling Visibility with `pub`" section mein unhe theek karenge. Achhi khabar yeh hai ki woh sirf warnings hain; hamara project successfully build ho gaya\!

Next, chaliye `network` module ko usi pattern ka upyog karke uski apni file mein extract karte hain. **src/lib.rs** mein, `network` module ki body ko delete karein aur declaration mein ek semicolon add karein, jaise:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
mod client;

mod network;
```

Phir ek naya **src/network.rs** file create karein aur neeche diya gaya code enter karein:

\<span class="filename"\>Filename: src/network.rs\</span\>

```rust
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

Dhyan dein ki is module file ke andar hamare paas abhi bhi ek `mod` declaration hai; yeh isliye hai kyunki hum ab bhi `server` ko `network` ka ek submodule banana chahte hain.

`cargo build` ko dobara chalaein. Safalta\! Humein ek aur module extract karna hai: `server`. Kyunki yeh ek submodule hai—yaani, ek module ke andar ek module—hamara current tactic jo ek module ko uske naam wali file mein extract karta hai, kaam nahi karega. Hum phir bhi koshish karenge taaki aap error dekh sakein. Sabse pehle, **src/network.rs** ko `server` module ke contents ke bajaye `mod server;` rakhne ke liye badal dein:

\<span class="filename"\>Filename: src/network.rs\</span\>

```rust,ignore
fn connect() {
}

mod server;
```

Phir ek **src/server.rs** file banayein aur usmein `server` module ke contents enter karein jo humne extract kiye the:

\<span class="filename"\>Filename: src/server.rs\</span\>

```rust
fn connect() {
}
```

Jab hum `cargo build` ko chalane ki koshish karte hain, toh humein Listing 7-5 mein dikhaye gaye error milega.

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
error: cannot declare a new module at this location
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
  |
note: maybe move this module `src/network.rs` to its own directory via `src/network/mod.rs`
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
note: ... or maybe `use` the module `server` instead of possibly redeclaring it
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
```

\<span class="caption"\>Listing 7-5: `server` submodule ko **src/server.rs** mein extract karne ki koshish karte samay error\</span\>

Error kehta hai ki hum **`cannot declare a new module at this location`** aur **src/network.rs** mein `mod server;` line ko point kar raha hai. Toh **src/network.rs** kisi na kisi tarah se **src/lib.rs** se alag hai: kyun yeh samajhne ke liye aage padhein.

Listing 7-5 ke beech mein diya gaya note asal mein bahut helpful hai kyunki yeh ek aisi cheez ko point out karta hai jiske bare mein humne abhi tak baat nahi ki hai:

```text
note: maybe move this module `network` to its own directory via
`network/mod.rs`
```

Pehle upyog kiye gaye file-naming pattern ko continue karne ke bajaye, hum wahi kar sakte hain jo note suggest karta hai:

1.  **`network`** naam ki ek nayi **directory** banayein, jo parent module ka naam hai.
2.  **src/network.rs** file ko nayi **network** directory mein move karein aur uska naam badalkar **src/network/mod.rs** kar dein.
3.  **src/server.rs** submodule file ko **network** directory mein move karein.

In steps ko poora karne ke liye commands yahaan hain:

```text
$ mkdir src/network
$ mv src/network.rs src/network/mod.rs
$ mv src/server.rs src/network
```

Ab jab hum `cargo build` ko chalane ki koshish karte hain, toh compilation kaam karega (halanki, humein ab bhi warnings milengi). Hamara module layout abhi bhi bilkul waisa hi dikhta hai jaisa jab hamare paas sara code **src/lib.rs** mein tha Listing 7-3 mein:

```text
communicator
 ├── client
 └── network
     └── server
```

Corresponding file layout ab is tarah dikhta hai:

```text
└── src
    ├── client.rs
    ├── lib.rs
    └── network
        ├── mod.rs
        └── server.rs
```

Toh jab hum `network::server` module ko extract karna chahte the, toh humein **src/network.rs** file ko **src/network/mod.rs** file mein badalna aur `network::server` ke liye code ko **network** directory mein **src/network/server.rs** mein kyun rakhna pada? Hum sirf `network::server` module ko **src/server.rs** mein kyun nahi extract kar sakte the? Reason yeh hai ki agar **server.rs** file **src** directory mein hoti, toh Rust yeh pehchan nahi pata ki `server` ko `network` ka submodule hona chahiye tha. Yahan Rust ke behavior ko clear karne ke liye, chaliye ek alag example consider karte hain jismein neeche di gayi module hierarchy hai, jahan sabhi definitions **src/lib.rs** mein hain:

```text
communicator
 ├── client
 └── network
     └── client
```

Is example mein, hamare paas phir se teen modules hain: `client`, `network`, aur `network::client`. Modules ko files mein extract karne ke liye humne jo steps pehle kiye the, unhein follow karte hue, hum `client` module ke liye **src/client.rs** banayenge. `network` module ke liye, hum **src/network.rs** banayenge. Lekin hum `network::client` module ko **src/client.rs** file mein extract nahi kar payenge kyunki woh top-level `client` module ke liye pehle se hi maujood hai\! Agar hum `client` aur `network::client` dono modules ke liye code **src/client.rs** file mein rakh pate, toh Rust ke paas yeh janne ka koi tarika nahi hota ki code `client` ke liye tha ya `network::client` ke liye.

Isliye, `network` module ke `network::client` submodule ke liye ek file extract karne ke liye, humein **src/network.rs** file ke bajaye `network` module ke liye ek directory banane ki zaroorat thi. Jo code `network` module mein hai, woh phir **src/network/mod.rs** file mein jata hai, aur submodule `network::client` ki apni **src/network/client.rs** file ho sakti hai. Ab top-level **src/client.rs** bilkul wahi code hai jo `client` module se belong karta hai.

-----

### Rules of Module Filesystems

Chaliye, files ke sandarbh mein modules ke rules ko summarize karte hain:

  * Agar `foo` naam ke ek module mein koi submodules nahi hain, toh aapko `foo` ke declarations ko **foo.rs** naam ki file mein rakhna chahiye.
  * Agar `foo` naam ke ek module mein submodules hain, toh aapko `foo` ke declarations ko **foo/mod.rs** naam ki file mein rakhna chahiye.

Yeh rules recursively lagu hote hain, isliye agar `foo` naam ke ek module mein `bar` naam ka ek submodule hai aur `bar` mein koi submodules nahi hain, toh aapki **src** directory mein neeche di gayi files honi chahiye:

```text
└── foo
    ├── bar.rs (contains the declarations in `foo::bar`)
    └── mod.rs (contains the declarations in `foo`, including `mod bar`)
```

Modules ko unke parent module ki file mein `mod` keyword ka upyog karke declare kiya jana chahiye.

Next, hum `pub` keyword ke bare mein baat karenge aur un warnings se chhutkara payenge\!
