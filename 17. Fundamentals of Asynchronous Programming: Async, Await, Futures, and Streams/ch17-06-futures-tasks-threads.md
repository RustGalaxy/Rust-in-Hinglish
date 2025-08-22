## Sabko Ek Saath Jodna: Futures, Tasks, aur Threads

Jaisa ki humne Chapter 16 mein dekha, threads concurrency ke liye ek approach pradan karte hain. Humne is chapter mein ek aur approach dekha hai: futures aur streams ke saath async ka istemal karna. Agar aap soch rahe hain ki ek method ko doosre par kab chunna hai, to jawab hai: yah nirbhar karta hai\! Aur kai maamlon mein, chunav threads *ya* async nahi, balki threads *aur* async hota hai.

Kai operating systems ab dashakon se threading-based concurrency models pradan kar rahe hain. Halaanki, in models ke apne tradeoffs hain. Kai operating systems par, ve har thread ke liye kaafi memory istemal karte hain, aur unhe shuru karne aur band karne mein kuch overhead aata hai. Threads tabhi ek vikalp hain jab aapka operating system aur hardware unhe support karte hain. Kuch embedded systems mein koi OS nahi hota hai, isliye unmein threads bhi nahi hote hain.

Async model ek alag—aur antतः poorak (complementary)—set of tradeoffs pradan karta hai. Async model mein, concurrent operations ko apne khud ke threads ki zaroorat nahi hoti. Iske bajaye, ve **tasks** par chal sakte hain. Ek task ek thread ke samaan hai, lekin operating system dwara prabandhit hone ke bajaye, yah library-level code dwara prabandhit hota hai: **runtime**.

Hum `std::thread` APIs ka istemal `get_intervals` function ke liye kar sakte hain, jaisa ki Listing 17-41 mein dikhaya gaya hai, aur baaki code par koi asar nahi padega.

```rust
// Filename: src/main.rs
use std::{thread, time::Duration};
use trpl::{channel, ReceiverStream, Stream};

fn get_intervals() -> impl Stream<Item = u32> {
    let (mut tx, rx) = channel(4);
    thread::spawn(move || {
        let mut count = 0;
        loop {
            thread::sleep(Duration::from_millis(1));
            count += 1;
            if tx.try_send(count).is_err() {
                break;
            }
        }
    });
    ReceiverStream::new(rx)
}
// ... (main function remains the same as before)
```

*Listing 17-41: `get_intervals` function ke liye async `trpl` APIs ke bajaye `std::thread` APIs ka istemal karna*

Is code ka output bilkul pehle jaisa hi hoga. Isse pata chalta hai ki hum dono models ko mix kar sakte hain. Unki samantaon ke bawajood, in dono approaches ka vyavhaar bahut alag hota hai. Hum kisi bhi modern personal computer par laakhon async tasks spawn kar sakte hain. Agar hum threads ke saath aisa karne ki koshish karte, to hamari memory hi khatm ho jaati\!

Threads synchronous operations ke sets ke liye ek seema (boundary) ke roop mein kaam karte hain; concurrency threads *ke beech* sambhav hai. Tasks *asynchronous* operations ke sets ke liye ek seema ke roop mein kaam karte hain; concurrency tasks *ke beech* aur *unke andar* dono jagah sambhav hai. Antतः, futures Rust ki concurrency ki sabse choti (granular) unit hain.

Iska matlab yah nahi hai ki async tasks hamesha threads se behtar hote hain. Threads ke saath concurrency kuch maamlon mein `async` se saral programming model hai. Ve "fire and forget" jaise hote hain. Threads mein cancellation ke liye koi native mechanism nahi hota.

In seemaon ke kaaran threads ko futures se compose karna bhi kathin hota hai. Futures, data structures ke roop mein zyada rich hone ke kaaran, adhik svabhavik roop se ek saath compose kiye ja sakte hain, jaisa ki humne dekha hai.

Aur yah pata chalta hai ki threads aur tasks aksar ek saath bahut acchi tarah se kaam karte hain, kyunki tasks threads ke beech move kiye ja sakte hain. Vastav mein, parde ke peeche, hum jo runtime istemal kar rahe hain, vah default roop se multithreaded hai\! Kai runtimes **work stealing** naamak ek approach ka istemal karte hain, jisse ve threads ke beech tasks ko anukoolit roop se move karte hain, taaki system ki overall performance behtar ho.

Kaun sa method kab istemal karna hai, iske baare mein sochte samay, in anubhavjanya niyamon (rules of thumb) par vichaar karein:

  * 👍 Agar kaam **bahut zyada parallelizable** hai, jaise ki data ke ek samooh ko process karna jahan har hisse ko alag se process kiya ja sakta hai, to **threads** ek behtar vikalp hain.
  * ⚡ Agar kaam **bahut zyada concurrent** hai, jaise ki kai alag-alag sroton se aane wale messages ko handle karna jo alag-alag samay ya dar par aa sakte hain, to **async** ek behtar vikalp hai.

Aur agar aapko parallelism aur concurrency dono ki zaroorat hai, to aapko threads aur async ke beech chunav karne ki zaroorat nahi hai. Aap unhe ek saath svatantra roop se istemal kar sakte hain. Udaharan ke liye, Listing 17-42 is prakaar ke mix ka ek aam udaharan dikhata hai.

```rust
// Filename: src/main.rs
use std::{thread, time::Duration};
use trpl::{channel, run, StreamExt};

fn main() {
    let (mut tx, mut rx) = channel(4);

    thread::spawn(move || {
        for i in 1..=10 {
            tx.try_send(i).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    run(async {
        while let Some(i) = rx.next().await {
            println!("Got {i}");
        }
    });
}
```

*Listing 17-42: Ek thread mein blocking code ke saath messages bhejna aur async block mein messages ka `await` karna*

Hum ek async channel banakar shuruaat karte hain, phir ek thread spawn karte hain jo channel ke sender side ka ownership le leta hai. Thread ke andar, hum 1 se 10 tak sankhyaएं bhejte hain, har ek ke beech ek second ke liye sote hain. Ant mein, hum `trpl::run` ko pass kiye gaye ek async block dwara banaye gaye future ko chalate hain. Us future mein, hum un messages ka `await` karte hain.

-----

## Saaraansh (Summary)

Yah is pustak mein concurrency ka ant nahi hai. Chapter 21 mein project in avadharanaon ko yahan charcha kiye gaye saral udaharanon se adhik yatharthvadi sthiti mein laagu karega.

Aap inmein se koi bhi approach chunein, Rust aapko surakshit, tez, concurrent code likhne ke liye zaroori tools deta hai—chahe vah ek high-throughput web server ho ya ek embedded operating system.

Aage, hum samasyaon ko model karne aur samadhanon ko sanrachit karne ke liye lokpriya (idiomatic) tarikon ke baare mein baat karenge jab aapke Rust programs bade ho jaate hain. Iske alawa, hum charcha karenge ki Rust ke idioms un idioms se kaise sambandhit hain jinse aap object-oriented programming se parichit ho sakte hain.