# Tests Kaise Likhein ✍️

Tests Rust functions hote hain jo non-test code ko **verify** karte hain ki wo expected manner mein kaam kar raha hai. Test functions ki bodies aam taur par ye teen actions perform karti hain:

1.  Koi bhi zaroori data ya state **set up** karna.
2.  Jis code ko aap test karna chahte hain, use **run** karna.
3.  **Assert** karna ki results wahi hain jo aap expect karte hain.

Aaiye un features ko dekhte hain jo Rust tests likhne ke liye provide karta hai, jismein **`test` attribute**, kuch macros, aur **`should_panic` attribute** shamil hain.

-----

### Ek Test Function ka Anatomy 🔬

Sabse simple roop mein, Rust mein ek test ek function hai jo **`test` attribute** ke saath annotate kiya gaya hai. Attributes Rust code ke pieces ke baare mein metadata hote hain. Ek function ko test function mein badalne ke liye, `fn` se pehle wali line par `#[test]` add karein. Jab aap **`cargo test`** command se apne tests run karte hain, to Rust ek test runner binary banata hai jo `test` attribute ke saath annotate kiye gaye functions ko run karta hai aur har test function ke **pass ya fail** hone ki report deta hai.

Jab hum Cargo se ek naya library project banate hain, to ek test module uske andar ek test function ke saath automatically generate ho jaata hai, jaisa **Listing 11-1** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

\<span class="caption"\>Listing 11-1: `cargo new` dwara automatically generate kiya gaya test module aur function\</span\>

`#[test]` annotation se test runner ko pata chalta hai ki ye ek test function hai. Function ki body **`assert_eq!` macro** ka upyog karti hai taaki ye assert kar sake ki `2 + 2` `4` ke barabar hai. Ye ek typical test ke format ka example hai. Chaliye isko run karke dekhte hain.

**`cargo test`** command hamare project ke saare tests ko run karta hai, jaisa **Listing 11-2** mein dikhaya gaya hai.

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.22 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

\<span class="caption"\>Listing 11-2: Automatically generate kiye gaye test ko run karne ka output\</span\>

`running 1 test` ke baad, `it_works` naam ke test function ka naam aur uska result **`ok`** dikhta hai. `test result: ok.` ka matlab hai ki saare tests pass ho gaye.

Aaiye ek aur test add karte hain, lekin is baar hum ek aisa test banayenge jo **fail** ho\! Tests tab fail hote hain jab test function ke andar kuch **`panic!`** hota hai. Har test ek naye **thread** mein run hota hai, aur jab main thread dekhta hai ki ek test thread died ho gaya hai, to test ko failed mark kar diya jaata hai. **Listing 11-3** mein naya test `another` add karte hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```

\<span class="caption"\>Listing 11-3: Ek doosra test add karna jo `panic!` macro call karne ki wajah se fail hoga\</span\>

Tests ko dobara run karein. **Listing 11-4** mein output dikhaya gaya hai, jismein hamara `exploration` test pass ho gaya hai aur `another` fail ho gaya hai.

```text
running 2 tests
test tests::exploration ... ok
test tests::another ... FAILED

failures:

---- tests::another stdout ----
    thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:10:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed
```

\<span class="caption"\>Listing 11-4: Test results jab ek test pass hota hai aur ek fail hota hai\</span\>

`FAILED` line ke baad ek naya section aata hai jo har test failure ka detailed reason dikhata hai. Yahan, `another` fail ho gaya kyunki isne `panic!` kiya. `test result: FAILED.` ka matlab hai ki overall test result fail hai.

-----

### `assert!` Macro se Results Check karna ✅

Standard library dwara provide kiya gaya **`assert!` macro** tab useful hota hai jab aap ensure karna chahte hain ki ek test mein koi condition `true` hai. Hum `assert!` macro ko ek argument dete hain jo ek Boolean value return karta hai. Agar value `true` hai, to `assert!` kuch nahi karta aur test pass ho jaata hai. Agar value `false` hai, to `assert!` macro `panic!` call karta hai, jisse test fail ho jaata hai.

**Chapter 5, Listing 5-15** mein humne `Rectangle` struct aur `can_hold` method ka upyog kiya tha, jaisa **Listing 11-5** mein repeat kiya gaya hai. Chaliye is code ko *src/lib.rs* file mein rakhte hain aur `assert!` macro ka upyog karke iske liye kuch tests likhte hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# fn main() {}
#[derive(Debug)]
pub struct Rectangle {
    length: u32,
    width: u32,
}

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length > other.length && self.width > other.width
    }
}
```

\<span class="caption"\>Listing 11-5: Chapter 5 se `Rectangle` struct aur uske `can_hold` method ka upyog\</span\>

`can_hold` method ek Boolean return karta hai, isliye ye `assert!` macro ke liye ek perfect use case hai. **Listing 11-6** mein, hum `assert!` macro ka upyog karte hain taaki ye check kar saken ki ek bada rectangle sach mein ek chote rectangle ko hold kar sakta hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(larger.can_hold(&smaller));
    }
}
```

\<span class="caption"\>Listing 11-6: `can_hold` ke liye ek test jo check karta hai ki kya ek bada rectangle sach mein ek chote rectangle ko hold kar sakta hai\</span\>

`use super::*;` se hum outer module mein define kiye gaye code ko `tests` module ke scope mein laate hain. Test run karne par, ye pass ho jaata hai. Ab, chaliye `can_hold` method mein ek bug daalte hain. Jab hum length ko compare karte hain to `>` ko `<` se badalte hain. Test dobara run karne par, wo bug pakad leta hai.

-----

### `assert_eq!` aur `assert_ne!` Macros se Equality Test karna ⚖️

Functionality test karne ka ek common tareeka hai ki test kiye ja rahe code ke result ko us value se compare karna jo aap expect karte hain, taaki ye sure ho sake ki wo barabar hain. `assert!` macro ka upyog karke `==` operator se aap aisa kar sakte hain, lekin standard library ek pair of macros provide karta hai, **`assert_eq!`** aur **`assert_ne!`**, jo ye test zyaada suvidha se karte hain.

Ye macros do arguments ko equality ya inequality ke liye compare karte hain. Agar assertion fail hoti hai, to ye dono values ko print bhi karte hain, jisse ye dekhna aasan ho jaata hai ki test fail kyon hua. **Listing 11-7** mein, hum `add_two` naam ka ek function likhte hain aur use `assert_eq!` macro se test karte hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# fn main() {}
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

\<span class="caption"\>Listing 11-7: `add_two` function ko `assert_eq!` macro se test karna\</span\>

Test pass ho jaata hai. Ab, `add_two` function mein ek bug daalte hain, jismein wo 2 ki jagah 3 add karta hai. Tests dobara run karne par, wo fail ho jaate hain.

```text
running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
        thread 'tests::it_adds_two' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:11:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

Test bug pakad leta hai\! `left: 4` aur `right: 5` dikhakar, ye message debugging shuru karne mein madad karta hai. `assert_eq!` aur `assert_ne!` macros `PartialEq` aur `Debug` traits ko implement karne wale values ko compare karte hain.

-----

### Custom Failure Messages Add karna 💬

Aap **`assert!`**, **`assert_eq!`**, aur **`assert_ne!`** macros mein optional arguments ke roop mein ek **custom message** bhi add kar sakte hain. Jab test fail hota hai, to ye message print hota hai, jisse problem ko samajhna aasan ho jaata hai.

```rust,ignore
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```

Jab hum is test ko run karte hain, to hamein ek informative error message milega:

```text
---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'Greeting did not
contain name, value was `Hello!`', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

`value was Hello!` dikhane se humein pata chalta hai ki `greeting` function ne kya return kiya tha.

-----

### `should_panic` se Panics Check karna 🚨

Sahi values check karne ke alawa, ye bhi zaroori hai ki hamara code error conditions ko expected tareeke se handle karta hai ya nahi. `Guess` type ke liye, hum ek test likh sakte hain jo ensure karta hai ki us range ke bahar ki value ke saath `Guess` instance banane ki koshish karne par **`panic!`** hota hai.

Hum aisa ek aur attribute, **`should_panic`**, apne test function mein add karke karte hain. Ye attribute ek test ko tab pass karta hai jab function ke andar ka code panic ho; agar code panic nahi hota, to test fail ho jaata hai. **Listing 11-8** mein ek test dikhaya gaya hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# fn main() {}
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

\<span class="caption"\>Listing 11-8: Test karna ki ek condition `panic!` cause karegi\</span\>

Is test ko run karne par, `ok` dikhta hai, jiska matlab hai ki test pass ho gaya.

`should_panic` tests ko aur zyaada precise banane ke liye, hum **`expected` parameter** add kar sakte hain. Test harness ensure karega ki failure message mein diya gaya text shamil ho. **Listing 11-9** mein, `new` function alag-alag messages ke saath panic karta hai, aur hum `expected` parameter ka upyog karte hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# fn main() {}
# pub struct Guess {
#     value: u32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 {
            panic!("Guess value must be greater than or equal to 1, got {}.",
                   value);
        } else if value > 100 {
            panic!("Guess value must be less than or equal to 100, got {}.",
                   value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

\<span class="caption"\>Listing 11-9: Test karna ki ek condition ek particular panic message ke saath `panic!` cause karegi\</span\>

Ye test pass ho jaayega. Agar `new` function ka bug wapas aata hai aur wrong message ke saath panic hota hai, to test fail ho jaata hai aur fail message batata hai ki panic message mein expected string shamil nahi thi.

Ab jab aap jaante hain ki tests kaise likhe jaate hain, to hum dekhenge ki `cargo test` ke saath alag-alag options ka upyog kaise karte hain.
