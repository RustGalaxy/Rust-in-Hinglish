## Shared-State Concurrency (Sajha-Avastha Concurrency)

Message passing concurrency ko handle karne ka ek accha tareeka hai, lekin yeh ekmatra tareeka nahi hai. Go language ke documentation ke us naare ke is hisse par phir se gaur karein: "communicate by sharing memory" (memory share karke communicate karo).

Memory share karke communicate karna kaisa dikhega? Iske alawa, message-passing ke samarthak iska istemal kyun nahi karenge aur iske vipreet kaam kyun karenge?

Ek tarah se, kisi bhi programming language mein channels single ownership ke samaan hain, kyunki ek baar jab aap ek value ko channel ke neeche transfer kar dete hain, to aapko us value ka istemal nahi karna chahiye. Shared memory concurrency **multiple ownership** ki tarah hai: kai threads ek hi samay mein ek hi memory location tak pahunch sakte hain. Jaisa ki aapne Chapter 15 mein dekha, jahan smart pointers ne multiple ownership ko sambhav banaya, multiple ownership jatilta (complexity) badha sakti hai kyunki in alag-alag owners ko prabandhit (manage) karne ki zaroorat hoti hai. Rust ka type system aur ownership rules is prabandhan ko sahi tarike se karne mein bahut sahayata karte hain. Udaharan ke liye, aaiye mutexes ko dekhen, jo shared memory ke liye sabse aam concurrency primitives mein se ek hain.

-----

### Ek Samay par Ek Thread se Data Access Dene ke Liye Mutexes ka Istemal

**Mutex**, **mutual exclusion** ka sankshipt roop hai, is arth mein ki ek mutex kisi bhi samay mein keval ek thread ko kuch data tak pahunchne ki anumati deta hai. Ek mutex mein data tak pahunchne ke liye, ek thread ko pehle mutex ka **lock** haasil karne ke liye poochhkar yah sanket dena hoga ki vah access chahta hai. Lock ek data structure hai jo mutex ka hissa hai jo is baat ka track rakhta hai ki vartaman mein kiske paas data ka exclusive access hai. Isliye, mutex ko locking system ke madhyam se apne paas rakhe data ki **raksha** (guarding) karte hue varnit kiya jaata hai.

Mutexes ka upyog karna kathin maana jaata hai kyunki aapko do niyam yaad rakhne honge:

  * Data ka upyog karne se pehle aapko lock haasil karne ka prayas karna chahiye.
  * Jab aap mutex dwara surakshit data ke saath kaam kar chuke hon, to aapko data ko **unlock** karna chahiye taaki anya threads lock haasil kar sakein.

Mutex ke liye ek vastavik duniya ka roopak (metaphor), ek sammelan mein ek panel charcha ki kalpana karein jismein keval ek microphone ho. Bolne se pehle, ek panelist ko yah poochna ya sanket dena hota hai ki vah microphone ka upyog karna chahta hai. Jab unhe microphone mil jaata hai, to ve jab tak chahein tab tak baat kar sakte hain aur phir microphone agle panelist ko de sakte hain jo bolne ka anurodh karta hai. Agar ek panelist apna kaam khatm hone par microphone dena bhool jaata hai, to koi aur bol nahi paata hai.

Mutexes ka prabandhan sahi tarike se karna behad mushkil ho sakta hai, yahi vajah hai ki itne saare log channels ko lekar utsaahit hain. Halaanki, Rust ke type system aur ownership rules ke kaaran, aap locking aur unlocking mein galti **nahi** kar sakte.

-----

#### `Mutex<T>` ka API

Ek mutex ka upyog kaise karein, iske udaharan ke liye, aaiye ek single-threaded context mein ek mutex ka upyog karke shuruaat karein, jaisa ki Listing 16-12 mein dikhaya gaya hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

\<span class="caption"\>Listing 16-12: Saralta ke liye ek single-threaded context mein `Mutex<T>` ke API ki jaanch karna\</span\>

Hum `new` associated function ka upyog karke ek `Mutex<T>` banate hain. Mutex ke andar ke data tak pahunchne ke liye, hum lock haasil karne ke liye `lock` method ka upyog karte hain. Yah call vartaman thread ko **block** kar dega taaki jab tak hamari lock ki baari na aaye, vah koi kaam na kar sake.

`lock` ka call vifal ho sakta hai agar lock rakhne wala koi doosra thread panic ho jaaye. Us sthiti mein, koi bhi kabhi bhi lock prapt nahi kar payega, isliye humne `unwrap` chunaut kiya hai.

Lock haasil karne ke baad, hum return value (`num`) ko andar ke data ke liye ek mutable reference ke roop mein treat kar sakte hain. Type system yah sunishchit karta hai ki hum `m` mein value ka upyog karne se pehle ek lock haasil karein: `Mutex<i32>` ek `i32` nahi hai, isliye `i32` value ka upyog karne ke liye humein lock haasil karna **hi** hoga.

Jaisa ki aapko sandeh ho sakta hai, `Mutex<T>` ek smart pointer hai. Adhik sateek roop se, `lock` ka call **MutexGuard** naamak ek smart pointer return karta hai. Yah smart pointer hamare andar ke data par point karne ke liye `Deref` implement karta hai; smart pointer mein ek `Drop` implementation bhi hota hai jo lock ko **svachaalit roop se release** karta hai jab ek `MutexGuard` scope se bahar jaata hai. Parinaamsvaroop, hum lock release karna bhoolne ka jokhim nahi uthaate.

Lock drop karne ke baad, hum mutex value ko print kar sakte hain aur dekh sakte hain ki hum andar ke `i32` ko 6 mein badal paaye.

-----

#### Multiple Threads ke Beech `Mutex<T>` Share Karna

Ab, aaiye `Mutex<T>` ka upyog karke kai threads ke beech ek value share karne ka prayas karein. Hum 10 threads shuru karenge aur un sabhi se ek counter value ko 1 se badhvayenge, taaki counter 0 se 10 tak jaaye. Listing 16-13 mein hamara shuruaati udaharan hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

\<span class="caption"\>Listing 16-13: Das threads har ek `Mutex<T>` dwara surakshit ek counter ko badhaate hain\</span\>

Humne sanket diya tha ki yah udaharan compile nahi hoga. Ab pata lagate hain kyon\!

```text
error[E0382]: capture of moved value: `counter`
 --> src/main.rs:10:27
  |
9 |         let handle = thread::spawn(move || {
  |                                      ------- value moved (into closure) here
10|             let mut num = counter.lock().unwrap();
  |                           ^^^^^^^ value captured here after move
```

Error message batata hai ki `counter` value ko closure mein **move** kar diya gaya hai. Rust humein bata raha hai ki hum `counter` ka ownership kai threads mein move **nahi** kar sakte.

Chapter 15 mein, humne smart pointer `Rc<T>` ka upyog karke ek value ko kai owners diye the. Aaiye yahan bhi wahi karein aur dekhen kya hota hai.

-----

#### Multiple Threads ke Saath Multiple Ownership

Hum `Mutex<T>` ko `Rc<T>` mein wrap karenge aur thread mein ownership move karne se pehle `Rc<T>` ko clone karenge.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

\<span class="caption"\>Listing 16-14: Kai threads ko `Mutex<T>` ka owner banane ke liye `Rc<T>` ka upyog karne ka prayas\</span\>

Ek baar phir, hum compile karte hain aur... alag-alag errors prapt karte hain\!

```text
error[E0277]: `std::rc::Rc<std::sync::Mutex<i32>>` cannot be sent between threads safely
    |
    = help: the trait `Send` is not implemented for `std::rc::Rc<std::sync::Mutex<i32>>`
```

Yahan mahatvapurna baat yah hai ki: **`std::rc::Rc<std::sync::Mutex<i32>>` cannot be sent between threads safely** (`Rc<T>` ko threads ke beech surakshit roop se nahi bheja ja sakta). Hum agle section mein `Send` trait ke baare mein baat karenge.

Durbhagyavash, `Rc<T>` ko threads ke paar share karna surakshit nahi hai. Jab `Rc<T>` reference count ko manage karta hai, to vah count mein badlav ko kisi anya thread dwara baadhit hone se rokne ke liye kisi bhi concurrency primitive ka upyog nahi karta hai. Isse galat count ho sakte hain. Humein ek aise type ki zaroorat hai jo bilkul `Rc<T>` jaisa ho lekin reference count mein badlav ko thread-safe tarike se karta ho.

-----

#### `Arc<T>` ke Saath Atomic Reference Counting

Saubhagyavash, **`Arc<T>`** ek `Rc<T>` jaisa type hai jo concurrent sthitiyon mein upyog karne ke liye surakshit hai. **a** ka arth **atomic** hai, yaani yah ek **atomically reference counted** type hai. Atomics primitive types ki tarah kaam karte hain lekin threads ke paar share karne ke liye surakshit hote hain.

Thread safety ek performance penalty ke saath aati hai jise aap tabhi bhugatna chahte hain jab aapko iski zaroorat ho. Yahi kaaran hai ki sabhi primitive types atomic nahi hote hain.

`Arc<T>` aur `Rc<T>` ka API samaan hai, isliye hum apne program ko `use` line, `new` ke call, aur `clone` ke call ko badal kar theek karte hain. Listing 16-15 ka code antतः compile aur run hoga:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

\<span class="caption"\>Listing 16-15: Kai threads mein ownership share karne ke liye `Mutex<T>` ko wrap karne ke liye `Arc<T>` ka upyog karna\</span\>

Yah code nimnalikhit print karega:

```text
Result: 10
```

Humne kar dikhaya\! Humne 0 se 10 tak gina, jo shayad bahut prabhavshali na lage, lekin isne humein `Mutex<T>` aur thread safety ke baare mein bahut kuch sikhaya.

-----

#### `RefCell<T>`/`Rc<T>` aur `Mutex<T>`/`Arc<T>` ke Beech Samantaayein

Aapne shayad dhyan diya hoga ki `counter` immutable hai lekin hum uske andar ki value ka ek mutable reference prapt kar sakte hain; iska matlab hai ki `Mutex<T>` **interior mutability** pradan karta hai. Jaise humne `RefCell<T>` ka upyog `Rc<T>` ke andar contents ko mutate karne ke liye kiya tha, vaise hi hum `Arc<T>` ke andar contents ko mutate karne ke liye `Mutex<T>` ka upyog karte hain.

Ek aur baat dhyan dene yogy hai ki `Mutex<T>` ka upyog karte samay Rust aapko har tarah ki logic errors se nahi bacha sakta. `Mutex<T>` ke saath **deadlocks** banane ka jokhim hota hai. Aisa tab hota hai jab ek operation ko do resources ko lock karne ki zaroorat hoti hai aur do threads ne pratyek ne ek-ek lock haasil kar liya hota hai, jisse ve ek doosre ka hamesha ke liye intezaar karte rehte hain.

Hum is adhyay ko `Send` aur `Sync` traits ke baare mein baat karke samapt karenge.