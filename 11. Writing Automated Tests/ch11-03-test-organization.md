# Tests ko Organize Karna 📊

Testing ek complex discipline hai, aur alag-alag log alag-alag terminology aur organization ka use karte hain. Rust community tests ke baare mein do main categories mein sochti hai: **unit tests** aur **integration tests**. Unit tests chote aur zyaada focused hote hain, jo ek samay mein ek module ko isolation mein test karte hain, aur **private interfaces** ko bhi test kar sakte hain. Integration tests aapki library se poori tarah se **external** hote hain aur aapke code ka upyog bilkul waise hi karte hain jaise koi doosra external code karega, sirf **public interface** ka upyog karke.

Dono tarah ke tests likhna zaroori hai taaki ye ensure ho sake ki aapki library ke pieces alag-alag aur ek saath wahi kaam kar rahe hain jo aap unse expect karte hain.

-----

### Unit Tests 🎯

Unit tests ka maqsad code ke har unit ko baki code se isolation mein test karna hai taaki jald se jald pata chal sake ki code kahan kaam kar raha hai aur kahan nahi. Aap unit tests ko us file mein **`src` directory** ke andar rakhenge jismein wo code hai jise wo test kar rahe hain. Convention ye hai ki har file mein **`tests`** naam ka ek module banaya jaaye jismein test functions hon aur us module ko **`cfg(test)`** se annotate kiya jaaye.

#### `tests` Module aur `#[cfg(test)]` 🛠️

`#[cfg(test)]` annotation Rust ko batati hai ki test code ko tabhi compile aur run karna hai jab aap **`cargo test`** run karein, na ki jab aap **`cargo build`** run karein. Isse compile time bachta hai aur resultant compiled artifact mein space bachta hai, kyunki tests shamil nahi hote.

Jab humne naya `adder` project generate kiya tha, to Cargo ne hamare liye ye code generate kiya tha:

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

`cfg` ka matlab **configuration** hai aur ye Rust ko batata hai ki agla item tabhi shamil hona chahiye jab ek certain configuration option diya gaya ho. Is case mein, configuration option `test` hai, jo Rust dwara tests ko compile aur run karne ke liye provide kiya gaya hai.

#### Private Functions ko Test karna 🔒

Testing community mein is baat par debate hai ki private functions ko directly test karna chahiye ya nahi, aur kuch languages mein private functions ko test karna mushkil ya impossible hota hai. Rust ke **privacy rules** aapko private functions ko test karne dete hain. **Listing 11-12** mein private function `internal_adder` ke saath code ko dekhein.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

\<span class="caption"\>Listing 11-12: Ek private function ko test karna\</span\>

Dhyan dein ki `internal_adder` function ko `pub` mark nahi kiya gaya hai, lekin tests Rust code hi hain aur `tests` module sirf ek aur module hai, to aap `internal_adder` ko ek test mein import aur call kar sakte hain.

-----

### Integration Tests 🧩

Rust mein, integration tests aapki library se poori tarah **external** hote hain. Wo aapki library ka upyog bilkul waise hi karte hain jaise koi doosra code karega, jiska matlab hai ki wo sirf un functions ko call kar sakte hain jo aapki library ke public API ka part hain. Unka maqsad ye test karna hai ki kya aapki library ke kayi parts ek saath sahi se kaam kar rahe hain ya nahi.

#### `tests` Directory 📁

Hum apne project directory ke top level par, `src` ke bagal mein ek **`tests` directory** banate hain. Cargo ko pata hai ki integration test files ko is directory mein dhoondhna hai. Hum is directory mein jitni chahein utni test files bana sakte hain.

Ek integration test banate hain. **Listing 11-12** ke code ke saath, ek `tests` directory banayein, ek nayi file *tests/integration\_test.rs* banayein aur **Listing 11-13** ka code enter karein.

\<span class="filename"\>Filename: tests/integration\_test.rs\</span\>

```rust,ignore
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

\<span class="caption"\>Listing 11-13: `adder` crate mein ek function ka integration test\</span\>

Humne code ke top par `extern crate adder` add kiya hai, jiski unit tests mein zaroorat nahi thi. Iska reason ye hai ki `tests` directory mein har test ek **separate crate** hai, to humein apni library ko har ek mein import karna padta hai.

`tests/integration_test.rs` mein kisi bhi code ko `#[cfg(test)]` se annotate karne ki zaroorat nahi hai. Cargo `tests` directory ko specially treat karta hai aur is directory mein files ko tabhi compile karta hai jab hum **`cargo test`** run karte hain. Ab `cargo test` run karein:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running target/debug/deps/adder-abcabcabc

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-ce99bcc2479f4607

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Output ke teen sections mein unit tests, integration test, aur doc tests shamil hain. Har integration test file ka apna section hota hai, to agar hum `tests` directory mein aur files add karte hain, to aur zyaada integration test sections honge.

-----

### Summary 📝

Rust ke testing features code ko is tarah specify karne ka tareeka provide karte hain ki wo expected tareeke se kaam karta rahe, bhale hi aap usmein changes karen. **Unit tests** ek library ke alag-alag parts ko separately exercise karte hain aur **private implementation details** ko test kar sakte hain. **Integration tests** check karte hain ki library ke kayi parts ek saath sahi se kaam kar rahe hain ya nahi, aur wo library ke **public API** ka use karke code ko test karte hain. Bhale hi Rust ka type system aur ownership rules kuch tarah ke bugs ko rokne mein madad karte hain, tests abhi bhi important hain.

Ab hum is chapter aur pichle chapters mein seekhe hue knowledge ko ek project par kaam karne ke liye jodenge\!
