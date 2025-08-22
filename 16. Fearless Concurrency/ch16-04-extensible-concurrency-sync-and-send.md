## `Sync` aur `Send` Traits ke Saath Vistar-yogya (Extensible) Concurrency

Dilchasp baat yah hai ki Rust bhasha mein *bahut kam* concurrency features hain. Is adhyay mein ab tak humne jin lagbhag sabhi concurrency features ke baare mein baat ki hai, ve bhasha ka nahi, balki standard library ka hissa rahe hain. Concurrency ko handle karne ke liye aapke vikalp bhasha ya standard library tak seemit nahi hain; aap apne khud ke concurrency features likh sakte hain ya doosron dwara likhe gaye features ka upyog kar sakte hain.

Halaanki, do concurrency concepts bhasha mein nihit hain: `std::marker` traits `Sync` aur `Send`.

### `Send` ke Saath Threads ke Beech Ownership Transfer ki Anumati Dena

**`Send`** marker trait yah sanket deta hai ki `Send` implement karne wale type ka ownership threads ke beech transfer kiya ja sakta hai. Lagbhag har Rust type `Send` hai, lekin kuch apvaad hain, jinmein `Rc<T>` shaamil hai: yah `Send` nahi ho sakta kyunki agar aapne ek `Rc<T>` value ko clone kiya aur clone ka ownership doosre thread mein transfer karne ki koshish ki, to dono threads ek hi samay mein reference count ko update kar sakte hain. Is kaaran se, `Rc<T>` ko single-threaded sthitiyon mein upyog ke liye implement kiya gaya hai jahan aap thread-safe performance penalty nahi chukana chahte.

Isliye, Rust ka type system aur trait bounds yah sunishchit karte hain ki aap kabhi bhi galti se ek `Rc<T>` value ko threads ke beech asurakshit roop se nahi bhej sakte. Jab humne Listing 16-14 mein aisa karne ki koshish ki, to humein `the trait Send is not implemented for Rc<Mutex<i32>>` error mila. Jab humne `Arc<T>` mein switch kiya, jo `Send` hai, to code compile ho gaya.

Poori tarah se `Send` types se bana koi bhi type svachaalit roop se `Send` ke roop mein bhi chihnit ho jaata hai. Lagbhag sabhi primitive types `Send` hain, raw pointers ko chhodkar, jinke baare mein hum Chapter 19 mein charcha karenge.

### `Sync` ke Saath Multiple Threads se Access ki Anumati Dena

**`Sync`** marker trait yah sanket deta hai ki `Sync` implement karne wale type ko kai threads se reference karna surakshit hai. Doosre shabdon mein, koi bhi type `T` `Sync` hai agar `&T` (`T` ka ek reference) `Send` hai, jiska matlab hai ki reference ko surakshit roop se doosre thread mein bheja ja sakta hai. `Send` ki tarah, primitive types `Sync` hain, aur poori tarah se `Sync` types se bane types bhi `Sync` hote hain.

Smart pointer `Rc<T>` bhi unhi kaaranon se `Sync` nahi hai jin kaaranon se vah `Send` nahi hai. `RefCell<T>` type (jiske baare mein humne Chapter 15 mein baat ki thi) aur `Cell<T>` types ka parivaar `Sync` nahi hain. `RefCell<T>` dwara runtime par ki jaane wali borrow checking ki implementation thread-safe nahi hai. Smart pointer `Mutex<T>` `Sync` hai aur iska upyog kai threads ke saath access share karne ke liye kiya ja sakta hai jaisa ki aapne pichle section mein dekha.

### `Send` aur `Sync` ko Manually Implement Karna Unsafe Hai

Kyunki jo types `Send` aur `Sync` traits se bane hote hain, ve svachaalit roop se bhi `Send` aur `Sync` ho jaate hain, humein un traits ko manually implement karne ki zaroorat nahi hoti hai. Marker traits ke roop mein, unke paas implement karne ke liye koi methods bhi nahi hote hain. Ve keval concurrency se sambandhit invariants ko laagoo karne ke liye upyogi hain.

In traits ko manually implement karne mein unsafe Rust code likhna shaamil hai. Hum Chapter 19 mein unsafe Rust code ka upyog karne ke baare mein baat karenge; abhi ke liye, mahatvapurna jaankari yah hai ki `Send` aur `Sync` hisson se nahi bane naye concurrent types banane ke liye suraksha guarantees ko banaye rakhne ke liye savdhani se sochne ki zaroorat hoti hai. [The Rustonomicon][The Rustonomicon] mein in guarantees aur unhe kaise banaye rakhein, iske baare mein adhik jaankari hai.

-----

## Summary

Yah is pustak mein concurrency ka ant nahi hai: Chapter 20 mein project is adhyay ke concepts ka upyog ek adhik yatharthvaadi sthiti mein karega.

Jaisa ki pehle bataya gaya hai, kyunki Rust concurrency ko kaise handle karta hai, uska bahut kam hissa bhasha ka ang hai, kai concurrency samadhan crates ke roop mein implement kiye jaate hain. Ye standard library se tezi se viksit hote hain, isliye multithreaded sthitiyon mein upyog karne ke liye vartaman, अत्याधुनिक crates ke liye online khoj karna sunishchit karein.

Rust standard library message passing ke liye channels aur smart pointer types, jaise ki `Mutex<T>` aur `Arc<T>`, pradan karti hai jo concurrent contexts mein upyog karne ke liye surakshit hain. Type system aur borrow checker yah sunishchit karte hain ki in samadhanon ka upyog karne wala code data races ya invalid references ke saath samapt na ho. Ek baar jab aap apne code ko compile kar lete hain, to aap aashvast ho sakte hain ki yah kai threads par khushi se chalega, bina un kathin-se-track-karne-wale bugs ke jo anya bhashaon mein aam hain. Concurrent programming ab darne wala concept nahi hai: aage badhein aur apne programs ko concurrent banayein, nidar hokar\!

Aage, hum samasyaon ko model karne aur samadhanon ko sanrachit karne ke liye lokpriya (idiomatic) tarikon ke baare mein baat karenge jab aapke Rust programs bade ho jaate hain. Iske alawa, hum charcha karenge ki Rust ke idioms un idioms se kaise sambandhit hain jinse aap object-oriented programming se parichit ho sakte hain.

[the rustonomicon]: https://www.google.com/search?q=%5Bhttps://doc.rust-lang.org/stable/nomicon/%5D\(https://doc.rust-lang.org/stable/nomicon/\)