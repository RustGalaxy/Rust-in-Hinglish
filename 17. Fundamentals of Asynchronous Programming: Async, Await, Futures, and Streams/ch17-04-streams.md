## Streams: Kram mein Futures (Futures in Sequence)

Ab tak is chapter mein, humne zyada tar individual futures par hi dhyan diya hai. Ek bada apvaad (exception) async channel tha jiska humne istemal kiya. Async `recv` method samay ke saath items ka ek kram (sequence) paida karta hai. Yah ek bahut hi aam pattern ka ek udaharan hai jise **stream** ke roop mein jaana jaata hai.

Ek stream iteration ka ek asynchronous roop jaisa hai. Jabki `trpl::Receiver` khaas taur par messages prapt karne ka intezaar karta hai, general-purpose stream API bahut vishal hai: yah `Iterator` ki tarah agla item pradan karta hai, lekin asynchronously.

Iterators aur streams ke beech ki samanta ka matlab hai ki hum vastav mein kisi bhi iterator se ek stream bana sakte hain. Ek iterator ki tarah, hum ek stream ke saath uske `next` method ko call karke aur phir output ka `await` karke kaam kar sakte hain.

```rust,ignore,does_not_compile
// ...
let stream = trpl::stream_from_iter(iter);
while let Some(value) = stream.next().await {
    println!("Got a value: {value}");
}
```

*Listing 17-30: Ek iterator se ek stream banana aur uske values ko print karna*

Durbhagya se, jab hum code chalane ki koshish karte hain, to yah compile nahi hota, balki yah batata hai ki `next` method uplabdh nahi hai:

```console
error[E0599]: no method named `next` found for struct `Iter` in the current scope
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
```

Jaisa ki is output mein samjhaya gaya hai, compiler error ka kaaran yah hai ki humein `next` method ka istemal karne ke liye sahi trait ko scope mein laana hoga. Vah trait `Stream` nahi, balki **`StreamExt`** hai. `Ext` (Extension ka short form) Rust community mein ek trait ko doosre ke saath vistarit karne ke liye ek aam pattern hai.

`Stream` trait ek low-level interface define karta hai. `StreamExt` `Stream` ke upar ek higher-level set of APIs pradan karta hai, jismein `next` method ke saath-saath `Iterator` trait dwara pradan kiye gaye anya utility methods bhi shaamil hain.

Compiler error ko theek karne ke liye `trpl::StreamExt` ke liye ek `use` statement jodna hai.

```rust
// Filename: src/main.rs
use trpl::{run, stream_from_iter, StreamExt};

fn main() {
    run(async {
        let nums = [1, 2, 3, 4, 5];
        let mut stream = stream_from_iter(nums.into_iter().map(|n| n * 2));

        while let Some(value) = stream.next().await {
            println!("Got a value: {value}");
        }
    });
}
```

*Listing 17-31: Ek iterator ko ek stream ke aadhar ke roop mein safaltapoorvak istemal karna*

Ab jab hamare paas `StreamExt` scope mein hai, to hum uske sabhi utility methods ka istemal kar sakte hain, bilkul iterators ki tarah. Udaharan ke liye, hum `filter` method ka istemal kar sakte hain.

-----

### Streams ko Compose Karna

Kai concepts svabhavik roop se streams ke roop mein represent kiye jaate hain: ek queue mein uplabdh hone wale items, network par samay ke saath aane wala data, aadi. Kyunki streams futures hain, hum unhe kisi bhi anya prakaar ke future ke saath istemal kar sakte hain aur unhe dilchasp tarikon se jod sakte hain.

Chaliye messages ka ek chhota stream banakar shuruaat karte hain jo ek real-time communication protocol se milne wale data stream ka pratinidhitv karega.

```rust
// Filename: src/main.rs
use trpl::{channel, run, Stream, StreamExt, ReceiverStream};

fn get_messages() -> impl Stream<Item = String> {
    let (mut tx, rx) = channel(10);
    for i in 0..10 {
        tx.send(format!("Message {i}"));
    }
    ReceiverStream::new(rx)
}

fn main() {
    run(async {
        let messages = get_messages();
        let mut messages = std::pin::pin!(messages);

        while let Some(message) = messages.next().await {
            println!("Message: '{message}'");
        }
    });
}
```

*Listing 17-33: `rx` receiver ko `ReceiverStream` ke roop mein istemal karna*

Yahan, hum ek naye type ka istemal karte hain: **`ReceiverStream`**, jo `trpl::channel` se `rx` receiver ko ek `Stream` mein badal deta hai jiska `next` method hota hai.

Ab chaliye ek aisi feature jodte hain jiske liye streams ki zaroorat hai: ek timeout jodna jo stream ke har item par laagu hota hai, aur hamare bheje jaane wale items par ek delay.

```rust
// ... updated get_messages function
use trpl::{channel, run, sleep, spawn_task, Stream, ReceiverStream};
use std::time::Duration;

fn get_messages() -> impl Stream<Item = String> {
    let (mut tx, rx) = channel(10);

    spawn_task(async move {
        let messages = ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j"];
        for (i, message) in messages.into_iter().enumerate() {
            let delay_ms = if i % 2 == 0 { 100 } else { 300 };
            sleep(Duration::from_millis(delay_ms)).await;
            if tx.send(message.to_string()).await.is_err() {
                break;
            }
        }
    });

    ReceiverStream::new(rx)
}
```

*Listing 17-35: `get_messages` ko async function banaye bina async delay ke saath `tx` ke madhyam se messages bhejna*

Messages ke beech `sleep` karne ke liye, humein async ka istemal karna hoga. Lekin, hum `get_messages` ko khud ek `async` function nahi bana sakte, kyunki tab yah `Stream` ke bajaye `Future<Output = Stream>` return karega. Await karne se poora loop chal jayega *is se pehle* ki stream bhi uplabdh ho, jisse per-item timeout bekaar ho jayega.

Iske bajaye, hum `get_messages` ko ek regular function ke roop mein chhod dete hain jo ek stream return karta hai, aur hum async `sleep` calls ko handle karne ke liye ek task spawn karte hain. Isse hamara timeout ab kaam karega, aur humein kuch messages ke liye timeout errors milenge.

-----

### Streams ko Merge Karna

Hum streams ko `merge` method se combine kar sakte hain, jo kai streams ko ek stream mein jodta hai jo kisi bhi source stream se items jaise hi uplabdh hote hain, paida karta hai.

Lekin `merge` kaam kare, iske liye dono streams ka item type ek jaisa hona chahiye. Agar ek stream `String` bhej raha hai aur doosra `u32`, to humein `map` jaise methods ka istemal karke ek stream ko transform karna hoga taaki unke types match karen.

Aakhri code do streams ko merge karta hai: ek messages ka aur ek har 100 milliseconds mein ek counter bhejta hai. `take(20)` ka istemal karke program 20 items ke baad band ho jaata hai.

```rust
// Filename: src/main.rs
// ... (get_messages and get_intervals functions) ...
fn main() {
    run(async {
        // ...
        let messages_with_timeout = messages.timeout(Duration::from_millis(200));

        let intervals_as_messages = intervals
            .throttle(Duration::from_millis(100))
            .map(|i| format!("Interval: {i}"))
            .timeout(Duration::from_secs(10));

        let mut stream = messages_with_timeout.merge(intervals_as_messages);
        let mut stream = std::pin::pin!(stream.take(20));
        
        // ... while let loop to print items ...
    });
}
```

*Listing 17-39: Merged streams ko manage karne ke liye `throttle` aur `take` ka istemal karna*

Ismein do naye methods ka istemal kiya gaya hai:

  * **`.throttle()`**: `intervals` stream par iska istemal kiya gaya hai taaki yah `messages` stream par haavi na ho. Throttling ek stream ke poll hone ki dar ko seemit karne ka ek tareeka hai.
  * **`.take()`**: `merged` stream par iska istemal kiya gaya hai taaki hum ek nishchit sankhya mein items ko process karne ke baad program ko band kar sakein.

Yah Rust ke futures ki "laziness" ko kaam mein laata hai, jisse hum apne performance characteristics chun sakte hain.

Aakhri cheez jo humein handle karni hai, vah hai errors\! `send` calls fail ho sakte hain. Ab tak, humne `unwrap` ka istemal kiya hai, lekin ek acche app mein, humein error ko spasht roop se handle karna chahiye, kam se kam loop ko `break` karke.