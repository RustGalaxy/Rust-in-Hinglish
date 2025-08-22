## Graceful Shutdown aur Cleanup

Listing 21-20 ka code, jaisa ki humne socha tha, ek thread pool ka istemal karke requests ko asynchronously respond kar raha hai. Humein `workers`, `id`, aur `thread` fields ke baare mein kuch warnings milti hain jinhe hum seedhe taur par istemal nahi kar rahe hain, jo humein yaad dilata hai ki hum kuch bhi cleanup nahi kar rahe hain. Jab hum kam elegant \<kbd\>ctrl\</kbd\>-\<kbd\>C\</kbd\> method ka istemal karke main thread ko rokte hain, toh baaki saare threads bhi turant ruk jaate hain, bhale hi woh kisi request ko serve karne ke beech mein hon.

Isliye, aage hum `Drop` trait implement karenge jo pool mein har thread par `join` call karega taaki woh band hone se pehle apne kaam par lage requests ko poora kar sakein. Phir hum ek tareeka implement karenge jisse threads ko bataya ja sake ki unhe naye requests accept karna band kar dena chahiye aur shut down ho jaana chahiye. Is code ko action mein dekhne ke liye, hum apne server ko modify karenge taaki woh apne thread pool ko gracefully shut down karne se pehle sirf do requests accept kare.

-----

### `ThreadPool` par `Drop` Trait ko Implement Karna

Chaliye apne thread pool par `Drop` implement karke shuruaat karte hain. Jab pool drop hota hai, toh hamare sabhi threads ko join ho jaana chahiye taaki woh apna kaam poora kar sakein. Listing 21-22 `Drop` implementation ka pehla prayaas dikhata hai; yeh code abhi theek se kaam nahi karega.

\<Listing number="21-22" file-name="src/lib.rs" caption="Thread pool ke scope se bahar jaane par har thread ko join karna"\>

```rust,ignore,does_not_compile
// in impl ThreadPool in src/lib.rs
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}
```

\</Listing\>

Pehle hum thread pool ke har `worker` par loop karte hain. Har `worker` ke liye, hum ek message print karte hain ki yeh khaas `Worker` instance shut down ho raha hai, aur phir hum us `Worker` instance ke thread par `join` call karte hain.

Yahan woh error hai jo humein is code ko compile karne par milta hai:

```console
$ cargo check
   ...
error[E0507]: cannot move out of `worker.thread` which is behind a mutable reference
  --> src/lib.rs:41:13
   |
41 |             worker.thread.join().unwrap();
   |             ^^^^^^^^^^^^^ move occurs because `worker.thread` has type `std::thread::JoinHandle<()>`, which does not implement the `Copy` trait
```

Error batata hai ki hum `join` call nahi kar sakte kyunki hamare paas har `worker` ka sirf ek mutable borrow hai aur `join` apne argument ki ownership le leta hai. Is samasya ko hal karne ke liye, humein thread ko `Worker` instance se bahar move karna hoga taaki `join` thread ko consume kar sake. Agar `Worker` `$Option<thread::JoinHandle<()>>$` hold karta, toh hum `Option` par `take` method call karke value ko `Some` variant se bahar move kar sakte the aur uski jagah `None` variant chhod sakte the.

Lekin, is maamle mein, ek behtar vikalp maujood hai: `Vec::drain` method. Yeh vector se sabhi values ko hata dega aur unka ek iterator return karega, jisse hum un par loop chala sakte hain.

Isliye humein `ThreadPool` `drop` implementation ko is tarah update karna hoga:

```rust,ignore
// In the impl Drop for ThreadPool block in src/lib.rs
for worker in self.workers.drain(..) {
    println!("Shutting down worker {}", worker.id);
    worker.thread.join().unwrap();
}
```

Isse compiler error aana band ho jaata hai aur hamare code mein kisi aur badlav ki zaroorat nahi padti.

-----

### Threads ko Jobs ke liye Sunna Band Karne ka Signal Dena

Hamara code ab bina kisi warning ke compile hota hai. Lekin, buri khabar yeh hai ki yeh code abhi tak waisa kaam nahi karta jaisa hum chahte hain. Asli baat `Worker` instances ke threads dwara chalaye ja rahe closures ke logic mein hai: abhi hum `join` call karte hain, lekin isse threads shut down nahi honge, kyunki woh jobs dhoondhne ke liye hamesha `loop` karte rehte hain. Agar hum apne `ThreadPool` ko apne current `drop` implementation ke saath drop karne ki koshish karte hain, toh main thread hamesha ke liye block ho jaayega, pehle thread ke poora hone ka intezaar karte hue.

Is samasya ko theek karne ke liye, humein `ThreadPool` `drop` implementation mein ek badlav aur phir `Worker` loop mein ek badlav karna hoga.

Pehle hum `ThreadPool` `drop` implementation ko threads ke finish hone ka intezaar karne se pehle `sender` ko spasht roop se drop karne ke liye badal denge.

```rust,ignore
// in impl Drop for ThreadPool
drop(self.sender.take());
```

`sender` ko drop karne se channel band ho jaata hai, jo yeh batata hai ki ab aur messages nahi bheje jaayenge. Jab aisa hota hai, toh `Worker` instances ke infinite loop mein kiye gaye `recv` ke saare calls ek error return karenge. Listing 21-24 mein, hum `Worker` loop ko us maamle mein gracefully loop se bahar nikalne ke liye badalte hain, jiska matlab hai ki jab `ThreadPool` `drop` implementation un par `join` call karega toh threads finish ho jaayenge.

\<Listing number="21-24" file-name="src/lib.rs" caption="`recv` ke error return karne par loop se spasht roop se bahar nikalna"\>

```rust,noplayground
// in Worker::new
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();

            match message {
                Ok(job) => {
                    println!("Worker {id} got a job; executing.");
                    job();
                }
                Err(_) => {
                    println!("Worker {id} disconnected; shutting down.");
                    break;
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

\</Listing\>

Is code ko action mein dekhne ke liye, chaliye `main` ko modify karte hain taaki server ko gracefully shut down karne se pehle sirf do requests accept kiye ja sakein, jaisa ki Listing 21-25 mein dikhaya gaya hai.

\<Listing number="21-25" file-name="src/main.rs" caption="Do requests serve karne ke baad loop se bahar nikal kar server ko shut down karna"\>

```rust,ignore
// in main in src/main.rs
for stream in listener.incoming().take(2) {
    // --snip--
}

println!("Shutting down.");
```

\</Listing\>

Aap ek asli web server ko sirf do requests serve karne ke baad shut down nahi karna chahenge. Yeh code sirf yeh dikhane ke liye hai ki graceful shutdown aur cleanup kaam kar raha hai.

`take` method `Iterator` trait mein define kiya gaya hai aur iteration ko pehle do items tak seemit rakhta hai. `ThreadPool` `main` ke ant mein scope se bahar chala jaayega, aur `drop` implementation chalega.

Server ko `cargo run` ke saath shuru karein, aur teen requests karein. Teesra request error dena chahiye, aur aapke terminal mein aapko is tarah ka output dikhna chahiye:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Badhai ho\! Humne ab apna project poora kar liya hai; hamare paas ek basic web server hai jo asynchronously respond karne ke liye ek thread pool ka istemal karta hai. Hum server ka graceful shutdown kar sakte hain, jo pool ke sabhi threads ko cleanup karta hai.

Yahan reference ke liye poora code hai:

\<Listing file-name="src/main.rs"\>

```rust,ignore
// Full final code for src/main.rs
use hello::ThreadPool;
use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
    thread,
    time::Duration,
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Shutting down.");
}

// ... rest of the code for handle_connection ...
```

\</Listing\>

\<Listing file-name="src/lib.rs"\>

```rust,noplayground
// Full final code for src/lib.rs
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};

// ... ThreadPool, Worker, Job definitions and implementations ...
```

\</Listing\>

Hum yahan aur bhi kuch kar sakte hain\! Agar aap is project ko aur behtar banana chahte hain, toh yahan kuch ideas hain:

  * `ThreadPool` aur uske public methods ke liye aur documentation jodein.
  * Library ki functionality ke liye tests jodein.
  * `unwrap` calls ko aur robust error handling se badlein.
  * Web requests serve karne ke alawa kisi aur kaam ke liye `ThreadPool` ka istemal karein.

## Summary

Bahut badhiya\! Aap is kitaab ke ant tak pahunch gaye hain\! Hum Rust ke is tour mein hamare saath shaamil hone ke liye aapka dhanyavaad karna chahte hain. Aap ab apne khud ke Rust projects implement karne aur doosre logon ke projects mein madad karne ke liye taiyaar hain. Yaad rakhein ki anya Rustaceans ka ek swagat karne wala samuday hai jo aapki Rust yatra mein aane wali kisi bhi chunauti mein aapki madad karna chahega.