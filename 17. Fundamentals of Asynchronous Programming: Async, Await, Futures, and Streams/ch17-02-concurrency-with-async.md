## Async ke Saath Concurrency Laagu Karna

Is section mein, hum async ko unhi concurrency chunautiyon par laagu karenge jinhe humne Chapter 16 mein threads ke saath tackle kiya tha. Is section mein hum threads aur futures ke beech kya alag hai, us par dhyan kendrit karenge.

Kai maamlon mein, async ka istemal karke concurrency ke liye APIs threads ke istemal ke liye APIs se bahut milte-julte hain. Doosre maamlon mein, ve kaafi alag hote hain. Bhale hi APIs threads aur async ke beech ek jaise *dikhein*, unka vyavhaar aksar alag hota hai—aur unke performance characteristics lagbhag hamesha alag hote hain.

-----

### `spawn_task` ke Saath Ek Naya Task Banana

Pehla operation jo humne threads ke saath kiya tha, woh do alag-alag threads par ginti karna tha. Chaliye wahi async ka istemal karke karte hain. `trpl` crate ek `spawn_task` function pradan karta hai jo `thread::spawn` API se bahut milta-julta hai, aur ek `sleep` function jo `thread::sleep` API ka async version hai. Hum inhein ek saath istemal karke ginti ka udaharan implement kar sakte hain, jaisa ki Listing 17-6 mein dikhaya gaya hai.

```rust
// Filename: src/main.rs
use trpl::{spawn_task, sleep, run};
use std::time::Duration;

fn main() {
    run(async {
        spawn_task(async {
            for i in 1..10 {
                println!("hi number {i} from the first task!");
                sleep(Duration::from_millis(500)).await;
            }
        });

        for i in 1..5 {
            println!("hi number {i} from the second task!");
            sleep(Duration::from_millis(500)).await;
        }
    });
}
```

*Listing 17-6: Ek naya task banana jo ek cheez print karta hai jabki main task kuch aur print karta hai*

> **Note:** Is chapter mein aage har udaharan mein `main` ke andar `trpl::run(async { ... });` ka istemal hoga, isliye hum ise aksar chhod denge. Apne code mein ise shaamil karna na bhoolein\!

Humne `trpl::spawn_task` ke body mein ek loop aur top-level `for` loop mein doosra loop rakha hai. Humne `sleep` calls ke baad ek `await` bhi joda hai.

Yeh code thread-based implementation ke samaan vyavhaar karta hai—aap apne terminal mein messages ko alag kram (order) mein dekh sakte hain:

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
...
```

Yah version jaise hi main async block ka `for` loop poora hota hai, ruk jaata hai, kyunki `spawn_task` dwara spawn kiya gaya task `main` function ke ant mein band ho jaata hai. Agar aap chahte hain ki vah task poora chale, to aapko join handle ka istemal karke pehle task ke poora hone ka intezaar karna hoga. Threads ke saath, humne `.join()` method ka istemal kiya tha. Yahan, hum **`await`** ka istemal kar sakte hain, kyunki task handle khud ek future hai.

```rust
// Filename: src/main.rs
// ...
fn main() {
    run(async {
        let handle = spawn_task(async {
            // ... first loop
        });

        // ... second loop

        handle.await.unwrap(); // task ke poora hone tak intezaar karein
    });
}
```

*Listing 17-7: Ek task ko poora chalane ke liye join handle ke saath `await` ka istemal karna*

Is updated version mein, dono loops poora hone tak chalte hain.

Ab tak, aisa lagta hai ki async aur threads humein ek hi jaise parinaam dete hain, bas syntax alag hai. Bada antar yeh hai ki humein aisa karne ke liye ek aur operating system thread spawn karne ki zaroorat nahi padi. Vastav mein, humein yahan tak ki ek task spawn karne ki bhi zaroorat nahi hai. Hum har loop ko ek `async` block mein daal sakte hain aur runtime ko `trpl::join` function ka istemal karke dono ko poora chalane de sakte hain.

`trpl::join` function futures ke liye `JoinHandle` ke `.join()` jaisa hai. Jab aap ise do futures dete hain, to yah ek naya future paida karta hai jiska output ek tuple hota hai jismein har future ka output hota hai, jab ve *dono* poore ho jaate hain.

```rust
// Filename: src/main.rs
// ...
fn main() {
    run(async {
        let fut1 = async {
            // ... first loop
        };

        let fut2 = async {
            // ... second loop
        };

        trpl::join(fut1, fut2).await;
    });
}
```

*Listing 17-8: Do anonymous futures ka `await` karne ke liye `trpl::join` ka istemal karna*

Ab, aapko har baar bilkul wahi kram dikhega, jo threads ke saath dekhe gaye se bahut alag hai. Aisa isliye hai kyunki `trpl::join` function **fair** (निष्पक्ष) hai, yaani yah har future ko barabar check karta hai. Threads ke saath, operating system tay karta hai ki kis thread ko check karna hai. Async Rust ke saath, runtime tay karta hai ki kis task ko check karna hai.

-----

### Message Passing ka Istemal Karke Do Tasks par Ginti Karna

Futures ke beech data share karna bhi jaana-pehchana lagega: hum phir se message passing ka istemal karenge, lekin is baar types aur functions ke async versions ke saath. Hum `trpl::channel` ka istemal karenge, jo Chapter 16 mein istemal kiye gaye channel API ka async version hai.

```rust
// Filename: src/main.rs
use trpl::{channel, run};

fn main() {
    run(async {
        let (mut tx, mut rx) = channel(4);
        tx.send(42).await;
        let msg = rx.recv().await;
        println!("Received message: {msg:?}");
    });
}
```

*Listing 17-9: Ek async channel banana*

Async version thoda alag hai: iska `recv` method ek future paida karta hai jise humein `await` karna padta hai na ki seedhe value deta hai.

Synchronous `Receiver::recv` method message milne tak **block** karta hai. `trpl::Receiver::recv` method block nahi karta, kyunki yah async hai. Block karne ke bajaye, yah control runtime ko vaapas de deta hai jab tak ki ya to ek message prapt na ho ya channel ka send side band na ho jaaye.

Is udaharan mein abhi tak koi concurrency nahi hai. Sab kuch kram mein hota hai. Chaliye har message ke beech `sleep` daalkar messages ki ek series bhejte hain.

Rust mein abhi tak asynchronous items ki series par `for` loop likhne ka koi tareeka nahi hai, isliye humein ek naya loop istemal karna hoga: **`while let`** conditional loop. Yah loop tab tak chalta rahega jab tak ki uske dwara nirdisht pattern value se match karta rahe.

```rust,ignore
// ...
while let Some(message) = rx.recv().await {
    println!("received '{message}'");
}
```

`rx.recv().await` ka result `Some(message)` hone par, humein message milta hai. Agar result `None` hai, to loop samapt ho jaata hai. Yah tab hota hai jab channel band ho jaata hai.

Is code mein abhi bhi do samasyaayein hain:

1.  Messages ek-ek karke aane ke bajaye, 2 second ke baad ek saath aate hain. Aisa isliye hai kyunki ek hi async block ke andar, `await` kram se execute hote hain. Abhi bhi koi concurrency nahi hai.
2.  Yah program kabhi samapt nahi hota\! Yah hamesha naye messages ka intezaar karta rehta hai.

Pehli samasya ko hal karne ke liye, humein `tx` (sender) aur `rx` (receiver) operations ko unke apne `async` blocks mein daalna hoga aur `trpl::join` ka istemal karna hoga.

```rust,ignore
// Filename: src/main.rs
// ...
let tx_fut = async {
    // ... sending logic with sleeps
};

let rx_fut = async {
    // ... receiving logic with `while let`
};

trpl::join(tx_fut, rx_fut).await;
```

*Listing 17-11: `send` aur `recv` ko alag `async` blocks mein daalna*

Isse messages sahi interval par aane lagte hain. Lekin program abhi bhi samapt nahi hota. Aisa isliye hai kyunki `rx` future tab tak poora nahi hoga jab tak `while let` loop samapt na ho, aur loop tab tak samapt nahi hoga jab tak channel band na ho. Channel tabhi band hoga jab `tx` (sender side) drop ho jaayega. Lekin `tx` tab tak drop nahi hoga jab tak poora program samapt na ho. Hum ek anant loop mein phans gaye hain\!

Hal yah hai ki `tx` ka ownership sending block mein **move** kar diya jaaye, taaki jab vah block samapt ho to `tx` drop ho jaaye. Hum ise `async` block ko `async move` mein badal kar karte hain.

```rust
// Filename: src/main.rs (Corrected version)
use trpl::{channel, join, run, sleep};
use std::time::Duration;

fn main() {
    run(async {
        let (mut tx, mut rx) = channel(4);

        let tx_fut = async move { // Note the 'move' keyword
            let vals = vec!["hi", "from", "the", "future"];
            for val in vals {
                tx.send(val).await;
                sleep(Duration::from_millis(500)).await;
            }
        };

        let rx_fut = async {
            while let Some(message) = rx.recv().await {
                println!("received '{message}'");
            }
        };

        join(tx_fut, rx_fut).await;
    });
}
```

*Listing 17-12: `async move` ka istemal karke program ko sahi dhang se band karna*

Is version ko chalane par, yah aakhri message bheje aur prapt kiye jaane ke baad aaram se band ho jaata hai.

Aap `tx` par `clone` call karke multiple futures se messages bhej sakte hain aur un sabhi ka `await` karne ke liye `trpl::join3` jaise functions ka istemal kar sakte hain.