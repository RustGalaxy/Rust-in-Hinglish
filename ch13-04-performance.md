# Performance Compare Karein: Loops vs. Iterators

Yeh decide karne ke liye ki loops use karein ya iterators, aapko yeh janna hoga ki hamare `search` functions ka kaun sa version fast hai: woh jismein `for` loop hai ya woh jismein iterators hain.

Humne ek benchmark run kiya jismein Sir Arthur Conan Doyle ki *The Adventures of Sherlock Holmes* ke poore content ko ek `String` mein load kiya aur usmein 'the' word ko search kiya. Yahan `for` loop wale version aur iterators wale version ke benchmark results hain:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

Iterator wala version thoda sa fast tha\! Hum yahan benchmark code ko explain nahi karenge, kyunki point yeh prove karna nahi hai ki dono versions equivalent hain, balki yeh samajhna hai ki performance ke maamle mein yeh dono implementations kaise compare karte hain.

Ek zyada comprehensive benchmark ke liye, aapko alag-alag size ke texts ko `contents` ke roop mein, alag-alag words aur alag-alag length ke words ko `query` ke roop mein, aur har tarah ke doosre variations ke saath check karna chahiye. Point yeh hai: **iterators, bhale hi ek high-level abstraction hain, compile hokar lagbhag waisa hi code banate hain jaisa aap khud lower-level code likhte.** Iterators Rust ke ***zero-cost abstractions*** mein se ek hain, jiska matlab hai ki abstraction use karne se koi extra runtime overhead nahi hota. Yeh bilkul waisa hi hai jaise Bjarne Stroustrup, C++ ke original designer, ne *zero-overhead* ko "Foundations of C++" (2012) mein define kiya hai:

> Aam taur par, C++ implementations zero-overhead principle ko follow karte hain: Jo aap use nahi karte, uske liye aap pay nahi karte. Aur aage: Jo aap use karte hain, use aap haath se isse behtar code nahi kar sakte.

Ek aur example ke liye, neeche diya gaya code ek audio decoder se liya gaya hai. Decoding algorithm linear prediction mathematical operation ka use karta hai taaki previous samples ke linear function ke basis par future values ka anuman lagaya ja sake. Yeh code ek iterator chain ka use karke scope mein maujood teen variables par kuch math karta hai: ek `buffer` slice of data, 12 `coefficients` ka ek array, aur `qlp_shift` amount jisse data ko shift karna hai. Humne is example mein variables declare kiye hain lekin unhe koi value nahi di hai; bhale hi is code ka apne context ke bahar zyada matlab na ho, lekin yeh ek chhota sa, real-world example hai ki kaise Rust high-level ideas ko low-level code mein translate karta hai.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                .zip(&buffer[i - 12..i])
                                .map(|(&c, &s)| c * s as i64)
                                .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

`prediction` ki value calculate karne ke liye, yeh code `coefficients` mein maujood 12 values mein se har ek par iterate karta hai aur `zip` method ka use karke coefficient values ko `buffer` ke pichle 12 values ke saath pair karta hai. Phir, har pair ke liye, hum values ko multiply karte hain, saare results ko sum karte hain, aur sum mein `qlp_shift` bits ko right mein shift karte hain.

Audio decoders jaise applications mein calculations aksar performance ko sabse zyada priority dete hain. Yahan, hum ek iterator bana rahe hain, do adaptors use kar rahe hain, aur phir value ko consume kar rahe hain. Yeh Rust code kis assembly code mein compile hoga? Well, jab yeh likha ja raha hai, tab tak yeh usi assembly mein compile hota hai jise aap haath se likhte. `coefficients` ki values par iteration ke liye koi loop hai hi nahi: Rust jaanta hai ki 12 iterations hain, isliye woh loop ko "unroll" kar deta hai. ***Unrolling*** **ek optimization hai jo loop control karne wale code ke overhead ko hata deta hai aur uski jagah loop ke har iteration ke liye repetitive code generate karta hai.**

Saare coefficients registers mein store ho jaate hain, jiska matlab hai ki values ko access karna bahut fast hai. Runtime par array access par koi bounds checks nahi hote. Yeh saare optimizations jo Rust apply kar paata hai, resulting code ko extremely efficient bana dete hain. Ab jab aap yeh jaante hain, aap iterators aur closures ko bina dare use kar sakte hain\! Woh code ko high-level dikhate hain lekin aisa karne ke liye koi runtime performance penalty nahi lagate.

-----

## Summary

Closures aur iterators Rust ke features hain jo functional programming language ke ideas se inspired hain. Yeh Rust ki capability mein contribute karte hain taaki high-level ideas ko low-level performance par saaf-saaf express kiya ja sake. Closures aur iterators ke implementations aise hain ki runtime performance par koi asar nahi padta. Yeh Rust ke zero-cost abstractions provide karne ke goal ka ek hissa hai.

Ab jab humne apne I/O project ki expressiveness ko improve kar liya hai, chaliye `cargo` ke kuch aur features dekhte hain jo humein project ko duniya ke saath share karne mein madad karenge.
