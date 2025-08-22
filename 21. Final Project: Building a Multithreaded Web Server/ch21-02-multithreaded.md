Bilkul, yahan is chapter ke agle hisse ka Hinglish anuvaad hai:

## Apne Single-Threaded Server ko ek Multithreaded Server mein Badalna

Abhi, server har request ko ek-ek karke process karega, jiska matlab hai ki yeh doosre connection ko tab tak process nahi karega jab tak pehla process poora na ho jaaye. Agar server ko aur zyada requests milne lagein, toh yeh serial execution kam se kam optimal hoga. Agar server ko ek aisi request milti hai jise process karne mein lamba samay lagta hai, toh baad ke requests ko tab tak intezaar karna padega jab tak lamba request poora na ho jaaye, bhale hi naye requests jaldi process kiye ja sakte hon. Humein ise theek karna hoga, lekin pehle hum is samasya ko action mein dekhenge.

-----

### Ek Slow Request ko Simulate Karna

Hum dekhenge ki ek slow-processing request hamare server ke current implementation mein kiye gaye doosre requests ko kaise prabhavit kar sakti hai. Listing 21-10, `_/sleep_` ke request ko handle karne ka code implement karta hai, jismein ek simulated slow response hai jo server ko response dene se pehle paanch second ke liye sleep kar dega.

\<Listing number="21-10" file-name="src/main.rs" caption="Paanch second ke liye sleep karke ek slow request simulate karna"\>

```rust,no_run
// in handle_connection in src/main.rs
// --snip--
let (status_line, filename) = match &request_line[..] {
    "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
    "GET /sleep HTTP/1.1" => {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK", "hello.html")
    }
    _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
};
// --snip--
```

\</Listing\>

Ab jab hamare paas teen cases hain, humne `if` se `match` par switch kar liya hai.

Aap dekh sakte hain ki hamara server kitna primitive hai: asli libraries multiple requests ki pehchaan ko bahut kam verbose tareeke se handle karti hain\!

Server ko `cargo run` ka istemal karke shuru karein. Phir do browser windows kholein: ek *[http://127.0.0.1:7878](http://127.0.0.1:7878)* ke liye aur doosri *[http://127.0.0.1:7878/sleep](http://127.0.0.1:7878/sleep)* ke liye. Agar aap `_/_` URI ko kuch baar enter karte hain, jaisa ki pehle kiya tha, toh aap dekhenge ki yeh jaldi response deta hai. Lekin agar aap `_/sleep_` enter karte hain aur phir `_/_` load karte hain, toh aap dekhenge ki `_/_` `sleep` ke poore paanch second tak sleep karne ka intezaar karta hai, uske baad hi load hota hai.

Is samasya se bachne ke liye hum kai techniques istemal kar sakte hain; jise hum implement karenge woh hai ek **thread pool**.

-----

### Thread Pool ke Saath Throughput Badhana

Ek **thread pool** pehle se banaye gaye (spawned) threads ka ek group hota hai jo ek task ko handle karne ke liye intezaar karte rehte hain. Jab program ko ek naya task milta hai, toh woh pool mein se ek thread ko task assign karta hai, aur woh thread task ko process karta hai. Pool mein baaki threads kisi bhi doosre task ko handle karne ke liye uplabdh hote hain. Jab pehla thread apna task poora kar leta hai, toh use idle threads ke pool mein wapas bhej diya jaata hai. Ek thread pool aapko connections ko concurrently process karne deta hai, jisse aapke server ka throughput badhta hai.

Hum pool mein threads ki sankhya ko ek chhoti sankhya tak seemit rakhenge taaki hum DoS (Denial of Service) attacks se bach sakein. Agar hamara program har request ke liye ek naya thread banata, toh koi hamare server par 10 million requests karke hamare server ke saare resources ka istemal karke tabahi macha sakta tha.

Asimit threads spawn karne ke bajaye, hamare paas pool mein ek nishchit sankhya mein threads honge. Aane wale requests ko process karne ke liye pool mein bheja jaata hai. Pool aane wale requests ki ek queue banaye rakhega. Pool mein har thread is queue se ek request nikalega, use handle karega, aur phir queue se doosra request maangega. Is design ke saath, hum ek saath `N` requests tak process kar sakte hain, jahan `N` threads ki sankhya hai.

Thread pool implement karna shuru karne se pehle, chaliye baat karte hain ki pool ka istemal kaisa dikhna chahiye. Code design karte samay, client interface ko pehle likhna aapke design ko guide kar sakta hai.

#### Har Request ke liye ek Thread Spawn Karna

Pehle, chaliye dekhte hain ki agar hamara code har connection ke liye ek naya thread banata toh kaisa dikhta. Jaisa ki pehle bataya gaya hai, asimit sankhya mein threads spawn karne ki samasyaon ke kaaran yeh hamara antim plan nahi hai, lekin yeh ek kaam karne wala multithreaded server banane ke liye ek shuruaati bindu hai.

Listing 21-11 `main` mein kiye jaane wale badlavon ko dikhata hai taaki `for` loop ke andar har stream ko handle karne ke liye ek naya thread spawn kiya ja sake.

\<Listing number="21-11" file-name="src/main.rs" caption="Har stream ke liye ek naya thread spawn karna"\>

```rust,no_run
// in main in src/main.rs
// --snip--
for stream in listener.incoming() {
    let stream = stream.unwrap();

    thread::spawn(|| {
        handle_connection(stream);
    });
}
// --snip--
```

\</Listing\>

Jaisa ki aapne Chapter 16 mein seekha tha, `thread::spawn` ek naya thread banayega aur phir closure mein diye gaye code ko naye thread mein chalayega. Agar aap is code ko chalate hain aur apne browser mein `_/sleep_` load karte hain, aur phir do aur browser tabs mein `_/_` load karte hain, toh aap dekhenge ki `_/_` ke requests ko `_/sleep_` ke poora hone ka intezaar nahi karna padta. Lekin, jaisa ki humne bataya, yeh ant mein system ko overwhelm kar dega kyunki aap bina kisi seema ke naye threads bana rahe honge.

#### Ek સીમિત (Finite) Sankhya mein Threads Banana

Hum chahte hain ki hamara thread pool ek jaise, parichit tareeke se kaam kare taaki threads se thread pool mein switch karne ke liye hamare API ka istemal karne wale code mein bade badlavon ki zaroorat na pade. Listing 21-12 ek `ThreadPool` struct ke liye kalpanik interface dikhata hai jise hum `thread::spawn` ke bajaye istemal karna chahte hain.

\<Listing number="21-12" file-name="src/main.rs" caption="Hamara aadarsh `ThreadPool` interface"\>

```rust,ignore,does_not_compile
// in main in src/main.rs
let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
let pool = ThreadPool::new(4);

for stream in listener.incoming() {
    let stream = stream.unwrap();

    pool.execute(|| {
        handle_connection(stream);
    });
}
```

\</Listing\>

Hum `ThreadPool::new` ka istemal karke ek naya thread pool banate hain jismein threads ki sankhya configure ki ja sakti hai, is maamle mein chaar. Phir, `for` loop mein, `pool.execute` ka interface `thread::spawn` jaisa hi hai, yaani yeh ek closure leta hai jise pool ko har stream ke liye chalana chahiye. Humein `pool.execute` ko implement karna hoga taaki yeh closure le aur use pool mein ek thread ko chalane ke liye de. Yeh code abhi compile nahi hoga, lekin hum koshish karenge taaki compiler humein ise theek karne mein guide kar sake.

-----

#### Compiler-Driven Development ka Istemal karke `ThreadPool` Banana

Listing 21-12 ke badlavon ko *src/main.rs* mein karein, aur phir chaliye `cargo check` se milne wale compiler errors ka istemal karke apne development ko aage badhayein. Yahan pehla error hai jo humein milta hai:

```console
$ cargo check
   ...
error[E0433]: failed to resolve: use of undeclared type `ThreadPool`
  --> src/main.rs:9:16
   |
9  | let pool = ThreadPool::new(4);
   |            ^^^^^^^^^^ use of undeclared type `ThreadPool`
```

Badhiya\! Yeh error humein batata hai ki humein ek `ThreadPool` type ya module ki zaroorat hai, isliye hum ab ek banayenge. Hamara `ThreadPool` implementation hamare web server ke kaam se swatantra hoga. Isliye chaliye `hello` crate ko ek binary crate se ek library crate mein badalte hain taaki hamara `ThreadPool` implementation usmein rakha ja sake.

Ek *src/lib.rs* file banayein jismein nimnalikhit code ho, jo `ThreadPool` struct ki sabse saral definition hai jo hum abhi ke liye rakh sakte hain:

```rust,noplayground
pub struct ThreadPool;
```

Phir *main.rs* file ko edit karein taaki library crate se `ThreadPool` ko scope mein laaya ja sake, *src/main.rs* ke upar nimnalikhit code jodkar:

```rust,ignore
use hello::ThreadPool;
```

Yeh code abhi bhi kaam nahi karega, lekin chaliye ise phir se check karte hain taaki agla error mile jise humein aage theek karna hai: `ThreadPool` ke liye `new` naam ka ek associated function nahi hai. Chaliye `new` function ka sabse saral implementation karte hain:

```rust,noplayground
// In src/lib.rs
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }
}
```

Chaliye code ko phir se check karte hain. Ab error isliye aa raha hai kyunki hamare paas `ThreadPool` par `execute` method nahi hai.

Hum `ThreadPool` par `execute` method ko ek closure ko parameter ke roop mein lene ke liye define karenge. Hum jante hain ki hum ant mein standard library ke `thread::spawn` implementation jaisa hi kuch karenge, isliye hum dekh sakte hain ki `thread::spawn` ke signature mein uske parameter par kya bounds hain. Documentation humein dikhata hai ki `spawn` apne parameter par `FnOnce` ka istemal trait bound ke roop mein karta hai.

`F` type parameter mein `Send` trait bound aur `'static` lifetime bound bhi hai, jo hamari sthiti mein upyogi hain: humein closure ko ek thread se doosre mein transfer karne ke liye `Send` ki zaroorat hai aur `'static` ki zaroorat hai kyunki hum nahi jante ki thread ko execute hone mein kitna samay lagega. Chaliye `ThreadPool` par ek `execute` method banate hain jo in bounds ke saath ek generic parameter `F` lega:

```rust,noplayground
// In the impl ThreadPool block in src/lib.rs
pub fn execute<F>(&self, f: F)
where
    F: FnOnce() + Send + 'static,
{
}
```

Chaliye ise phir se check karte hain. Yeh compile hota hai\! Lekin dhyan dein ki agar aap `cargo run` karke browser mein request karte hain, toh aapko wahi errors dikhenge jo humne chapter ki shuruaat mein dekhe the. Hamari library `execute` ko pass kiye gaye closure ko abhi tak call nahi kar rahi hai\!

-----

#### Threads ko Store Karne ke liye Jagah Banana

Ab jab hamare paas pool mein store karne ke liye threads ki ek valid sankhya hai, hum un threads ko bana sakte hain aur unhe `ThreadPool` struct mein store kar sakte hain. Lekin hum ek thread ko "store" kaise karein? `spawn` function ek `JoinHandle<T>` return karta hai. Chaliye `JoinHandle` ka istemal karke dekhte hain. Hamare maamle mein, closures kuch bhi return nahi karenge, isliye `T`, unit type `()` hoga.

Listing 21-14 ka code compile ho jaayega lekin abhi tak koi thread nahi banata hai. Humne `ThreadPool` ki definition ko `$thread::JoinHandle<()>)$` instances ka ek vector hold karne ke liye badal diya hai, vector ko `size` ki capacity ke saath initialize kiya hai, aur threads banane ke liye ek `for` loop set up kiya hai.

\<Listing number="21-14" file-name="src/lib.rs" caption="`ThreadPool` ke liye threads ko hold karne ke liye ek vector banana"\>

```rust,ignore,not_desired_behavior
use std::thread;

pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut threads = Vec::with_capacity(size);

        for _ in 0..size {
            // create some threads and store them in the vector
        }

        ThreadPool { threads }
    }

    // --snip--
}
```

\</Listing\>

#### Worker Struct ka istemal

Hum chahte hain ki hamare banaye gaye `Worker` structs `ThreadPool` mein rakhi ek queue se execute karne ke liye code lein aur us code ko apne thread mein chalane ke liye bhej dein.

Chapter 16 mein seekhe gaye **channels** is use case ke liye perfect honge. Hum ek channel ko jobs ki queue ke roop mein istemal karenge, aur `execute` `ThreadPool` se `Worker` instances ko ek job bhejega, jo job ko apne thread mein bhejenge. Yahan plan hai:

1.  `ThreadPool` ek channel banayega aur sender ko apne paas rakhega.
2.  Har `Worker` receiver ko apne paas rakhega.
3.  Hum ek naya `Job` struct banayenge jo un closures ko hold karega jinhe hum channel ke neeche bhejna chahte hain.
4.  `execute` method us job ko bhejega jise woh sender ke madhyam se execute karna chahta hai.
5.  Apne thread mein, `Worker` apne receiver par loop karega aur kisi bhi job ke closures ko execute karega jo use milta hai.

Lekin, hum `receiver` ko kai `Worker` instances ko pass karne ki koshish kar rahe hain. Yeh kaam nahi karega, kyunki Rust ka channel implementation **multiple producer, single consumer** hai. Iska matlab hai ki hum channel ke consuming end ko clone nahi kar sakte.

Is samasya ko hal karne ke liye, humein `$Arc<Mutex<T>>$` ka istemal karna hoga. `Arc` type multiple `Worker` instances ko receiver ka owner banne dega, aur `Mutex` sunishchit karega ki ek samay mein sirf ek `Worker` receiver se ek job prapt kare. Listing 21-18 mein kiye jaane wale badlavon ko dikhata hai.

\<Listing number="21-18" file-name="src/lib.rs" caption="`Arc` aur `Mutex` ka istemal karke receiver ko `Worker` instances ke beech share karna"\>

```rust,noplayground
use std::sync::{mpsc, Arc, Mutex};
// --snip--

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }
    // --snip--
}

struct Worker {
    // --snip--
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--
    }
}
```

\</Listing\>

`ThreadPool::new` mein, hum receiver ko ek `Arc` aur ek `Mutex` mein daalte hain. Har naye `Worker` ke liye, hum `Arc` ko clone karte hain taaki reference count badh jaaye aur `Worker` instances receiver ki ownership share kar sakein.

#### `execute` Method ko Implement Karna

Chaliye ant mein `ThreadPool` par `execute` method ko implement karte hain. Hum `Job` ko ek struct se ek type alias mein bhi badal denge jo us closure ke type ko hold karne wale trait object ke liye hai jo `execute` receive karta hai.

\<Listing number="21-19" file-name="src/lib.rs" caption="Har closure ko hold karne ke liye ek `Box` ke liye `Job` type alias banana aur phir job ko channel ke neeche bhejna"\>

```rust,noplayground
// in src/lib.rs
type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    // --snip--
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
}
// --snip--
```

\</Listing\>

Lekin hum abhi tak poora nahi hue hain\! `Worker` mein, `thread::spawn` ko pass kiya gaya hamara closure abhi bhi sirf channel ke receiving end ko *reference* kar raha hai. Iske bajaye, humein closure ko hamesha ke liye loop karne ki zaroorat hai, channel ke receiving end se ek job maangte hue aur jab use ek job milta hai toh use chalate hue. Chaliye `Worker::new` mein Listing 21-20 mein dikhaya gaya badlav karte hain.

\<Listing number="21-20" file-name="src/lib.rs" caption="`Worker` instance ke thread mein jobs ko receive aur execute karna"\>

```rust,noplayground
// in Worker::new in src/lib.rs
fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
    let thread = thread::spawn(move || loop {
        let job = receiver.lock().unwrap().recv().unwrap();

        println!("Worker {id} got a job; executing.");

        job();
    });

    Worker { id, thread }
}
```

\</Listing\>

Yahan, hum pehle `receiver` par `lock` call karke mutex acquire karte hain, aur phir kisi bhi error par panic hone ke liye `unwrap` call karte hain. Agar humein mutex par lock mil jaata hai, toh hum channel se ek `Job` receive karne ke liye `recv` call karte hain.

`recv` ka call block hota hai, isliye agar abhi tak koi job nahi hai, toh current thread tab tak intezaar karega jab tak ek job uplabdh na ho jaaye. `$Mutex<T>$` sunishchit karta hai ki ek samay mein sirf ek `Worker` thread hi ek job request karne ki koshish kar raha hai.

Hamara thread pool ab ek kaam karne ki sthiti mein hai\! Ise `cargo run` karein aur kuch requests karein. Safalta\! Ab hamare paas ek thread pool hai jo connections ko asynchronously execute karta hai. Kabhi bhi chaar se zyada threads nahi banaye jaate, isliye agar server ko bahut saare requests milte hain toh hamara system overload nahi hoga. Agar hum `_/sleep_` par ek request karte hain, toh server doosre requests ko doosre thread se chalakar serve kar payega.