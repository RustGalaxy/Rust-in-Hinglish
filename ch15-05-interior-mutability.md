## `RefCell<T>` aur Interior Mutability Pattern

*Interior mutability* Rust mein ek design pattern hai jo aapko data ko mutate (change) karne ki anumati deta hai, bhale hi us data ke immutable references maujood hon; aam taur par, borrowing rules is kaam ko mana karte hain. Data ko mutate karne ke liye, yeh pattern ek data structure ke andar `unsafe` code ka istemal karta hai taaki Rust ke mutation aur borrowing ko control karne waale aam niyamon ko thoda badal sake. Humne abhi tak unsafe code cover nahi kiya hai; hum ise Chapter 19 mein karenge. Hum un types ka istemal kar sakte hain jo interior mutability pattern ka upyog karte hain jab hum yeh sunishchit kar sakte hain ki borrowing rules runtime par follow kiye jayenge, bhale hi compiler iski guarantee na de sake. Ismein shaamil `unsafe` code ko phir ek safe API mein wrap kar diya jaata hai, aur baahari type abhi bhi immutable rehta hai.

Aaiye is concept ko `RefCell<T>` type ko dekh kar samjhein jo interior mutability pattern ko follow karta hai.

-----

## `RefCell<T>` ke Saath Runtime par Borrowing Rules ko Enforce Karna

`Rc<T>` ke vipreet, `RefCell<T>` type apne andar rakhe data par **single ownership** darshaata hai. Toh, `RefCell<T>` ko `Box<T>` jaise type se kya alag banata hai? Chapter 4 mein seekhe gaye borrowing rules ko yaad karein:

  * Kisi bhi samay, aapke paas ya toh ek mutable reference ho sakta hai ya kitne bhi immutable references ho sakte hain (dono ek saath nahi).
  * References hamesha valid hone chahiye.

References aur `Box<T>` ke saath, borrowing rules ke invariants **compile time par** enforce kiye jaate hain. `RefCell<T>` ke saath, ye invariants **runtime par** enforce kiye jaate hain. References ke saath, agar aap in niyamon ko todte hain, toh aapko ek compiler error milega. `RefCell<T>` ke saath, agar aap in niyamon ko todte hain, toh aapka program **panic** hokar exit ho jayega.

Compile time par borrowing rules ko check karne ke fayde yeh hain ki errors development process mein jaldi pakde jaate hain, aur runtime performance par koi asar nahi padta kyunki saara analysis pehle hi poora ho jaata hai. Inhi kaaranon se, zyadatar cases mein compile time par borrowing rules ko check karna sabse achha vikalp hai, aur isliye yeh Rust ka default hai.

Iske bajaye runtime par borrowing rules ko check karne ka fayda yeh hai ki kuch memory-safe scenarios tab anumati prapt karte hain, jabki weh compile-time checks dwara anumati nahi prapt karte. Static analysis, jaise Rust compiler, swabhav se conservative (satark) hota hai. Code ke kuch guno ko code ka vishleshan karke pata lagana asambhav hai: iska sabse prasiddh udaharan Halting Problem hai, jo is kitaab ke daayre se bahar hai lekin research karne ke liye ek dilchasp vishay hai.

Kyunki kuch analysis asambhav hai, agar Rust compiler yeh sunishchit nahi kar paata ki code ownership rules ka paalan karta hai, toh woh ek sahi program ko reject kar sakta hai; is tarah, woh conservative hai. Agar Rust ek galat program ko accept kar leta, toh users Rust dwara di gayi guarantees par bharosa nahi kar paate. Lekin, agar Rust ek sahi program ko reject karta hai, toh programmer ko asuvidha hogi, lekin kuch bhi anarth nahi ho sakta. `RefCell<T>` type tab upyogi hota hai jab aapko yakeen hai ki aapka code borrowing rules ka paalan karta hai lekin compiler ise samajhne aur guarantee dene mein asamarth hai.

`Rc<T>` ki tarah, `RefCell<T>` sirf **single-threaded scenarios** mein istemal ke liye hai aur agar aap ise multithreaded context mein istemal karne ki koshish karenge toh aapko compile-time error dega. Hum Chapter 16 mein baat karenge ki `RefCell<T>` ki functionality ko multithreaded program mein kaise prapt kiya jaaye.

Yahan `Box<T>`, `Rc<T>`, ya `RefCell<T>` chunne ke kaaranon ka ek recap hai:

  * `Rc<T>` ek hi data ke **multiple owners** ko saksham banata hai; `Box<T>` aur `RefCell<T>` ke **single owners** hote hain.
  * `Box<T>` compile time par check kiye gaye immutable ya mutable borrows ki anumati deta hai; `Rc<T>` sirf compile time par check kiye gaye immutable borrows ki anumati deta hai; `RefCell<T>` runtime par check kiye gaye immutable ya mutable borrows ki anumati deta hai.
  * Kyunki `RefCell<T>` runtime par check kiye gaye mutable borrows ki anumati deta hai, aap `RefCell<T>` ke andar ki value ko **mutate kar sakte hain, bhale hi `RefCell<T>` khud immutable ho**.

Ek immutable value ke andar ki value ko mutate karna hi **interior mutability** pattern hai. Aaiye ek aisi sthiti dekhen jismein interior mutability upyogi hai aur jaanchein ki yeh kaise sambhav hai.

-----

## Interior Mutability: Ek Immutable Value ke liye Mutable Borrow

Borrowing rules ka ek parinaam yeh hai ki jab aapke paas ek immutable value hoti hai, toh aap use mutably borrow nahi kar sakte. Udaharan ke liye, yeh code compile nahi hoga:

```rust,ignore
fn main() {
    let x = 5;
    let y = &mut x;
}
```

Agar aap is code ko compile karne ki koshish karte, toh aapko nimnlikhit error milta:

```text
error[E0596]: cannot borrow immutable local variable `x` as mutable
 --> src/main.rs:3:18
  |
2 |     let x = 5;
  |         - consider changing this to `mut x`
3 |     let y = &mut x;
  |                  ^ cannot borrow mutably
```

Halaanki, aisi sthitiyan hoti hain jinmein ek value ke liye apne methods mein khud ko mutate karna upyogi hoga, lekin doosre code ke liye woh immutable dikhe. Value ke methods ke bahar ka code us value ko mutate nahi kar payega. `RefCell<T>` ka istemal interior mutability ki kshamata prapt karne ka ek tareeka hai. Lekin `RefCell<T>` borrowing rules ko poori tarah se bypass nahi karta: compiler mein borrow checker is interior mutability ki anumati deta hai, aur borrowing rules runtime par check kiye jaate hain. Agar aap niyamon ka ullanghan karte hain, toh aapko compiler error ke bajaye ek `panic!` milega.

Aaiye ek practical example ke madhyam se kaam karte hain jahan hum `RefCell<T>` ka istemal ek immutable value ko mutate karne ke liye kar sakte hain aur dekhen ki yeh kyun upyogi hai.

### Interior Mutability ka ek Use Case: Mock Objects

Ek *test double* ek general programming concept hai us type ke liye jo testing ke dauraan doosre type ki jagah istemal hota hai. *Mock objects* ek vishesh prakaar ke test doubles hote hain jo yeh record karte hain ki test ke dauraan kya hota hai taaki aap yeh assert kar sakein ki sahi actions hue the.

Rust mein objects us tarah se nahi hote jaise doosri languages mein hote hain, aur Rust mein standard library mein mock object functionality built-in nahi hai jaisa ki kuch doosri languages mein hota hai. Lekin, aap nishchit roop se ek struct bana sakte hain jo mock object ke samaan uddeshyon ko poora karega.

Yahan woh scenario hai jise hum test karenge: hum ek library banayenge jo ek value ko ek maximum value ke khilaaf track karti hai aur current value maximum value ke kitne kareeb hai, uske aadhar par messages bhejti hai. Is library ka istemal, udaharan ke liye, ek user ke API calls ki quota ko track karne ke liye kiya ja sakta hai.

Hamari library sirf yeh functionality pradan karegi ki ek value maximum ke kitne kareeb hai aur kis samay kya messages hone chahiye. Hamari library ka istemal karne waale applications se yeh ummeed ki jayegi ki ve messages bhejne ka mechanism pradan karenge: application message ko application mein daal sakta hai, ek email bhej sakta hai, ek text message bhej sakta hai, ya kuch aur. Library ko us detail ko jaanne ki zaroorat nahi hai. Use bas kuch aisa chahiye jo hamare dwara pradan kiye gaye `Messenger` naamak trait ko implement kare. Listing 15-20 library code dikhata hai:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: 'a + Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
    where T: Messenger {
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 0.75 && percentage_of_max < 0.9 {
            self.messenger.send("Warning: You've used up over 75% of your quota!");
        } else if percentage_of_max >= 0.9 && percentage_of_max < 1.0 {
            self.messenger.send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        }
    }
}
```

\<span class="caption"\>Listing 15-20: Ek library jo track karti hai ki ek value maximum value ke kitne kareeb hai aur jab value kuch levels par hoti hai toh chetavani deti hai\</span\>

Is code ka ek mahatvapurna hissa yeh hai ki `Messenger` trait mein `send` naamak ek method hai jo `self` ka ek immutable reference aur message ka text leta hai. Yahi interface hamare mock object ko chahiye. Doosra mahatvapurna hissa yeh hai ki hum `LimitTracker` par `set_value` method ke behavior ko test karna chahte hain. Hum `value` parameter ke liye jo pass karte hain, use badal sakte hain, lekin `set_value` humein assertions karne ke liye kuch bhi return nahi karta. Hum yeh kehna chahte hain ki agar hum `Messenger` trait ko implement karne wali kisi cheez aur `max` ke liye ek vishesh value ke saath ek `LimitTracker` banate hain, jab hum `value` ke liye alag-alag number pass karte hain, toh messenger ko uchit messages bhejne ke liye kaha jaata hai.

Humein ek mock object ki zaroorat hai jo `send` call karne par email ya text message bhejne ke bajaye, sirf un messages ka track rakhe jo use bhejne ke liye kaha gaya hai. Listing 15-21 aesa karne ke liye ek mock object implement karne ki koshish dikhata hai, lekin borrow checker iski anumati nahi dega:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: vec![] }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

\<span class="caption"\>Listing 15-21: `MockMessenger` implement karne ki ek koshish jo borrow checker dwara anumati nahi hai\</span\>

Yeh test code ek `MockMessenger` struct define karta hai jismein ek `sent_messages` field hai jismein `Vec<String>` values hain taaki un messages ka track rakha ja sake jo use bhejne ke liye kaha gaya hai. Hum `send` method ki definition mein, `self` ka ek immutable reference lete hain. Hum `MockMessenger` list `sent_messages` mein message ko store karte hain.

Test mein, hum `LimitTracker` ko `max` value ke 75 percent se zyada `value` set karne par kya hota hai, yeh test kar rahe hain. Hum `LimitTracker` par `set_value` method ko 80 ki value ke saath call karte hain. Phir hum assert karte hain ki `MockMessenger` dwara track kiye ja rahe messages ki list mein ab ek message hona chahiye.

Lekin, is test ke saath ek samasya hai, jaisa ki yahan dikhaya gaya hai:

```text
error[E0596]: cannot borrow immutable field `self.sent_messages` as mutable
  --> src/lib.rs:52:13
   |
51 |         fn send(&self, message: &str) {
   |                   ----- use `&mut self` here to make mutable
52 |             self.sent_messages.push(String::from(message));
   |             ^^^^^^^^^^^^^^^^^^ cannot mutably borrow immutable field
```

Hum `MockMessenger` ko messages ka track rakhne ke liye modify nahi kar sakte, kyunki `send` method `self` ka ek immutable reference leta hai. Hum error text se `&mut self` ka istemal karne ka sujhav bhi nahi le sakte, kyunki tab `send` ka signature `Messenger` trait definition mein signature se match nahi karega.

Yeh ek aisi sthiti hai jismein interior mutability madad kar sakti hai\! Hum `sent_messages` ko ek `RefCell<T>` ke andar store karenge, aur tab `send` message `sent_messages` ko modify kar payega taaki humare dekhe gaye messages ko store kar sake. Listing 15-22 dikhata hai ki yeh kaisa dikhta hai:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: RefCell::new(vec![]) }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--
#         let mock_messenger = MockMessenger::new();
#         let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);
#         limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

\<span class="caption"\>Listing 15-22: `RefCell<T>` ka istemal karke ek andar ki value ko mutate karna jabki bahari value immutable maani jaati hai\</span\>

`sent_messages` field ab `Vec<String>` ke bajaye `RefCell<Vec<String>>` type ka hai. `new` function mein, hum empty vector ke around ek naya `RefCell<Vec<String>>` instance banate hain.

`send` method ke implementation ke liye, pehla parameter abhi bhi `self` ka ek immutable borrow hai. Hum `self.sent_messages` mein `RefCell<Vec<String>>` par `borrow_mut` call karte hain taaki `RefCell` ke andar ki value, jo ki vector hai, ka ek mutable reference prapt kar sakein. Phir hum vector ke mutable reference par `push` call kar sakte hain.

Aakhri badlav jo humein karna hai, woh assertion mein hai: andar ke vector mein kitne items hain, yeh dekhne ke liye, hum `RefCell<Vec<String>>` par `borrow` call karte hain taaki vector ka ek immutable reference prapt kar sakein.

Ab jab aapne `RefCell<T>` ka istemal karna dekh liya hai, aaiye gehraai se dekhen ki yeh kaise kaam karta hai\!

### `RefCell<T>` ke Saath Runtime par Borrows ka Track Rakhna

Jab immutable aur mutable references banate hain, hum kramashah `&` aur `&mut` syntax ka istemal karte hain. `RefCell<T>` ke saath, hum `borrow` aur `borrow_mut` methods ka istemal karte hain. `borrow` method `Ref<T>` smart pointer type return karta hai, aur `borrow_mut` `RefMut<T>` smart pointer type return karta hai. Dono types `Deref` implement karte hain, isliye hum unhein regular references ki tarah treat kar sakte hain.

`RefCell<T>` is baat ka track rakhta hai ki kitne `Ref<T>` aur `RefMut<T>` smart pointers vartamaan mein active hain. Har baar jab hum `borrow` call karte hain, `RefCell<T>` apne active immutable borrows ki ginti badha deta hai. Jab ek `Ref<T>` value scope se bahar jaati hai, toh immutable borrows ki ginti ek se kam ho jaati hai. Compile-time borrowing rules ki tarah hi, `RefCell<T>` humein kisi bhi samay mein kai immutable borrows ya ek mutable borrow rakhne deta hai.

Agar hum in niyamon ka ullanghan karne ki koshish karte hain, toh compiler error milne ke bajaye, `RefCell<T>` ka implementation runtime par panic ho jayega. Listing 15-23 ek hi scope mein jaan boojh kar do mutable borrows banane ki koshish karta hai:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
impl Messenger for MockMessenger {
    fn send(&self, message: &str) {
        let mut one_borrow = self.sent_messages.borrow_mut();
        let mut two_borrow = self.sent_messages.borrow_mut();

        one_borrow.push(String::from(message));
        two_borrow.push(String::from(message));
    }
}
```

\<span class="caption"\>Listing 15-23: Ek hi scope mein do mutable references banana yeh dekhne ke liye ki `RefCell<T>` panic hoga\</span\>

Yeh code compile ho jayega lekin test fail ho jayega:

```text
---- tests::it_sends_an_over_75_percent_warning_message stdout ----
	thread 'tests::it_sends_an_over_75_percent_warning_message' panicked at
'already borrowed: BorrowMutError', src/libcore/result.rs:906:4
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Dhyan dein ki code `already borrowed: BorrowMutError` message ke saath panic hua. Is tarah `RefCell<T>` runtime par borrowing rules ke ullanghan ko handle karta hai.

Runtime par borrowing errors pakadne ka matlab hai ki aapko apne code mein galti development process mein der se pata chalegi. Saath hi, runtime par borrows ka track rakhne ke kaaran aapke code mein thodi runtime performance penalty lagegi. Halaanki, `RefCell<T>` ka istemal ek aisa mock object likhna sambhav banata hai jo khud ko modify kar sakta hai jabki aap use ek aise context mein istemal kar rahe hain jahan sirf immutable values ki anumati hai.

-----

## `Rc<T>` aur `RefCell<T>` ko Combine karke Mutable Data ke Multiple Owners Rakhna

`RefCell<T>` ka istemal karne ka ek aam tareeka `Rc<T>` ke saath combination mein hai. Yaad karein ki `Rc<T>` aapko kuch data ke multiple owners rakhne deta hai, lekin yeh us data tak sirf immutable access deta hai. Agar aapke paas ek `Rc<T>` hai jo ek `RefCell<T>` hold karta hai, toh aapko ek aisi value mil sakti hai jiske multiple owners ho sakte hain *aur* jise aap mutate kar sakte hain\!

Listing 15-24 dikhata hai ki `Cons` definition mein `RefCell<T>` ka istemal karke, hum lists mein store ki gayi value ko modify kar sakte hain:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

\<span class="caption"\>Listing 15-24: `Rc<RefCell<i32>>` ka istemal karke ek `List` banana jise hum mutate kar sakte hain\</span\>

Hum `value` naamak variable mein `Rc<RefCell<i32>>` ka ek instance banate hain. Phir hum `a` mein ek `List` banate hain jo `value` hold karta hai. Humein `value` ko clone karna hoga taaki `a` aur `value` dono ke paas andar ki `5` value ka ownership ho.

Jab humne `a`, `b`, aur `c` mein lists bana li hain, hum `value` mein 10 jodte hain. Hum yeh `value` par `borrow_mut` call karke karte hain. `borrow_mut` method ek `RefMut<T>` smart pointer return karta hai, aur hum us par dereference operator ka istemal karke andar ki value ko badal dete hain.

Jab hum `a`, `b`, aur `c` ko print karte hain, toh hum dekh sakte hain ki un sabhi ke paas 5 ke bajaye modified value 15 hai:

```text
a after = Cons(RefCell { value: 15 }, Nil)
b after = Cons(RefCell { value: 6 }, Cons(RefCell { value: 15 }, Nil))
c after = Cons(RefCell { value: 10 }, Cons(RefCell { value: 15 }, Nil))
```

Yeh takneek kaafi acchi hai\! `RefCell<T>` ka istemal karke, hamare paas ek baahari roop se immutable `List` value hai. Lekin hum `RefCell<T>` ke methods ka istemal kar sakte hain jo uske interior mutability tak pahunch pradan karte hain taaki hum zaroorat padne par apne data ko modify kar sakein. Borrowing rules ke runtime checks humein data races se bachate hain, aur kabhi-kabhi apne data structures mein is lacheelepan ke liye thodi si speed trade off karna aarthik hota hai.

Standard library mein aur bhi types hain jo interior mutability pradan karte hain, jaise ki `Cell<T>`. Ek `Mutex<T>` bhi hai, jo interior mutability pradan karta hai jo threads ke beech istemal karne ke liye surakshit hai; hum iske istemal par Chapter 16 mein charcha karenge.
