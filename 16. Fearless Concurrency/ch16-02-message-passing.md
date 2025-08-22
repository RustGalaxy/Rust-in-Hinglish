## Threads ke Beech Data Transfer Karne ke Liye Message Passing ka Istemal

Surakshit concurrency sunishchit karne ka ek tezi se lokpriya ho raha approach hai **message passing**, jahan threads ya actors data waale messages ek doosre ko bhejkar communicate karte hain. Is idea ko Go language ke documentation se ek naare mein samjha ja sakta hai: "Memory share karke communicate mat karo; balki, communicate karke memory share karo."

Message-sending concurrency ko haasil karne ke liye Rust ka ek bada tool hai **channel**, ek programming concept jiska implementation Rust ki standard library pradan karti hai. Aap programming mein ek channel ki kalpana paani ke ek channel, jaise ki ek dhara ya nadi, ki tarah kar sakte hain. Agar aap ek stream mein rubber duck ya naav jaisi koi cheez daalte hain, to vah paani ke bahaav ke saath neeche ki or jaakar jalmarg ke ant tak pahunch jayegi.

Programming mein ek channel ke do hisse hote hain: ek transmitter aur ek receiver. **Transmitter** wala hissa upar ki or (upstream) ka sthan hai jahan aap nadi mein rubber ducks daalte hain, aur **receiver** wala hissa vah jagah hai jahan rubber duck neeche ki or (downstream) pahunchti hai. Aapke code ka ek hissa transmitter par methods ko us data ke saath call karta hai jise aap bhejna chahte hain, aur doosra hissa aane waale messages ke liye receiving end ko check karta hai. Ek channel ko **closed** tab kaha jaata hai jab ya to transmitter ya receiver hissa drop ho jaata hai.

Yahan, hum ek program banayenge jismein ek thread values generate karega aur unhe ek channel ke neeche bhejega, aur doosra thread values prapt karega aur unhe print karega.

Sabse pehle, Listing 16-6 mein, hum ek channel banayenge lekin uske saath kuch nahi karenge. Dhyan dein ki yah abhi compile nahi hoga kyunki Rust yah nahi bata sakta ki hum channel par kis prakaar ki values bhejna chahte hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
#     tx.send(()).unwrap();
}
```

\<span class="caption"\>Listing 16-6: Ek channel banana aur do hisson ko `tx` aur `rx` mein assign karna\</span\>

Hum `mpsc::channel` function ka upyog karke ek naya channel banate hain; `mpsc` ka arth hai **multiple producer, single consumer**. Sankṣhep mein, Rust ki standard library jis tarah se channels ko implement karti hai, uska matlab hai ki ek channel ke kai *sending* ends ho sakte hain jo values produce karte hain, lekin keval ek *receiving* end hota hai jo un values ko consume karta hai. Kalpana kijiye ki kai dharaayein ek saath ek badi nadi mein mil rahi hain: kisi bhi dhara mein neeche bheji gayi har cheez ant mein ek hi nadi mein pahunchegi.

`mpsc::channel` function ek tuple return karta hai, jiska pehla element sending end hai aur doosra element receiving end hai. `tx` aur `rx` sankṣhep roop paramparik roop se kai kshetron mein kramashah **transmitter** aur **receiver** ke liye upyog kiye jaate hain, isliye hum apne variables ko usi tarah naam dete hain.

Chaliye, transmitting end ko ek spawned thread mein move karte hain aur usse ek string bhejvate hain taaki spawned thread main thread ke saath communicate kar sake, jaisa ki Listing 16-7 mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
```

\<span class="caption"\>Listing 16-7: `tx` ko ek spawned thread mein move karna aur "hi" bhejna\</span\>

Hum `move` ka upyog kar rahe hain taaki `tx` ko closure mein move kiya ja sake taaki spawned thread `tx` ka owner ho. Spawned thread ko channel ke through messages bhej paane ke liye channel ke transmitting end ka owner hona zaroori hai.

Transmitting end mein ek `send` method hota hai jo us value ko leta hai jise hum bhejna chahte hain. `send` method ek `Result<T, E>` type return karta hai, isliye agar receiving end pehle hi drop ho chuka hai, to send operation ek error return karega. Is udaharan mein, hum error ke maamle mein panic hone ke liye `unwrap` call kar rahe hain.

Listing 16-8 mein, hum main thread mein channel ke receiving end se value prapt karenge.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

\<span class="caption"\>Listing 16-8: Main thread mein "hi" value prapt karna aur use print karna\</span\>

Channel ke receiving end ke do upyogi methods hain: `recv` aur `try_recv`. Hum `recv` ka upyog kar rahe hain, jo **receive** ka sankṣhep hai. Yah main thread ke execution ko **block** kar dega aur tab tak intezaar karega jab tak ki channel mein koi value na bheji jaaye. Ek baar value bhej di jaati hai, to `recv` use `Result<T, E>` mein return karega.

`try_recv` method block nahi karta, balki turant ek `Result<T, E>` return karega: `Ok` value jismein message hoga agar koi uplabdh hai, aur `Err` value agar is baar koi message nahi hai.

Jab hum Listing 16-8 mein code chalate hain, to hum main thread se print hui value dekhenge:

```text
Got: hi
```

-----

### Channels aur Ownership Transference

Ownership rules message sending mein ek mahatvapurna bhoomika nibhate hain kyunki ve aapko surakshit, concurrent code likhne mein madad karte hain. Chaliye ek prayog karte hain yah dikhane ke liye ki channels aur ownership samasyaon ko rokne ke liye ek saath kaise kaam karte hain: hum spawned thread mein ek `val` value ka upyog karne ki koshish karenge *jab humne use channel mein bhej diya ho*.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val);
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

\<span class="caption"\>Listing 16-9: `val` ko channel mein bhej dene ke baad use istemal karne ki koshish\</span\>

Yahan, hum `tx.send` ke madhyam se channel mein `val` bhej dene ke baad use print karne ki koshish karte hain. Iski anumati dena ek bura vichar hoga: ek baar value doosre thread ko bhej di gayi hai, to vah thread use dobara istemal karne ki koshish se pehle use modify ya drop kar sakta hai. Halaanki, Rust humein Listing 16-9 mein code compile karne ki koshish karne par ek error deta hai:

```text
error[E0382]: use of moved value: `val`
 --> src/main.rs:10:31
  |
9 |         tx.send(val).unwrap();
  |                 --- value moved here
10|         println!("val is {}", val);
  |                               ^^^ value used here after move
```

Hamari concurrency ki galti ne ek compile-time error paida kiya. `send` function apne parameter ka **ownership** le leta hai, aur jab value move ho jaati hai, to receiver uska ownership le leta hai. Yah humein galti se value ko bhejne ke baad dobara istemal karne se rokta hai; ownership system jaanchta hai ki sab kuch theek hai.

-----

### Multiple Values Bhejna aur Receiver ko Intezaar Karte Dekhna

Listing 16-10 mein kuch badlav kiye gaye hain jo saabit karenge ki code concurrently chal raha hai: spawned thread ab kai messages bhejega aur har message ke beech ek second ke liye rukega.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

\<span class="caption"\>Listing 16-10: Kai messages bhejna aur har ek ke beech rukna\</span\>

Main thread mein, hum ab `recv` function ko explicitly call nahi kar rahe hain: iske bajaye, hum `rx` ko ek **iterator** ke roop mein treat kar rahe hain. Har prapt value ke liye, hum use print kar rahe hain. Jab channel band ho jayega, to iteration samapt ho jayega.

Is code ko chalaane par, aapko har line ke beech 1 second ke viraam ke saath nimnalikhit output dikhna chahiye:

```text
Got: hi
Got: from
Got: the
Got: thread
```

Kyunki main thread ke `for` loop mein koi code nahi hai jo rukta hai, hum bata sakte hain ki main thread spawned thread se values prapt karne ka intezaar kar raha hai.

-----

### Transmitter ko Clone Karke Multiple Producers Banana

Pehle humne bataya tha ki `mpsc` ka matlab *multiple producer, single consumer* hai. Hum channel ke transmitting hisse ko clone karke multiple threads bana sakte hain jo sabhi ek hi receiver ko values bhejte hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# use std::thread;
# use std::sync::mpsc;
# use std::time::Duration;
#
# fn main() {
// --snip--

let (tx, rx) = mpsc::channel();

let tx1 = tx.clone();
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx {
    println!("Got: {}", received);
}

// --snip--
# }
```

\<span class="caption"\>Listing 16-11: Kai producers se kai messages bhejna\</span\>

Is baar, pehla spawned thread banane se pehle, hum channel ke sending end par `clone` call karte hain. Isse humein ek naya sending handle milega jise hum pehle spawned thread ko pass kar sakte hain. Hum channel ka original sending end doosre spawned thread ko pass karte hain. Isse humein do threads milte hain, har ek alag-alag messages channel ke receiving end ko bhej raha hai.

Jab aap code chalate hain, to aapka output kuch is tarah dikhna chahiye:

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Aapko values kisi aur kram mein dikh sakti hain; yah aapke system par nirbhar karta hai. Yahi cheez concurrency ko dilchasp aur saath hi kathin banati hai.

Ab jab humne dekh liya hai ki channels kaise kaam karte hain, chaliye concurrency ke ek alag tarike ko dekhen.