## Async ke Liye Traits par Ek Gehri Nazar

Is chapter mein, humne `Future`, `Pin`, `Unpin`, `Stream`, aur `StreamExt` traits ka vibhinn tarikon se istemal kiya hai. Ab tak, humne unke kaam karne ya ek saath fit hone ke details mein zyada gehraai se jaane se bacha hai, jo aapke rozmarra ke Rust kaam ke liye aam taur par theek hai. Kabhi-kabhi, aap aisi sthitiyon ka saamna karenge jahan aapko inmein se kuch aur details samajhne ki zaroorat hogi. Is section mein, hum un drishyon mein madad karne ke liye paryapt gehraai se jaanchenge.

-----

### `Future` Trait

Chaliye `Future` trait kaise kaam karta hai, is par ek gehri nazar daalkar shuruaat karte hain. Rust ise is prakaar define karta hai:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Is trait definition mein kai naye types aur kuch syntax shaamil hain jo humne pehle nahi dekha hai, to chaliye is definition ko tukde-tukde karke samjhein.

Pehla, `Future` ka associated type `Output` batata hai ki future kis cheez mein resolve hoga. Doosra, `Future` mein `poll` method bhi hai, jo ek `Poll<Self::Output>` return karta hai. Aaiye `Poll` type par dhyan kendrit karein:

```rust
enum Poll<T> {
    Ready(T),
    Pending,
}
```

Yah `Poll` type `Option` ke samaan hai. Iska ek variant hai jismein ek value hoti hai, `Ready(T)`, aur ek jismein nahi hoti, `Pending`. Halaanki, `Poll` ka arth `Option` se kaafi alag hai\! `Pending` variant sanket deta hai ki future ko abhi bhi kaam karna hai, isliye caller ko baad mein phir se check karna hoga. `Ready` variant sanket deta hai ki future ne apna kaam poora kar liya hai aur `T` value uplabdh hai.

> **Note:** Zyada tar futures ke saath, `Ready` return hone ke baad caller ko `poll` ko dobara call nahi karna chahiye. Kai futures `Ready` hone ke baad dobara poll kiye jaane par panic ho sakte hain.

Jab aap `await` ka istemal karne wala code dekhte hain, to Rust parde ke peeche use `poll` call karne wale code mein compile karta hai. `Pending` hone par kya karna chahiye? Humein baar-baar koshish karne ka koi tareeka chahiye jab tak ki future antतः taiyaar na ho jaaye. Doosre shabdon mein, humein ek loop ki zaroorat hai. Agar Rust ise theek usi code mein compile karta, to har `await` blocking ho jaata. Iske bajaye, Rust yah sunishchit karta hai ki loop control ek aisi cheez ko saunp sake jo is future par kaam rok kar doosre futures par kaam kar sake aur phir baad mein is par vaapas aa sake. Jaisa ki humne dekha hai, vah cheez ek **async runtime** hai.

Runtime har future ko **poll** karta hai jiske liye vah zimmedaar hai, aur jab tak vah taiyaar na ho, future ko vaapas "sula" deta hai.

-----

### `Pin` aur `Unpin` Traits

Jab humne `Pinning` ka idea pesh kiya, to humein ek bahut jatil error message ka saamna karna pada. Iska प्रासंगिक hissa phir se yahan hai:

```text
error[E0277]: `{async block}` cannot be unpinned
   ... the trait `Unpin` is not implemented for `{async block}`
   ... required for `Box<{async block}>` to implement `Future`
```

Yah error message humein na keval yah batata hai ki humein values ko pin karne ki zaroorat hai, balki yah bhi batata hai ki pinning kyon aavashyak hai. `Future` trait ke `poll` method mein `self` parameter ka type `Pin<&mut Self>` hota hai.

`Pin` `&`, `&mut`, `Box` jaise pointer-like types ke liye ek wrapper hai. `Pin` khud ek pointer nahi hai; yah sirf ek tool hai jiska istemal compiler pointer upyog par badhaayein laagoo karne ke liye kar sakta hai.

`async` blocks dwara banaye gaye futures **self-referential** ho sakte hain (yaani, unke fields mein khud ke references ho sakte hain).

*Figure 17-4: Ek self-referential data type.*

Default roop se, koi bhi object jiska khud ka reference ho, use move karna unsafe hota hai, kyunki references hamesha us cheez ke vastavik memory address par point karte hain jise ve refer karte hain. Agar aap data structure ko hi move karte hain, to ve internal references purane location par point karte reh jayenge, jo ab invalid hai.

*Figure 17-5: Ek self-referential data type ko move karne ka unsafe parinaam*

Agar hum yah sunishchit kar sakein ki prashn mein data structure *memory mein move na ho*, to humein kisi bhi reference ko update karne ki zaroorat nahi hogi. Yahi vah guarantee hai jo **`Pin`** humein deta hai. Jab hum ek value ko **pin** karte hain, to vah ab move nahi ho sakti.

Halaanki, zyada tar types bilkul surakshit roop se move kiye ja sakte hain. Humein keval un items ke baare mein pinning ke baare mein sochna hoga jinmein internal references hote hain. Humein compiler ko yah batane ka ek tareeka chahiye ki aise maamlon mein items ko move karna theek hai—aur yahi vah jagah hai jahan **`Unpin`** kaam aata hai.

`Unpin` ek marker trait hai, jo `Send` aur `Sync` traits ke samaan hai. `Unpin` compiler ko soochit karta hai ki ek diye gaye type ko yah guarantee banaye rakhne ki zaroorat nahi hai ki prashn mein value surakshit roop se move ki ja sakti hai ya nahi.

  * **`Unpin`**: "Normal" case hai. Iska matlab hai ki type move karne ke liye surakshit hai, bhale hi vah pinned ho. Rust mein zyada tar types `Unpin` hote hain.
  * **`!Unpin`**: "Special" case hai. Iska matlab hai ki type self-referential ho sakta hai aur use move karna unsafe hai. `async` blocks dwara banaye gaye futures `!Unpin` hote hain.

Ab hum `join_all` call ke liye report kiye gaye errors ko samajh sakte hain. Humne `async` blocks dwara paida kiye gaye futures ko ek `Vec` mein move karne ki koshish ki, lekin jaisa ki humne dekha hai, un futures mein internal references ho sakte hain, isliye ve `Unpin` implement nahi karte. Unhe pin karne ki zaroorat hai, aur phir hum `Pin` type ko `Vec` mein pass kar sakte hain, is vishvas ke saath ki futures mein underlying data move *nahi* hoga.

`Pin` aur `Unpin` zyada tar lower-level libraries banane ke liye mahatvapurna hain. Jab aap error messages mein in traits ko dekhte hain, to ab aapke paas apne code ko theek karne ke baare mein ek behtar idea hoga\!

-----

### `Stream` Trait

Ab jab aapko `Future`, `Pin`, aur `Unpin` traits ki gehri samajh ho gayi hai, hum apna dhyan `Stream` trait par kendrit kar sakte hain. Jaisa ki aapne chapter mein pehle seekha, streams asynchronous iterators ke samaan hain.

Chaliye `Stream` trait ko `Iterator` aur `Future` traits ke ideas ko milakar dekhte hain.

  * `Iterator` se, humein ek kram (sequence) ka idea milta hai: iska `next` method ek `Option<Self::Item>` pradan karta hai.
  * `Future` se, humein samay ke saath taiyaari (readiness) ka idea milta hai: iska `poll` method ek `Poll<Self::Output>` pradan karta hai.

Samay ke saath taiyaar hone wale items ke kram ko represent karne ke liye, hum ek `Stream` trait define karte hain jo un features ko ek saath rakhta hai:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

`Stream` trait `Item` naamak ek associated type define karta hai. `Stream` `poll_next` naamak ek method bhi define karta hai. Iska return type `Poll` ko `Option` ke saath jodta hai.

  * Outer type `Poll` hai, kyunki ise readiness ke liye check karna padta hai, bilkul ek future ki tarah.
  * Inner type `Option` hai, kyunki ise yah sanket dene ki zaroorat hai ki aur messages hain ya nahi, bilkul ek iterator ki tarah.

Lekin, humne streaming ke section mein `poll_next` ka istemal nahi kiya, balki `next` aur `StreamExt` ka istemal kiya. `await` ka istemal karna bahut accha hai, aur **`StreamExt`** trait `next` method pradan karta hai taaki hum theek wahi kar sakein:

```rust
trait StreamExt: Stream {
    async fn next(&mut self) -> Option<Self::Item>
    where
        Self: Unpin,
    {
        // ... default implementation ...
    }
    // ... other methods like `filter`, `map`, etc. ...
}
```

`StreamExt` trait streams ke saath istemal ke liye uplabdh sabhi dilchasp methods ka bhi ghar hai. `StreamExt` svachaalit roop se har us type ke liye implement kiya jaata hai jo `Stream` implement karta hai. Iska matlab hai ki bhale hi aapko apna khud ka streaming data type likhna pade, aapko *sirf* `Stream` implement karna hoga, aur phir koi bhi jo aapke data type ka istemal karta hai, vah `StreamExt` aur uske methods ka istemal svachaalit roop se kar sakta hai.