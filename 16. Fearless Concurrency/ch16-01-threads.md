## Ek Saath Code Chalane ke Liye Threads ka Istemal

Zyadatar current operating systems mein, ek execute kiye gaye program ka code ek **process** mein chalta hai, aur operating system ek saath multiple processes ko manage karta hai. Apne program ke andar, aapke paas independent hisse bhi ho sakte hain jo ek saath chalte hain. In independent hisson ko chalane wale features ko **threads** kaha jaata hai.

Apne program ke computation ko multiple threads mein baantne se performance behtar ho sakti hai kyunki program ek hi samay mein kai kaam karta hai, lekin isse jatilta (complexity) bhi badh jaati hai. Kyunki threads ek saath chal sakte hain, is baat ki koi anivarya guarantee nahi hai ki aapke code ke alag-alag threads par chalne wale hisse kis kram (order) mein chalenge. Isse samasyaayein ho sakti hain, jaise:

  * **Race conditions**, jahan threads data ya resources ko ek asangat (inconsistent) kram mein access kar rahe hain.
  * **Deadlocks**, jahan do threads ek doosre ka intezaar kar rahe hain ki woh us resource ka istemal khatm kare jo doosre thread ke paas hai, jisse dono threads aage nahi badh paate.
  * Bugs jo sirf kuch khaas paristhitiyon mein hote hain aur jinhe vishvasniya roop se reproduce karna aur theek karna mushkil hota hai.

Rust threads ka istemal karne ke nakaratmak prabhavon ko kam karne ki koshish karta hai, lekin ek multithreaded context mein programming karne ke liye abhi bhi savdhani se sochne ki zaroorat hai aur ek aise code structure ki avashyakta hai jo single thread mein chalne wale programs se alag ho.

-----

Programming languages threads ko kuch alag-alag tarikon se implement karti hain. Kai operating systems naye threads banane ke liye ek API pradan karte hain. Yeh model jahan ek language operating system APIs ko call karke threads banati hai, kabhi-kabhi **1:1** kehlata hai, jiska matlab hai ek language thread ke liye ek operating system thread.

Kai programming languages threads ka apna khaas implementation pradan karti hain. Programming language dwara pradan kiye gaye threads ko **green threads** ke roop mein jaana jaata hai, aur jo languages in green threads ka istemal karti hain, ve unhe alag-alag sankhya ke operating system threads ke context mein execute karengi. Is karan se, green-threaded model ko **M:N** model kaha jaata hai: `N` operating system threads ke liye `M` green threads hote hain, jahan `M` aur `N` zaroori nahi ki samaan sankhya ho.

Har model ke apne fayde aur trade-offs hote hain, aur Rust ke liye sabse mahatvapurna trade-off runtime support hai. **Runtime** ek bhramit karne wala shabd hai aur alag-alag sandarbhon mein iske alag-alag arth ho sakte hain.

Is sandarbh mein, **runtime** se hamara matlab us code se hai jo language dwara har binary mein shaamil kiya jaata hai. Yeh code language ke aadhar par bada ya chhota ho sakta hai, lekin har non-assembly language mein kuch had tak runtime code hoga. Is karan se, aam bolchaal mein jab log kehte hain ki ek language ka "koi runtime nahi hai," to unka aksar matlab hota hai "chhota runtime." Chhote runtimes mein kam features hote hain lekin unka fayda yeh hota hai ki ve chhote binaries banate hain, jisse language ko anya languages ke saath adhik sandarbhon mein jodna aasan ho jaata hai. Halaanki kai languages adhik features ke badle runtime size badhane mein theek hain, Rust ko lagbhag koi runtime nahi chahiye aur performance banaye rakhne ke liye C mein call karne ki kshamata se samjhauta nahi kar sakta.

Green-threading M:N model ko threads manage karne ke liye ek bade language runtime ki avashyakta hoti hai. Isliye, Rust standard library keval 1:1 threading ka implementation pradan karti hai. Kyunki Rust ek low-level language hai, aisi crates maujood hain jo M:N threading implement karti hain agar aap overhead ke badle kuch pehluon jaise ki kaun se threads kab chalenge is par adhik niyantran aur context switching ki kam laagat chahte hain.

Ab jab humne Rust mein threads ko paribhashit kar liya hai, aaiye standard library dwara pradan kiye gaye thread-related API ka istemal karna seekhen.

-----

### `spawn` ke Saath Ek Naya Thread Banana

Ek naya thread banane ke liye, hum `thread::spawn` function ko call karte hain aur use ek closure (humne Chapter 13 mein closures ke baare mein baat ki thi) pass karte hain jismein woh code hota hai jise hum naye thread mein chalana chahte hain. Listing 16-1 ka udaharan ek main thread se kuch text aur ek naye thread se anya text print karta hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

\<span class="caption"\>Listing 16-1: Ek naya thread banana jo ek cheez print karta hai jabki main thread kuch aur print karta hai\</span\>

Dhyan dein ki is function ke saath, naya thread tab band ho jayega jab main thread samapt hoga, chahe usne apna kaam poora kiya ho ya nahi. Is program ka output har baar thoda alag ho sakta hai, lekin yeh neeche diye gaye jaisa dikhega:

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

`thread::sleep` ka call ek thread ko uske execution ko thodi der ke liye rokne par majboor karta hai, jisse ek alag thread ko chalne ka mauka milta hai. Threads shayad baari-baari se chalenge, lekin iski guarantee nahi hai: yeh is par nirbhar karta hai ki aapka operating system threads ko kaise schedule karta hai. Is run mein, main thread ne pehle print kiya, bhale hi spawned thread ka print statement code mein pehle aata hai. Aur bhale hi humne spawned thread ko `i` 9 tak print karne ke liye kaha tha, woh keval 5 tak hi pahunch paaya isse pehle ki main thread band ho gaya.

-----

### Sabhi Threads ke Khatm Hone ka Intezaar Karna `join` Handles ke Saath

Listing 16-1 ka code na keval spawned thread ko samay se pehle rok deta hai, balki is baat ki bhi guarantee nahi de sakta ki spawned thread ko chalne ka mauka milega bhi ya nahi. Iska kaaran yeh hai ki threads ke chalne ke kram par koi guarantee nahi hai\!

Hum `thread::spawn` ke return value ko ek variable mein save karke is samasya ko theek kar sakte hain. `thread::spawn` ka return type `JoinHandle` hai. Ek `JoinHandle` ek owned value hai, jab hum is par `join` method call karte hain, to yeh apne thread ke samapt hone ka intezaar karega. Listing 16-2 dikhata hai ki kaise `JoinHandle` ka istemal karke yeh sunishchit kiya jaaye ki spawned thread `main` se bahar nikalne se pehle samapt ho jaaye:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

\<span class="caption"\>Listing 16-2: `thread::spawn` se ek `JoinHandle` save karna taaki yeh guarantee ho ki thread poora chalega\</span\>

Handle par `join` call karna vartaman mein chal rahe thread ko **block** kar deta hai jab tak ki handle dwara represent kiya gaya thread samapt nahi ho jaata. Ek thread ko *block* karne ka matlab hai ki us thread ko kaam karne ya bahar nikalne se roka jaata hai. Kyunki humne `join` ka call main thread ke `for` loop ke baad rakha hai, Listing 16-2 ko chalane par is tarah ka output aana chahiye:

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Dono threads baari-baari se chalte rehte hain, lekin main thread `handle.join()` ke call ke kaaran intezaar karta hai aur tab tak samapt nahi hota jab tak ki spawned thread samapt nahi ho jaata.

Lekin aaiye dekhen ki kya hota hai jab hum `handle.join()` ko `main` mein `for` loop se pehle le jaate hain:

```rust
// ...
    handle.join().unwrap();

    for i in 1..5 {
// ...
```

Main thread spawned thread ke samapt hone ka intezaar karega aur phir apna `for` loop chalayega, isliye output ab interleaved nahi hoga, jaisa ki yahan dikhaya gaya hai:

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
// ... (up to 9)
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
// ... (up to 4)
hi number 4 from the main thread!
```

Chhoti-chhoti baatein, jaise ki `join` kahan call kiya jaata hai, is baat par asar daal sakti hain ki aapke threads ek hi samay par chalte hain ya nahi.

-----

### Threads ke Saath `move` Closures ka Istemal

`move` closure ka istemal aksar `thread::spawn` ke saath kiya jaata hai kyunki yeh aapko ek thread se data doosre thread mein istemal karne ki anumati deta hai.

Chapter 13 mein, humne bataya tha ki hum `move` keyword ka istemal closure ki parameter list se pehle kar sakte hain taaki closure ko un values ka ownership lene ke liye force kiya ja sake jinhe woh environment mein istemal karta hai. Yeh technique khaas taur par naye threads banate samay values ka ownership ek thread se doosre mein transfer karne ke liye upyogi hai.

Listing 16-3 main thread mein ek vector banane aur use spawned thread mein istemal karne ki koshish karta hai. Lekin, yeh abhi kaam nahi karega.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

\<span class="caption"\>Listing 16-3: Main thread dwara banaye gaye vector ko doosre thread mein istemal karne ki koshish\</span\>

Jab hum is example ko compile karte hain, to humein yeh error milta hai:

```text
error[E0373]: closure may outlive the current function, but it borrows `v`,
which is owned by the current function
  |
help: to force the closure to take ownership of `v` (and any other referenced
variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ^^^^^^^
```

Rust *andaza lagata hai* ki `v` ko kaise capture karna hai, aur kyunki `println!` ko sirf `v` ka reference chahiye, closure `v` ko borrow karne ki koshish karta hai. Lekin, ek samasya hai: Rust yeh nahi bata sakta ki spawned thread kitni der tak chalega, isliye use nahi pata ki `v` ka reference hamesha valid rahega ya nahi. Agar main thread spawned thread ke execute hone se pehle hi `v` ko drop kar de, to reference invalid ho jayega.

Compiler error ke sujhav ka palan karke hum is samasya ko theek kar sakte hain. Closure se pehle `move` keyword jodkar, hum closure ko un values ka **ownership** lene ke liye force karte hain jinhe woh istemal kar raha hai. Listing 16-5 mein dikhaya gaya aavedan compile hoga aur jaisa hum chahte hain waisa chalega:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

\<span class="caption"\>Listing 16-5: `move` keyword ka istemal karke closure ko un values ka ownership lene ke liye force karna jinhe woh istemal karta hai\</span\>

Agar hum us code mein `move` closure ka istemal karte jahan main thread `drop` call kar raha tha, to kya hota? Tab bhi yeh kaam nahi karta; humein ek alag error milta. Agar hum closure mein `move` jodte, to hum `v` ko closure ke environment mein move kar dete, aur hum ab use main thread mein drop nahi kar paate.

Rust ke ownership rules ne humein phir se bacha liya\! `move` keyword Rust ke conservative default of borrowing ko override karta hai; yeh humein ownership rules ka ullanghan nahi karne deta.

Ab jab humein threads aur thread API ki buniyadi samajh ho gayi hai, aaiye dekhen ki hum threads ke saath kya kar sakte hain.