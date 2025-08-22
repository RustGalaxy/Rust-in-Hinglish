## Referring to Names in Different Modules

### `use` Keyword se Names ko Scope Mein Laana

Rust mein, modules ke andar defined functions ko refer karna unke poore path ke saath thoda lamba ho sakta hai, jaise **Listing 7-7** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

fn main() {
    a::series::of::nested_modules();
}
```

Is lengthy process ko chhota karne ke liye, Rust **`use` keyword** ka use karta hai.

-----

### `use` Keyword se Names ko Scope Mein Laana

**`use`** keyword function ke modules ko scope mein laakar call ko chhota karta hai. Jaise, aap **`a::series::of`** module ko binary crate ke root scope mein la sakte hain:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of;

fn main() {
    of::nested_modules();
}
```

Yahan, **`use a::series::of;`** ka matlab hai ki aap jahan bhi **`of`** module ko refer karna chahte hain, wahan poora **`a::series::of`** path likhne ki bajaye sirf **`of`** likh sakte hain.

-----

### Directly Function ko Refer Karna

Aap `use` statement mein directly function ka naam bhi specify kar sakte hain. Isse aap saare modules ko chhodkar sidhe function ko refer kar sakte hain:

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of::nested_modules;

fn main() {
    nested_modules();
}
```

-----

### Multiple Items aur Enum Variants ko Refer Karna

Agar aap ek hi namespace se multiple items ko scope mein laana chahte hain, to aap **curly brackets (`{}`) aur commas** ka use kar sakte hain. Ye syntax **enums** ke variants ke liye bhi kaam karta hai, kyunki enums bhi modules ki tarah ek namespace banate hain.

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::{Red, Yellow};

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = TrafficLight::Green;
}
```

Is example mein, **`Green`** variant ko abhi bhi **`TrafficLight::`** namespace ki zaroorat hai, kyunki humne use **`use`** statement mein include nahi kiya tha.

-----

### Glob Operator (`*`) ka Upyog

Ek hi baar mein namespace ke saare items ko scope mein laane ke liye, aap **glob operator (`*`)** ka use kar sakte hain.

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::*;

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = Green;
}
```

**`*`** operator **`TrafficLight`** namespace ke sabhi visible items ko scope mein le aayega. Iska upyog savdhani se karein, kyunki ye naam conflict ka karan ban sakta hai.

-----

### Parent Module ko Access karne ke liye `super` ka Upyog

Tests mein, `communicator` project ke `tests` module ke andar **`client::connect`** function ko call karne ke liye, compilation error aati hai, kyunki paths hamesha **current module ke relative** hote hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        client::connect();
    }
}
```

Ye error isliye aati hai, kyunki **`tests`** module `client` module ko seedhe-seedhe nahi pahchan sakta.

Is problem ko solve karne ke liye, aap ya to poore **absolute path** (`::client::connect();`) ka use kar sakte hain ya fir **`super` keyword** ka upyog kar sakte hain.

**`super`** keyword aapko current module se ek level up **parent module** mein jaane ki anumati deta hai.

```rust,ignore
super::client::connect();
```

Jab aap module hierarchy mein kafi andar hote hain, to **`super`** shortcut bahut helpful hota hai.

Tests mein **`use super::client;`** ka upyog karna aam taur par best solution hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
#[cfg(test)]
mod tests {
    use super::client;

    #[test]
    fn it_works() {
        client::connect();
    }
}
```

Isse **`tests`** module **`client`** module ko scope mein la sakta hai, aur **`cargo test`** successfully run hoga.
