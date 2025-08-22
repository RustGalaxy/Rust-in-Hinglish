# `panic!` se Unrecoverable Errors 💥

Kabhi-kabhi, aapke code mein kuch aisi buri cheezein ho jaati hain, jiske baare mein aap kuch nahi kar sakte. Aise cases mein, Rust ke paas **`panic!` macro** hai. Jab `panic!` macro execute hota hai, to aapka program ek failure message print karta hai, stack ko unwind aur clean up karta hai, aur fir quit ho jaata hai. Aisa aam taur par tab hota hai jab kisi tarah ka bug detect ho gaya ho aur programmer ko samajh na aaye ki is error ko kaise handle kiya jaaye.

### Panic ke Response mein Stack ko Unwind ya Abort karna ↩️

By default, jab ek panic hota hai, to program **unwinding** shuru kar deta hai, jiska matlab hai ki Rust stack par wapas upar jaata hai aur har function se data ko clean up karta hai. Lekin ye wapas jaana aur cleanup karna kaafi kaam hai. Dusra option hai **turant abort karna**, jo bina cleanup ke program ko end kar deta hai. Aisi situation mein, program jo memory use kar raha tha use operating system dwara clean kiya jaayega. Agar aapko apne project mein resultant binary ko jitna ho sake utna chhota banana hai, to aap `Cargo.toml` file mein `panic = 'abort'` add karke unwinding se abort karne par switch kar sakte hain. Jaise, agar aap release mode mein panic hone par abort karna chahte hain, to ye add karein:

```toml
[profile.release]
panic = 'abort'
```

Chaliye ek simple program mein `panic!` ko call karke dekhte hain:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,should_panic
fn main() {
    panic!("crash and burn");
}
```

Jab aap program run karenge, to aapko kuch is tarah ka output dikhega:

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25 secs
     Running `target/debug/panic`
thread 'main' panicked at 'crash and burn', src/main.rs:2:4
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

`panic!` ki wajah se ye error message aata hai. Pehli line hamara panic message aur hamare source code mein panic ki jagah dikhati hai: *src/main.rs:2:4* ka matlab hai ki ye *src/main.rs* file ki doosri line, chautha character hai.

Is case mein, line hamare code ka part hai. Lekin doosre cases mein, `panic!` call us code mein ho sakta hai jise hamara code call karta hai. Aise mein, error message mein report kiya gaya filename aur line number kisi aur ke code ka hoga, na ki hamare code ki us line ka jisne eventually `panic!` call ko lead kiya. Hum **backtrace** ka upyog kar sakte hain ye pata lagane ke liye ki hamare code ka kaunsa hissa problem create kar raha hai.

-----

### `panic!` Backtrace ka Upyog 🔍

Aaiye ek aur example dekhte hain ki jab `panic!` call seedhe hamare code se nahi, balki hamare code mein ek bug ki wajah se kisi library se aata hai to kaisa hota hai. **Listing 9-1** mein, kuch code ek vector mein index se ek element ko access karne ki koshish karta hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,should_panic
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

\<span class="caption"\>Listing 9-1: Vector ke end ke baad ke element ko access karne ki koshish, jisse `panic!` call hoga\</span\>

Yahan, hum vector ke 100th element ko access karne ki koshish kar rahe hain (jo index 99 par hai kyunki indexing zero se shuru hoti hai), lekin ismein sirf 3 elements hain. Is situation mein, Rust panic ho jaayega. `[]` ka upyog karne ka matlab hai ki ek element return ho, lekin agar aap ek invalid index dete hain, to Rust yahan koi sahi element return nahi kar sakta.

C jaisi doosri languages is situation mein aapko wahi dene ki koshish karengi jo aapne manga hai, bhale hi wo aap nahi chahte: aapko us memory location par jo kuch bhi hai wo milega jo vector ke us element se correspond karta hai, bhale hi wo memory vector ki nahi hai. Isse **buffer overread** kehte hain aur ye security vulnerabilities ka karan ban sakta hai.

Apne program ko is tarah ki vulnerability se bachane ke liye, agar aap ek aise index par element padhne ki koshish karte hain jo exist nahi karta, to Rust execution rok dega aur aage badhne se mana kar dega. Aaiye isko try karte hain aur dekhte hain:

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is
99', /checkout/src/liballoc/vec.rs:1555:10
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Ye error ek aisi file ko point karta hai jo humne nahi likhi hai, *vec.rs*. Ye standard library mein `Vec<T>` ka implementation hai. Jab hum apne vector `v` par `[]` ka use karte hain, to jo code run hota hai wo *vec.rs* mein hai, aur yahi par asal mein `panic!` ho raha hai.

Agli note line humein batati hai ki hum **`RUST_BACKTRACE` environment variable** ko set karke exact backtrace pa sakte hain ki error kis wajah se aayi. Ek **backtrace** उन saare functions ki list hai jinhein is point tak call kiya gaya hai. Rust mein backtraces doosri languages ki tarah hi kaam karte hain: backtrace ko padhne ka tareeka ye hai ki aap upar se shuru karein aur tab tak padhein jab tak aapko apne likhe hue files na milen. Yahi wo jagah hai jahan problem shuru hui thi. Chaliye `RUST_BACKTRACE` environment variable ko kisi bhi value ke saath set karke ek backtrace paane ki koshish karte hain, jaisa **Listing 9-2** mein dikhaya gaya hai.

```text
$ RUST_BACKTRACE=1 cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', /checkout/src/liballoc/vec.rs:1555:10
stack backtrace:
// --snip--
  11: panic::main
             at src/main.rs:4
// --snip--
```

\<span class="caption"\>Listing 9-2: `RUST_BACKTRACE` environment variable set hone par `panic!` se generate hua backtrace\</span\>

**Listing 9-2** ke output mein, backtrace ki line 11 hamare project ki us line ko point karti hai jo problem cause kar rahi hai: *src/main.rs* ki line 4. Agar hum nahi chahte ki hamara program panic ho, to hamare likhe hue file ko mention karne wali pehli line hi wo jagah hai jahan humein investigation shuru karni chahiye. **Listing 9-1** mein, jahan humne jaanbujhkar aisa code likha tha jo panic ho, is panic ko fix karne ka tareeka ye hai ki hum ek aise vector se index 99 par element na maangen jismein sirf 3 items hain. Jab bhavishya mein aapka code panic hoga, to aapko pata lagana hoga ki code kis action aur values ke saath panic ho raha hai aur uske bajaye use kya karna chahiye.

Hum is chapter mein baad mein `panic!` par wapas aayenge aur ye discuss karenge ki error conditions ko handle karne ke liye humein `panic!` ka upyog kab karna chahiye aur kab nahi. Ab hum dekhte hain ki `Result` ka upyog karke error se kaise recover kiya jaata hai.
