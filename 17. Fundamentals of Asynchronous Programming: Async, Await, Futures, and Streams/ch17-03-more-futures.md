## Kisi bhi Sankhya ke Futures ke Saath Kaam Karna

Jab hum pichle section mein do futures se teen par switch hue, to humein `join` se `join3` ka istemal karna pada. Har baar jab hum futures ki sankhya badalte hain, to ek alag function call karna pareshani bhara ho sakta hai. Khushkismati se, hamare paas `join` ka ek macro roop hai jismein hum kitne bhi arguments pass kar sakte hain. Yah futures ko `await` karne ka kaam bhi khud hi kar leta hai.

```rust
// Filename: src/main.rs
// ...
// Using the join! macro
trpl::join!(tx_fut, rx_fut, tx1_fut);
```

*Listing 17-14: Kai futures ka intezaar karne ke liye `join!` ka istemal karna*

Yah nishchit roop se ek sudhaar hai\! Halaanki, yah macro roop bhi tabhi kaam karta hai jab humein pehle se futures ki sankhya pata ho. Asli Rust code mein, futures ko ek collection mein daalna aur phir un sabhi ke poora hone ka intezaar karna ek aam pattern hai.

Kisi collection mein sabhi futures ko check karne ke liye, humein un sab par iterate karke join karna hoga. `trpl::join_all` function kisi bhi type ko accept karta hai jo `Iterator` trait ko implement karta hai. Chaliye apne futures ko ek vector mein daalkar `join!` ko `join_all` se replace karne ki koshish karte hain.

```rust,ignore,does_not_compile
// Filename: src/main.rs (Attempt with Vec)
// ...
let futures = vec![tx1_fut, rx_fut, tx_fut];
trpl::join_all(futures).await;
```

*Listing 17-15: Anonymous futures ko ek vector mein store karna aur `join_all` call karna*

Durbhagya se, yah code compile nahi hota hai. Humein yah error milta hai:

```text
error[E0308]: mismatched types
   = note: no two async blocks, even if identical, have the same type
```

Yah shayad aashcharyajanak ho. Aakhirkaar, har async block ek `Future<Output = ()>` paida karta hai. Lekin yaad rakhein ki `Future` ek trait hai, aur compiler har `async` block ke liye ek anokha,गुमनाम (anonymous) type banata hai. Aap ek `Vec` mein do alag-alag structs nahi daal sakte, aur yahi niyam compiler dwara generate kiye gaye alag-alag types par bhi laagu hota hai.

Ise kaam karne ke liye, humein **trait objects** ka istemal karna hoga. Trait objects ka istemal karke hum in types dwara paida kiye gaye har anonymous future ko ek hi type ke roop mein treat kar sakte hain, kyunki ve sabhi `Future` trait ko implement karte hain.

Iske liye humein kuch kadam uthane honge. Pehle, hum har future ko `Box::new` mein wrap karte hain. Lekin yah abhi bhi kaafi nahi hai. Humein `futures` variable ke type ko spasht roop se batana hoga taaki compiler samjhe ki hum trait objects ka istemal karna chahte hain. Iske baad bhi, humein ek naya error milega jo `Unpin` trait se sambandhit hai.

**`Pin` aur `Unpin`** async Rust ke advanced concepts hain. Abhi ke liye, hum bas compiler ki salaah maan sakte hain. Antim, sahi code `Pin` ka istemal karta hai, jo ek future ko memory mein "pin" kar deta hai, aur `Box::pin` ka istemal har future ko heap par allocate karke use pin karne ke liye karta hai.

```rust
// Filename: src/main.rs
use std::pin::Pin;
use std::future::Future;
// ...
fn main() {
    run(async {
        // ... (tx, rx, tx1 setup) ...
        let tx_fut = async move { /* ... */ };
        let rx_fut = async { /* ... */ };
        let tx1_fut = async move { /* ... */ };

        let mut futures: Vec<Pin<Box<dyn Future<Output = ()>>>> = vec![];
        futures.push(Box::pin(tx_fut));
        futures.push(Box::pin(rx_fut));
        futures.push(Box::pin(tx1_fut));
        
        trpl::join_all(futures).await;
    });
}
```

*Listing 17-18: `Vec` ko type check karvane ke liye `Pin` aur `Box::pin` ka istemal karna*

Isse code aakhirkaar compile aur run ho jaata hai\!

Yah ek maulik tradeoff hai: hum `join_all` ke saath dynamic sankhya ke futures ke saath kaam kar sakte hain, jab tak ki un sabka type ek hi ho (trait objects ke zariye). Ya hum `join!` macro ke saath ek nishchit sankhya ke futures ke saath kaam kar sakte hain, bhale hi unke types alag-alag hon.

-----

### Racing Futures

Jab hum `join` ke saath futures ko "join" karte hain, to hum un sabhi ke poora hone ka intezaar karte hain. Kabhi-kabhi, humein sirf ek set se *kuch* futures ke poora hone ki zaroorat hoti hai.

Listing 17-21 mein, hum do futures, `slow` aur `fast`, ko ek doosre ke khilaaf race karvane ke liye `trpl::race` ka istemal karte hain.

```rust
// Filename: src/main.rs
// ...
fn main() {
    run(async {
        let slow = async {
            println!("'slow' started.");
            trpl::sleep(Duration::from_secs(2)).await;
            println!("'slow' finished.");
        };

        let fast = async {
            println!("'fast' started.");
            trpl::sleep(Duration::from_secs(1)).await;
            println!("'fast' finished.");
        };

        trpl::race(slow, fast).await;
    });
}
```

*Listing 17-21: Jo future pehle poora hota hai, uska result prapt karne ke liye `race` ka istemal karna*

Ismein koi aashcharya nahi hai: `fast` jeet jaata hai. Dhyan dein ki `race` function ka yah implementation **fair nahi hai**. Yah hamesha arguments ko unke pass kiye gaye kram mein chalata hai.

Yaad rakhein ki har `await` point par, Rust runtime ko task pause karne aur doosre par switch karne ka mauka deta hai. Iska ulta bhi sach hai: Rust *keval* ek `await` point par hi async blocks ko pause karta hai. `await` points ke beech ka sab kuch synchronously chalta hai.

Iska matlab hai ki agar aap ek async block mein bina `await` point ke bahut saara kaam karte hain, to vah future doosre futures ko progress karne se rok dega. Ise kabhi-kabhi ek future ka doosre futures ko **starve** karna (bhukha maarna) kaha jaata hai.

-----

### Runtime ko Control Yield Karna

Chaliye ek lamba chalne wala, blocking operation simulate karte hain `std::thread::sleep` ka istemal karke. Agar hum ise do futures ke andar istemal karte hain, to hum dekhenge ki unke kaam mein koi interleaving nahi hogi. Future 'a' apne pehle `await` tak poora kaam karega, phir future 'b' apne pehle `await` tak poora kaam karega.

Futures ko unke slow tasks ke beech progress karne dene ke liye, humein `await` points ki zaroorat hai taaki hum control runtime ko vaapas de sakein. Hum `slow` operations ke beech `trpl::sleep(...).await` calls daal sakte hain. Isse dono futures ka kaam interleave ho jayega.

Lekin hum yahan vastav mein *sona* (sleep) nahi chahte; hum jitni jaldi ho sake progress karna chahte hain. Hum bas control runtime ko vaapas dena chahte hain. Hum yah `yield_now` function ka istemal karke seedhe kar sakte hain.

```rust
// Filename: src/main.rs
// ... (slow function definition) ...
fn main() {
    run(async {
        let a = async {
            println!("'a' started.");
            slow(30);
            trpl::yield_now().await; // Yield control
            slow(10);
            trpl::yield_now().await; // Yield control
            slow(20);
            println!("'a' finished.");
        };
        // ... (similar for future 'b') ...
        trpl::race(a, b).await;
    });
}
```

*Listing 17-25: `yield_now` ka istemal karke operations ko switch karne dena*

Yah code na keval asli iraade ke baare mein zyada spasht hai, balki `sleep` ka istemal karne se kaafi tez bhi ho sakta hai.

Yah **cooperative multitasking** ka ek roop hai, jahan har future tay karta hai ki vah `await` points ke madhyam se control kab saunpega.

-----

### Apne Khud ke Async Abstractions Banana

Hum naye patterns banane ke liye futures ko ek saath compose bhi kar sakte hain. Udaharan ke liye, hum `timeout` function bana sakte hain.

`timeout` ka idea `race` aur `sleep` ka istemal karna hai: hum diye gaye future ko ek `sleep` future ke khilaaf race karvate hain.

  * Agar hamara future pehle poora hota hai, to hum `Ok(result)` return karte hain.
  * Agar `sleep` timer pehle poora hota hai, to hum `Err(timeout_duration)` return karte hain.

<!-- end list -->

```rust
// Filename: src/main.rs
use trpl::{sleep, race, Either};
use std::time::Duration;
use std::future::Future;

async fn timeout<F>(
    future_to_try: F,
    max_time: Duration,
) -> Result<F::Output, Duration>
where
    F: Future,
{
    let timer = sleep(max_time);
    match race(future_to_try, timer).await {
        Either::Left(output) => Ok(output),
        Either::Right(_) => Err(max_time),
    }
}
```

*Listing 17-29: `race` aur `sleep` ke saath `timeout` define karna*

Iske saath, hamare paas do anya async helpers se bana ek kaam karne wala `timeout` hai. Kyunki futures doosre futures ke saath compose hote hain, aap chhote async building blocks ka istemal karke bahut shaktishaali tools bana sakte hain.