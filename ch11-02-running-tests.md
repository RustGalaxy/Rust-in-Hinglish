# Tests ko Kaise Run Karein 🏃‍♂️

Bilkul `cargo run` ki tarah jo aapke code ko compile karta hai aur fir resultant binary ko run karta hai, `cargo test` aapke code ko **test mode** mein compile karta hai aur fir resultant **test binary** ko run karta hai. Aap **command line options** specify karke `cargo test` ka default behavior change kar sakte hain. Jaise, `cargo test` se bane binary ka default behavior saare tests ko **parallel** mein run karna aur test runs ke dauran generate hue output ko **capture** karna hota hai, jisse output display nahi hota aur test results se related output padhna aasan ho jaata hai.

Kuch command line options `cargo test` ko jaate hain, aur kuch resultant test binary ko jaate hain. In do tarah ke arguments ko alag karne ke liye, aap `cargo test` ko jaane wale arguments ko list karte hain aur fir **separator `--`** aur fir test binary ko jaane wale arguments ko list karte hain.

-----

### Tests ko Parallel ya Consecutively Run karna 🤝

Jab aap multiple tests run karte hain, to by default wo **threads** ka upyog karke parallel mein run hote hain. Iska matlab hai ki tests jaldi run ho jaayenge taaki aapko jaldi feedback mil sake. Kyunki tests ek hi time par run hote hain, isliye ensure karein ki aapke tests ek doosre par ya kisi **shared state** par nirbhar na ho.

For example, agar aapke har test mein ek file banti hai jiska naam *test-output.txt* hai, to jab tests parallel mein run hote hain, to ek test doosre test ki file ko overwrite kar sakta hai, jisse test fail ho jaayega. Is problem ka ek solution ye hai ki aap har test ko alag file mein likhne ke liye kahein; doosra solution ye hai ki tests ko **ek-ek karke** run kiya jaaye.

Agar aap tests ko parallel mein run nahi karna chahte ya use hone wale threads ki sankhya par zyaada fine-grained control chahte hain, to aap **`--test-threads` flag** aur jitne threads aap use karna chahte hain wo number test binary ko bhej sakte hain.

```text
$ cargo test -- --test-threads=1
```

Humne test threads ki sankhya `1` set kar di hai, jo program ko **parallelism** ka upyog na karne ke liye kehta hai. Isse tests run hone mein zyaada time lagega, lekin agar wo shared state use karte hain to wo ek doosre ke saath interfere nahi karenge.

-----

### Function ka Output Dikhana 🖨️

By default, agar ek test pass hota hai, to Rust ki test library standard output par print ki gayi har cheez ko capture kar leti hai. Jaise, agar hum ek test mein `println!` call karte hain aur test pass ho jaata hai, to hum terminal mein `println!` ka output nahi dekh paate; humein sirf wo line dikhti hai jo batati hai ki test pass ho gaya hai. Agar ek test fail hota hai, to humein standard output par print ki gayi har cheez test failure message ke saath dikhti hai.

**Listing 11-10** mein ek function hai jo apne parameter ki value print karta hai aur 10 return karta hai, saath hi ek test jo pass hota hai aur ek test jo fail hota hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

\<span class="caption"\>Listing 11-10: Ek function ke liye tests jo `println!` call karta hai\</span\>

Jab hum in tests ko `cargo test` ke saath run karte hain, to hamein output mein `I got the value 4` nahi dikhta, jo passing test ne print kiya tha. Lekin failing test ka output `I got the value 8` failure message ke saath dikhta hai.

Agar hum passing tests ke liye bhi printed values dekhna chahte hain, to hum `--nocapture` flag ka upyog karke output capture behavior ko disable kar sakte hain:

```text
$ cargo test -- --nocapture
```

-----

### Naam se Tests ka Subset Run karna 🎯

Kabhi-kabhi, poora **test suite** run karne mein bahut samay lag sakta hai. Agar aap kisi particular area ke code par kaam kar rahe hain, to aap sirf us code se related tests run karna chahenge. Aap `cargo test` ko argument ke roop mein us test ka naam ya naamon ko pass karke chun sakte hain ki kaunse tests run karne hain.

Ye dikhane ke liye ki tests ke subset ko kaise run karte hain, hum hamare `add_two` function ke liye teen tests banayenge, jaisa **Listing 11-11** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```

\<span class="caption"\>Listing 11-11: Teen alag-alag naamon wale tests\</span\>

#### Single Tests ko Run karna 🚀

Hum `cargo test` ko kisi bhi test function ka naam pass kar sakte hain taaki sirf wahi test run ho:

```text
$ cargo test one_hundred
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out
```

Sirf `one_hundred` naam wala test run hua; doosre do tests ne naam match nahi kiya.

#### Filtering se Multiple Tests Run karna 🧹

Hum test ke naam ka ek part specify kar sakte hain, aur koi bhi test jiska naam us value se match karta hai wo run hoga. For example, kyunki hamare do tests ke naam mein `add` shamil hai, hum `cargo test add` run karke un do tests ko run kar sakte hain:

```text
$ cargo test add
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 2 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

-----

### Kuch Tests ko Ignore karna Jab tak ki Specifically Request na ki jaaye 🙈

Kabhi-kabhi kuch khaas tests ko run karne mein bahut samay lag sakta hai, isliye aap unhein `cargo test` ki zyadatar runs ke dauran exclude karna chahenge. Aap **`ignore` attribute** ka upyog karke un time-consuming tests ko annotate kar sakte hain:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

`#[test]` ke baad hum us test mein `#[ignore]` line add karte hain jise hum exclude karna chahte hain. Ab jab hum apne tests run karte hain, to `it_works` run hota hai, lekin `expensive_test` nahi:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out
```

`expensive_test` function ko `ignored` ke roop mein list kiya gaya hai. Agar hum sirf ignored tests ko run karna chahte hain, to hum **`cargo test -- --ignored`** ka upyog kar sakte hain:

```text
$ cargo test -- --ignored
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

Is tarah, aap apne `cargo test` results ko fast rakh sakte hain. Jab aap `ignored` tests ke results ko check karne ka sense banta hai, to aap `cargo test -- --ignored` run kar sakte hain.
