## Unsafe Rust

Ab tak humne jitna bhi code discuss kiya hai, us sab par Rust ki memory safety guarantees compile time par enforce ki gayi hain. Lekin, Rust ke andar ek doosri language chhipi hui hai jo in memory safety guarantees ko enforce nahi karti: ise *unsafe Rust* kehte hain aur yeh bilkul regular Rust ki tarah kaam karti hai, lekin humein extra superpowers deti hai.

Unsafe Rust isliye exist karti hai kyunki, by nature, static analysis conservative hota hai. Jab compiler yeh determine karne ki koshish karta hai ki code guarantees ko poora karta hai ya nahi, toh uske liye kuch valid programs ko reject karna behtar hai, bajaye iske ki woh kuch invalid programs ko accept karle. Bhale hi code *shayad* theek ho, lekin agar Rust compiler ke paas confident hone ke liye पर्याप्त information nahi hai, toh woh code ko reject kar dega. Aise cases mein, aap unsafe code ka istemal karke compiler ko bata sakte hain, "Trust me, I know what I’m doing." (Mujh par bharosa karo, mujhe pata hai main kya kar raha hoon). Lekin saavdhan rahen, aap unsafe Rust ko apne risk par istemal karte hain: agar aap unsafe code ko galat tareeke se istemal karte hain, toh memory unsafety ke kaaran problems ho sakti hain, jaise ki null pointer dereferencing.

Rust ka ek unsafe alter ego hone ka ek aur kaaran yeh hai ki underlying computer hardware inherently unsafe hota hai. Agar Rust aapko unsafe operations karne nahi deta, toh aap kuch khaas tasks nahi kar paate. Rust ko aapko low-level systems programming karne ki anumati deni hogi, jaise ki operating system ke saath seedhe interact karna ya apna khud ka operating system likhna. Low-level systems programming ke saath kaam karna is language ke goals mein se ek hai. Chaliye dekhte hain ki hum unsafe Rust ke saath kya kar sakte hain aur kaise kar sakte hain.

### Unsafe Superpowers

Unsafe Rust mein switch karne ke liye, `unsafe` keyword ka istemal karein aur phir ek naya block shuru karein jismein unsafe code hoga. Aap unsafe Rust mein paanch aise actions le sakte hain jo aap safe Rust mein nahi le sakte, jinko hum *unsafe superpowers* kehte hain. Un superpowers mein yeh shaamil hain:

1.  Ek raw pointer ko dereference karna
2.  Ek unsafe function ya method ko call karna
3.  Ek mutable static variable ko access ya modify karna
4.  Ek unsafe trait ko implement karna
5.  `union` ke fields ko access karna

Yeh samajhna zaroori hai ki `unsafe` borrow checker ko band nahi karta ya Rust ke kisi aur safety check ko disable nahi karta: agar aap unsafe code mein ek reference ka istemal karte hain, toh use abhi bhi check kiya jaayega. `unsafe` keyword aapko sirf in paanch features tak access deta hai jinki memory safety compiler dwara check nahi ki jaati. Aapko ek unsafe block ke andar bhi kuch had tak safety milegi.

Iske alawa, `unsafe` ka matlab yeh nahi hai ki block ke andar ka code zaroori taur par khatarnak hai ya usmein memory safety problems hongi: iska matlab yeh hai ki programmer ke taur par, aap yeh sunishchit karenge ki `unsafe` block ke andar ka code memory ko valid tareeke se access karega.

Insaan galatiyan karte hain aur galatiyan hongi, lekin in paanch unsafe operations ko `unsafe` se annotate kiye gaye blocks ke andar rakhne ki zaroorat se, aapko pata chal jaayega ki memory safety se related koi bhi error ek `unsafe` block ke andar hi honi chahiye. `unsafe` blocks ko chhota rakhein; jab aap memory bugs investigate karenge toh aapko iske liye shukriya mehsoos hoga.

Unsafe code ko jitna ho sake isolate karne ke liye, behtar hai ki aise code ko ek safe abstraction ke andar band karein aur ek safe API pradaan karein, jiske baare mein hum is chapter mein baad mein unsafe functions aur methods par charcha karte samay baat karenge. Standard library ke kuch hisse unsafe code ke upar safe abstractions ke roop mein implement kiye gaye hain jinka audit kiya gaya hai. Unsafe code ko ek safe abstraction mein wrap karne se `unsafe` ka istemal un sabhi jagahon par failne se rokta hai jahan aap ya aapke users `unsafe` code se implement ki gayi functionality ka istemal karna chahte hain, kyunki ek safe abstraction ka istemal karna safe hai.

Chaliye ek-ek karke paancho unsafe superpowers ko dekhte hain. Hum kuch aise abstractions ko bhi dekhenge jo unsafe code ke liye ek safe interface pradaan karte hain.

-----

### Ek Raw Pointer ko Dereference Karna

Chapter 4 mein, "Dangling References" mein, humne bataya tha ki compiler yeh sunishchit karta hai ki references hamesha valid hon. Unsafe Rust mein do naye types hain jinhe *raw pointers* kaha jaata hai jo references ke samaan hain. References ki tarah, raw pointers immutable ya mutable ho sakte hain aur inhe kramashah `$*const T$` aur `$*mut T$` ke roop mein likha jaata hai. Asterisk (`*`) dereference operator nahi hai; yeh type name ka hissa hai. Raw pointers ke context mein, *immutable* ka matlab hai ki pointer ko dereference karne ke baad seedhe assign nahi kiya ja sakta.

References aur smart pointers se alag, raw pointers:

  * Borrowing rules ko ignore kar sakte hain, jaise ki ek hi location par immutable aur mutable pointers dono ka hona ya multiple mutable pointers ka hona.
  * Yeh guaranteed nahi hai ki woh valid memory ko point karenge.
  * Null hone ki anumati hai.
  * Koi automatic cleanup implement nahi karte.

Rust dwara in guarantees ko enforce karwane se opt out karke, aap behtar performance ya kisi doosri language ya hardware ke saath interface karne ki kshamta ke badle mein guaranteed safety ko chhod sakte hain, jahan Rust ki guarantees लागू nahi hoti.

Listing 20-1 dikhata hai ki ek immutable aur ek mutable raw pointer kaise banayein.

\<Listing number="20-1" caption="Raw borrow operators ke saath raw pointers banana"\>

```rust
let mut num = 5;

let r1 = &raw const num as *const i32;
let r2 = &raw mut num as *mut i32;
```

\</Listing\>

Dhyan dein ki hum is code mein `unsafe` keyword shaamil nahi karte hain. Hum safe code mein raw pointers bana sakte hain; hum bas ek unsafe block ke bahar raw pointers ko dereference nahi kar sakte, jaisa ki aap thodi der mein dekhenge.

Humne raw borrow operators ka istemal karke raw pointers banaye hain: `&raw const num` ek `$*const i32$` immutable raw pointer banata hai, aur `&raw mut num` ek `$*mut i32$` mutable raw pointer banata hai. Kyunki humne inhe seedhe ek local variable se banaya hai, hum jaante hain ki yeh khaas raw pointers valid hain, lekin hum kisi bhi raw pointer ke baare mein yeh anumaan nahi laga sakte.

Ise dikhane ke liye, aage hum ek aisa raw pointer banayenge jiski validity ke baare mein hum itne sure nahi ho sakte, `as` keyword ka istemal karke ek value ko cast karenge bajaye raw borrow operator ka istemal karne ke. Listing 20-2 dikhata hai ki memory mein ek arbitrary location par ek raw pointer kaise banayein. Arbitrary memory ka istemal karne ki koshish karna undefined hai: us address par data ho sakta hai ya nahi, compiler code ko optimize kar sakta hai taaki koi memory access na ho, ya program segmentation fault ke saath terminate ho sakta hai. Aam taur par, is tarah ka code likhne ka koi accha kaaran nahi hota, khaas kar un maamlon mein jahan aap raw borrow operator ka istemal kar sakte hain, lekin yeh sambhav hai.

\<Listing number="20-2" caption="Ek arbitrary memory address par ek raw pointer banana"\>

```rust
let address = 0x012345usize;
let r = address as *const i32;
```

\</Listing\>

Yaad rakhein ki hum safe code mein raw pointers bana sakte hain, lekin hum raw pointers ko *dereference* karke pointed data ko padh nahi sakte. Listing 20-3 mein, hum dereference operator `*` ka istemal ek raw pointer par karte hain jiske liye ek `unsafe` block ki zaroorat hoti hai.

\<Listing number="20-3" caption="Ek `unsafe` block ke andar raw pointers ko dereference karna"\>

```rust
let mut num = 5;

let r1 = &raw const num as *const i32;
let r2 = &raw mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

\</Listing\>

Pointer banana koi nuksaan nahi pahunchata; jab hum us value ko access karne ki koshish karte hain jise woh point kar raha hai, tabhi hum ek invalid value se deal kar sakte hain.

Yeh bhi dhyan dein ki Listings 20-1 aur 20-3 mein, humne `$*const i32$` aur `$*mut i32$` raw pointers banaye jo dono ek hi memory location ko point kar rahe the, jahan `num` store hai. Agar hum iske bajaye `num` ke liye ek immutable aur ek mutable reference banane ki koshish karte, toh code compile nahi hota kyunki Rust ke ownership rules ek hi samay par immutable references ke saath ek mutable reference ki anumati nahi dete. Raw pointers ke saath, hum ek hi location par ek mutable pointer aur ek immutable pointer bana sakte hain aur mutable pointer ke zariye data badal sakte hain, jisse data race ho sakta hai. Saavdhan rahen\!

In sabhi khatron ke saath, aap raw pointers ka istemal kyun karenge? Ek bada use case C code ke saath interface karna hai, jaisa ki aap agle section mein dekhenge. Ek aur case tab hota hai jab aap aise safe abstractions bana rahe hote hain jinhe borrow checker nahi samajhta. Hum unsafe functions introduce karenge aur phir ek safe abstraction ka example dekhenge jo unsafe code ka istemal karta hai.

-----

### Ek Unsafe Function ya Method ko Call Karna

Doosra operation jo aap ek unsafe block mein kar sakte hain, woh hai unsafe functions ko call karna. Unsafe functions aur methods bilkul regular functions aur methods ki tarah dikhte hain, lekin unke definition ke baaki hisse se pehle ek extra `unsafe` hota hai. Is context mein `unsafe` keyword yeh batata hai ki function ki kuch aisi zarooratein hain jinhe humein is function ko call karte samay poora karna hoga, kyunki Rust yeh guarantee nahi de sakta ki humne in zarooraton ko poora kiya hai. Ek `unsafe` block ke andar ek unsafe function ko call karke, hum yeh keh rahe hain ki humne is function ki documentation padh li hai aur hum function ke contracts ko poora karne ki zimmedari lete hain.

Yahan ek unsafe function hai jiska naam `dangerous` hai jo apni body mein kuch nahi karta:

```rust
unsafe fn dangerous() {}
```

Humein `dangerous` function ko ek alag `unsafe` block ke andar call karna hoga. Agar hum `dangerous` ko `unsafe` block ke bina call karne ki koshish karte hain, toh humein ek error milega:

```console
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0133]: call to unsafe function is unsafe and requires unsafe function or block
 --> src/main.rs:4:5
  |
4 |     dangerous();
  |     ^^^^^^^^^^^ call to unsafe function
  |
  = note: consult the documentation on unsafe functions for more information

For more information about this error, try `rustc --explain E0133`.
error: could not compile `unsafe-example` due to previous error
```

`unsafe` block ke saath, hum Rust ko yeh bata rahe hain ki humne function ki documentation padh li hai, hum samajhte hain ki ise aakhir kaise istemal karna hai, aur humne verify kar liya hai ki hum function ke contract ko poora kar rahe hain.

`unsafe` function ke body mein unsafe operations perform karne ke liye, aapko abhi bhi ek `unsafe` block ka istemal karna hoga, bilkul ek regular function ki tarah, aur agar aap bhool jaate hain toh compiler aapko warn karega. Isse humein `unsafe` blocks ko jitna ho sake chhota rakhne mein madad milti hai, kyunki poore function body mein unsafe operations ki zaroorat nahi ho sakti.

#### Unsafe Code ke Upar ek Safe Abstraction Banana

Sirf isliye ki ek function mein unsafe code hai, iska matlab yeh nahi hai ki humein poore function ko unsafe mark karna hoga. Asal mein, unsafe code ko ek safe function mein wrap karna ek aam abstraction hai. Ek example ke taur par, chaliye standard library ke `split_at_mut` function ko study karte hain, jiske liye kuch unsafe code ki zaroorat hoti hai. Hum dekhenge ki hum ise kaise implement kar sakte hain. Yeh safe method mutable slices par define kiya gaya hai: yeh ek slice leta hai aur argument ke roop mein diye gaye index par slice ko split karke do bana deta hai. Listing 20-4 dikhata hai ki `split_at_mut` ka istemal kaise karein.

\<Listing number="20-4" caption="Safe `split_at_mut` function ka istemal"\>

```rust
let mut v = vec![1, 2, 3, 4, 5, 6];

let r = &mut v[..];

let (a, b) = r.split_at_mut(3);

assert_eq!(a, &mut [1, 2, 3]);
assert_eq!(b, &mut [4, 5, 6]);
```

\</Listing\>

Hum is function ko sirf safe Rust ka istemal karke implement nahi kar sakte. Ek koshish Listing 20-5 jaisi dikh sakti hai, jo compile nahi hogi. Saralta ke liye, hum `split_at_mut` ko ek function ke roop mein implement karenge, na ki ek method ke roop mein, aur sirf `i32` values ke slices ke liye, na ki ek generic type `T` ke liye.

\<Listing number="20-5" caption="Sirf safe Rust ka istemal karke `split_at_mut` ka ek prayaasit implementation"\>

```rust,ignore,does_not_compile
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();

    assert!(mid <= len);

    (&mut values[..mid], &mut values[mid..])
}
```

\</Listing\>

Yeh function pehle slice ki total length leta hai. Phir yeh assert karta hai ki parameter ke roop mein diya gaya index slice ke andar hai, yeh check karke ki kya woh length se chhota ya barabar hai. Is assertion ka matlab hai ki agar hum slice ko split karne ke liye length se bada index pass karte hain, toh function us index ka istemal karne ki koshish karne se pehle panic ho jaayega.

Phir hum ek tuple mein do mutable slices return karte hain: ek original slice ke shuruaat se `mid` index tak aur doosra `mid` se slice ke ant tak.

Jab hum Listing 20-5 mein code ko compile karne ki koshish karenge, toh humein ek error milega:

```console
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0499]: cannot borrow `*values` as mutable more than once at a time
 --> src/main.rs:6:38
  |
1 | fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  |                         - let's call the lifetime of this reference `'1`
...
6 |     (&mut values[..mid], &mut values[mid..])
  |     --------------------      ^^^^^^^^^^^^^ second mutable borrow occurs here
  |     |
  |     first mutable borrow occurs here
  |     returning this value requires that `*values` is borrowed for `'1`

For more information about this error, try `rustc --explain E0499`.
error: could not compile `unsafe-example` due to previous error
```

Rust ka borrow checker yeh nahi samajh sakta ki hum slice ke alag-alag hisson ko borrow kar rahe hain; woh sirf itna jaanta hai ki hum ek hi slice se do baar borrow kar rahe hain. Ek slice ke alag-alag hisson ko borrow karna aam taur par theek hai kyunki dono slices overlap nahi ho rahe hain, lekin Rust itna smart nahi hai ki yeh jaan sake. Jab hum jaante hain ki code theek hai, lekin Rust nahi jaanta, tab unsafe code ka sahara lene ka samay hai.

Listing 20-6 dikhata hai ki `split_at_mut` ke implementation ko kaam karne ke liye ek `unsafe` block, ek raw pointer, aur kuch unsafe functions ke calls ka istemal kaise karein.

\<Listing number="20-6" caption="`split_at_mut` function ke implementation mein unsafe code ka istemal"\>

```rust
use std::slice;

fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    let ptr = values.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

\</Listing\>

Chapter 4 ke "The Slice Type" se yaad karein ki ek slice kuch data ka pointer aur slice ki length hota hai. Hum `len` method ka istemal karke slice ki length prapt karte hain aur `as_mut_ptr` method ka istemal karke slice ke raw pointer tak pahunchte hain. Is case mein, kyunki hamare paas `i32` values ke liye ek mutable slice hai, `as_mut_ptr` ek `$*mut i32$` type ka raw pointer return karta hai, jise humne `ptr` variable mein store kiya hai.

Hum is assertion ko banaye rakhte hain ki `mid` index slice ke andar hai. Phir hum unsafe code par aate hain: `slice::from_raw_parts_mut` function ek raw pointer aur ek length leta hai, aur yeh ek slice banata hai. Hum is function ka istemal ek slice banane ke liye karte hain jo `ptr` se shuru hota hai aur `mid` items lamba hai. Phir hum `ptr` par `add` method ko `mid` ke saath as an argument call karte hain taaki ek raw pointer prapt kiya ja sake jo `mid` se shuru hota hai, aur hum us pointer aur `mid` ke baad bache hue items ki sankhya ko length ke roop mein istemal karke ek slice banate hain.

`slice::from_raw_parts_mut` function unsafe hai kyunki yeh ek raw pointer leta hai aur is par vishwas karna padta hai ki yeh pointer valid hai. Raw pointers par `add` method bhi unsafe hai kyunki ise vishwas karna padta hai ki offset location bhi ek valid pointer hai. Isliye, humein `slice::from_raw_parts_mut` aur `add` ke calls ke aas paas ek `unsafe` block lagana pada taaki hum unhe call kar sakein. Code ko dekh kar aur yeh assertion jodkar ki `mid` `len` se kam ya barabar hona chahiye, hum keh sakte hain ki `unsafe` block ke andar istemal kiye gaye sabhi raw pointers slice ke andar data ke liye valid pointers honge. Yeh `unsafe` ka ek sweekarya aur uchit istemal hai.

Dhyan dein ki humein parinaami `split_at_mut` function ko `unsafe` mark karne ki zaroorat nahi hai, aur hum is function ko safe Rust se call kar sakte hain. Humne unsafe code ke liye ek safe abstraction banaya hai jismein function ka ek implementation `unsafe` code ka istemal ek surakshit tareeke se karta hai, kyunki yeh sirf is function ke access mein data se valid pointers banata hai.

Iske vipreet, Listing 20-7 mein `slice::from_raw_parts_mut` ka istemal slice ka istemal karte samay crash ho sakta hai. Yeh code ek arbitrary memory location leta hai aur 10,000 items lamba ek slice banata hai.

\<Listing number="20-7" caption="Ek arbitrary memory location se ek slice banana"\>

```rust
use std::slice;

fn main() {
    let address = 0x012345usize;
    let r = address as *mut i32;

    let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
}
```

\</Listing\>

Hum is arbitrary location par memory ke maalik nahi hain, aur is baat ki koi guarantee nahi hai ki yeh code jo slice banata hai usmein valid `i32` values hain. `values` ko is tarah istemal karne ki koshish karna jaise ki yeh ek valid slice hai, undefined behavior ka parinaam hota hai.

#### External Code ko Call Karne ke liye `extern` Functions ka Istemal

Kabhi-kabhi aapke Rust code ko kisi doosri language mein likhe code ke saath interact karne ki zaroorat pad sakti hai. Iske liye, Rust mein `extern` keyword hai jo ek *Foreign Function Interface (FFI)* banane aur istemal karne mein suvidha pradaan karta hai. FFI ek tareeka hai jisse ek programming language functions ko define karti hai aur ek alag (videshi) programming language ko un functions ko call karne mein saksham banati hai.

Listing 20-8 dikhata hai ki C standard library se `abs` function ke saath ek integration kaise set up karein. `extern` blocks ke andar declare kiye gaye functions aam taur par Rust code se call karne ke liye unsafe hote hain, isliye `extern` blocks ko bhi `unsafe` mark karna chahiye. Kaaran yeh hai ki doosri languages Rust ke niyamon aur guarantees ko laagoo nahi karti hain, aur Rust unhe check nahi kar sakta, isliye suraksha sunishchit karne ki zimmedari programmer par aati hai.

\<Listing number="20-8" file-name="src/main.rs" caption="Kisi doosri language mein define kiye gaye `extern` function ko declare aur call karna"\>

```rust
unsafe extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

\</Listing\>

`unsafe extern "C"` block ke andar, hum doosri language se un external functions ke naam aur signatures list karte hain jinhe hum call karna chahte hain. `"C"` hissa yeh define karta hai ki external function kaun sa *application binary interface (ABI)* istemal karta hai: ABI define karta hai ki function ko assembly level par kaise call karna hai. `"C"` ABI sabse aam hai aur C programming language ke ABI ka paalan karta hai.

Har item jo `unsafe extern` block ke andar declare kiya jaata hai, woh implicitly unsafe hota hai. Lekin, kuch FFI functions call karne ke liye safe *hain*. Udaharan ke liye, C ke standard library ke `abs` function mein koi memory safety considerations nahi hain aur hum jaante hain ki ise kisi bhi `i32` ke saath call kiya ja sakta hai. Aise maamlon mein, hum `safe` keyword ka istemal karke yeh keh sakte hain ki yeh vishisht function call karne ke liye safe hai, bhale hi yeh `unsafe extern` block mein ho. Jab hum yeh badlaav karte hain, toh ise call karne ke liye ab `unsafe` block ki zaroorat nahi hoti, jaisa ki Listing 20-9 mein dikhaya gaya hai.

\<Listing number="20-9" file-name="src/main.rs" caption="Ek `unsafe extern` block ke andar ek function ko spasht roop se `safe` mark karna aur use surakshit roop se call karna"\>

```rust
extern "C" {
    #[safe]
    fn abs(input: i32) -> i32;
}

fn main() {
    println!("Absolute value of -3 according to C: {}", abs(-3));
}
```

\</Listing\>

Ek function ko `safe` mark karna use swabhavik roop se surakshit nahi banata\! Balki, yeh ek vaade ki tarah hai jo aap Rust se kar rahe hain ki yeh surakshit hai. Is vaade ko poora karna abhi bhi aapki zimmedari hai\!

#### Doosri Languages se Rust Functions ko Call Karna

Hum `extern` ka istemal ek aisa interface banane ke liye bhi kar sakte hain jo doosri languages ko Rust functions call karne ki anumati deta hai. Ek poora `extern` block banane ke bajaye, hum `extern` keyword jodte hain aur `fn` keyword se theek pehle istemal karne ke liye ABI specify karte hain. Humein ek `#[unsafe(no_mangle)]` annotation bhi jodna hoga taaki Rust compiler ko is function ke naam ko mangle na karne ke liye kaha ja sake. *Mangling* tab hota hai jab ek compiler hamare dwara diye gaye function ke naam ko ek alag naam mein badal deta hai jismein compilation process ke doosre hisson ke liye zyada jaankari hoti hai lekin yeh insaanon ke liye kam padhne yogya hota hai. Har programming language ka compiler naamon ko thoda alag tareeke se mangle karta hai, isliye ek Rust function ko doosri languages dwara naam diye jaane ke liye, humein Rust compiler ke name mangling ko disable karna hoga. Yeh unsafe hai kyunki bina built-in mangling ke libraries ke beech naam takraav ho sakte hain, isliye yeh hamari zimmedari hai ki hum yeh sunishchit karein ki hamare dwara chuna gaya naam bina mangling ke export karne ke liye surakshit hai.

Nimnlikhit example mein, hum `call_from_c` function ko C code se accessible banate hain, jab ise ek shared library mein compile kiya jaata hai aur C se link kiya jaata hai:

```rust
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

`extern` ka yeh istemal `unsafe` ki zaroorat sirf attribute mein rakhta hai, `extern` block par nahi.

-----

### Ek Mutable Static Variable ko Access ya Modify Karna

Is kitaab mein, humne abhi tak global variables ke baare mein baat nahi ki hai, jise Rust support karta hai lekin Rust ke ownership rules ke saath samasya ho sakti hai. Agar do threads ek hi mutable global variable ko access kar rahe hain, toh yeh ek data race ka kaaran ban sakta hai.

Rust mein, global variables ko *static* variables kaha jaata hai. Listing 20-10 ek static variable ka declaration aur istemal ka ek example dikhata hai jismein value ke roop mein ek string slice hai.

\<Listing number="20-10" file-name="src/main.rs" caption="Ek immutable static variable ko define aur istemal karna"\>

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```

\</Listing\>

Static variables constants ke samaan hain, jinke baare mein humne Chapter 3 mein "Constants" mein charcha ki thi. Static variables ke naam convention ke anusaar `SCREAMING_SNAKE_CASE` mein hote hain. Static variables sirf `'static` lifetime wale references ko store kar sakte hain, jiska matlab hai ki Rust compiler lifetime ka pata laga sakta hai aur humein ise spasht roop se annotate karne ki zaroorat nahi hai. Ek immutable static variable ko access karna surakshit hai.

Constants aur immutable static variables ke beech ek sookshm antar yeh hai ki ek static variable mein values ka memory mein ek fixed address hota hai. Value ka istemal hamesha ek hi data ko access karega. Doosri taraf, constants ko jab bhi istemal kiya jaata hai, unke data ko duplicate karne ki anumati hoti hai. Ek aur antar yeh hai ki static variables mutable ho sakte hain. Mutable static variables ko access karna aur modify karna *unsafe* hai. Listing 20-11 dikhata hai ki `COUNTER` naam ke ek mutable static variable ko kaise declare, access, aur modify karein.

\<Listing number="20-11" file-name="src/main.rs" caption="Ek mutable static variable se padhna ya usmein likhna unsafe hai."\>

```rust
static mut COUNTER: u32 = 0;

unsafe fn add_to_count(inc: u32) {
    // SAFETY: We must only call this function from a single thread.
    COUNTER += inc;
}

fn main() {
    unsafe {
        add_to_count(3);

        // SAFETY: We must only access the counter from a single thread.
        println!("COUNTER: {}", COUNTER);
    }
}
```

\</Listing\>

Regular variables ki tarah, hum `mut` keyword ka istemal karke mutability specify karte hain. Koi bhi code jo `COUNTER` se padhta ya likhta hai, woh ek `unsafe` block ke andar hona chahiye. Listing 20-11 ka code compile hota hai aur `COUNTER: 3` print karta hai jaisa ki hum ummeed karte hain kyunki yeh single-threaded hai. Multiple threads dwara `COUNTER` ko access karne se data races ho sakti hain, isliye yeh undefined behavior hai. Isliye, humein poore function ko `unsafe` mark karna hoga aur suraksha seema ko document karna hoga, taaki function ko call karne wala koi bhi jaanta ho ki woh kya kar sakte hain aur kya nahi kar sakte surakshit roop se.

Jab bhi hum ek unsafe function likhte hain, toh `SAFETY` se shuru hone wala ek comment likhna ek idiomatic tareeka hai aur yeh samjhana hai ki caller ko function ko surakshit roop se call karne ke liye kya karna hoga. Isi tarah, jab bhi hum ek unsafe operation karte hain, toh `SAFETY` se shuru hone wala ek comment likhna ek idiomatic tareeka hai yeh samjhane ke liye ki suraksha niyam kaise banaye rakhe jaate hain.

Iske atirikt, compiler default roop se ek mutable static variable ke references banane ki kisi bhi koshish ko ek compiler lint ke madhyam se inkaar kar dega. Aapko ya toh us lint ke suraksha se spasht roop se opt-out karna hoga `#[allow(static_mut_refs)]` annotation jodkar ya mutable static variable ko raw borrow operators mein se ek ke saath banaye gaye raw pointer ke madhyam se access karna hoga. Ismein woh maamle shaamil hain jahan reference adrishya roop se banaya jaata hai, jaise ki jab is code listing mein `println!` mein istemal kiya jaata hai. Static mutable variables ke references ko raw pointers ke madhyam se banane ki aavashyakta unke istemal ke liye suraksha aavashyaktaon ko aur adhik spasht banane mein madad karti hai.

Globally accessible mutable data ke saath, yeh sunishchit karna mushkil hai ki koi data race na ho, yahi kaaran hai ki Rust mutable static variables ko unsafe maanta hai. Jahan sambhav ho, concurrency techniques aur thread-safe smart pointers ka istemal karna behtar hai jinke baare mein humne Chapter 16 mein charcha ki thi, taaki compiler check kare ki alag-alag threads se data access surakshit tareeke se kiya jaata hai.

-----

### Ek Unsafe Trait ko Implement Karna

Hum `unsafe` ka istemal ek unsafe trait ko implement karne ke liye kar sakte hain. Ek trait unsafe tab hota hai jab uske kam se kam ek method mein koi aisa invariant ho jise compiler verify nahi kar sakta. Hum ek trait ko `unsafe` `trait` se pehle `unsafe` keyword jodkar declare karte hain aur trait ke implementation ko bhi `unsafe` mark karte hain, jaisa ki Listing 20-12 mein dikhaya gaya hai.

\<Listing number="20-12" caption="Ek unsafe trait ko define aur implement karna"\>

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

\</Listing\>

`unsafe impl` ka istemal karke, hum yeh vaada kar rahe hain ki hum un invariants ko banaye rakhenge jinhe compiler verify nahi kar sakta.

Ek example ke taur par, `Send` aur `Sync` marker traits ko yaad karein jinke baare mein humne Chapter 16 mein charcha ki thi: compiler in traits ko automatically implement karta hai agar hamare types poori tarah se doosre types se bane hain jo `Send` aur `Sync` implement karte hain. Agar hum ek aisa type implement karte hain jismein ek aisa type hai jo `Send` ya `Sync` implement nahi karta, jaise ki raw pointers, aur hum us type ko `Send` ya `Sync` ke roop mein mark karna chahte hain, toh humein `unsafe` ka istemal karna hoga. Rust yeh verify nahi kar sakta ki hamara type un guarantees ko poora karta hai ki use threads ke beech surakshit roop se bheja ja sakta hai ya multiple threads se access kiya ja sakta hai; isliye, humein un checks ko manually karna hoga aur `unsafe` ke saath iska sanket dena hoga.

-----

### Union ke Fields ko Access Karna

Antim action jo sirf `unsafe` ke saath kaam karta hai, woh hai ek union ke fields ko access karna. Ek *union* ek `struct` ke samaan hai, lekin ek vishisht instance mein ek samay mein sirf ek declared field ka istemal hota hai. Unions ka mukhya roop se C code mein unions ke saath interface karne ke liye istemal hota hai. Union fields ko access karna unsafe hai kyunki Rust yeh guarantee nahi de sakta ki union instance mein vartamaan mein store kiye ja rahe data ka type kya hai.

-----

### Unsafe Code ko Check Karne ke liye Miri ka Istemal

Unsafe code likhte samay, aap yeh check karna chah sakte hain ki aapne jo likha hai woh vastav mein surakshit aur sahi hai. Aisa karne ka sabse accha tareeka Miri ka istemal karna hai, jo undefined behavior ka pata lagane ke liye ek official Rust tool hai. Jabki borrow checker ek *static* tool hai jo compile time par kaam karta hai, Miri ek *dynamic* tool hai jo runtime par kaam karta hai. Yeh aapke code ko aapke program, ya uske test suite ko chala kar check karta hai, aur jab aap un niyamon ka ullanghan karte hain jinhe woh samajhta hai ki Rust ko kaise kaam karna chahiye, toh pata lagata hai.

Miri ka istemal karne ke liye Rust ka nightly build zaroori hai. Aap Rust ka nightly version aur Miri tool dono ko `rustup +nightly component add miri` type karke install kar sakte hain. Isse aapke project dwara istemal kiya jaane wala Rust ka version nahi badalta; yeh sirf tool ko aapke system mein jodta hai taaki aap jab chahein iska istemal kar sakein. Aap ek project par Miri ko `cargo +nightly miri run` ya `cargo +nightly miri test` type karke chala sakte hain.

Yeh kitna madadgaar ho sakta hai, iska ek example ke liye, sochiye ki jab hum ise Listing 20-7 ke khilaaf chalate hain toh kya hota hai.

```console
$ cargo +nightly miri run
error: Undefined Behavior: pointer to unallocated memory was dereferenced
   --> src/main.rs:6:51
    |
6   |     let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
    |                                                   ^^^^^^^^^^^^^^^^^^ pointer to unallocated memory was dereferenced
    |
    = help: this indicates a bug in the program: it performed an invalid operation, and caused Undefined Behavior
    = help: see https://doc.rust-lang.org/nightly/reference/behavior-considered-undefined.html for further information
    = note: BACKTRACE:
    = note: inside `main` at src/main.rs:6:51

error: aborting due to previous error
```

Miri sahi tareeke se humein chetavni deta hai ki hum ek integer ko pointer mein cast kar rahe hain, jo ek samasya ho sakti hai lekin Miri pata nahi laga sakta ki kya hai kyunki woh nahi jaanta ki pointer kaise utpann hua. Phir, Miri ek error return karta hai jahan Listing 20-7 mein undefined behavior hai kyunki hamare paas ek dangling pointer hai. Miri ka shukriya, ab hum jaante hain ki undefined behavior ka khatra hai, aur hum soch sakte hain ki code ko surakshit kaise banayein. Kuch maamlon mein, Miri galatiyon ko theek karne ke baare mein sifarish bhi kar sakta hai.

Miri har us cheez ko nahi pakadta jo aap unsafe code likhte samay galat kar sakte hain. Miri ek dynamic analysis tool hai, isliye yeh sirf us code ke saath samasyaon ko pakadta hai jo vastav mein chalta hai. Iska matlab hai ki aapko apne likhe hue unsafe code ke baare mein apna vishwas badhane ke liye achhi testing techniques ke saath iska istemal karna hoga. Miri aapke code ke aswasth hone ke har sambhav tareeke ko bhi cover nahi karta hai.

Doosre shabdon mein: Agar Miri koi samasya pakadta hai, toh aap jaante hain ki ek bug hai, lekin sirf isliye ki Miri koi bug nahi pakadta, iska matlab yeh nahi hai ki koi samasya nahi hai. Yeh bahut kuch pakad sakta hai, haalaanki. Is chapter mein unsafe code ke doosre examples par ise chala kar dekhein aur dekhein ki yeh kya kehta hai\!

-----

### Unsafe Code Kab Istemal Karein

Upar bataye gaye paanch superpowers mein se kisi ek ka istemal karne ke liye `unsafe` ka istemal karna galat ya bura nahi maana jaata, lekin `unsafe` code ko sahi karna zyada mushkil hai kyunki compiler memory safety banaye rakhne mein madad nahi kar sakta. Jab aapke paas `unsafe` code istemal karne ka koi kaaran ho, toh aap kar sakte hain, aur spasht `unsafe` annotation hone se samasya hone par problem ke source ko track karna aasan ho jaata hai. Jab bhi aap unsafe code likhte hain, toh aap Miri ka istemal karke apne likhe code ke Rust ke niyamon ka paalan karne ke baare mein adhik vishwas haasil kar sakte hain.

Unsafe Rust ke saath prabhavi dhang se kaam karne ke baare mein aur gehraai se jaanne ke liye, Rust ka official guide, [Rustonomicon][nomicon] padhein.

[nomicon]: https://www.google.com/search?q=%5Bhttps://doc.rust-lang.org/nomicon/%5D\(https://doc.rust-lang.org/nomicon/\)