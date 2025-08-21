# `panic!` Karein Ya Nahi? 🤔

Aap kaise decide karein ki kab `panic!` call karna hai aur kab `Result` return karna hai? Jab code **panic** karta hai, to usse recover karne ka koi tareeka nahi hai. Aap kisi bhi error situation ke liye `panic!` call kar sakte hain, bhale hi recover karne ka koi tareeka ho ya na ho, lekin tab aap apne code ko call karne wale code ke liye ye decision le rahe hain ki ye situation unrecoverable hai. Jab aap `Result` value return karte hain, to aap calling code ko options dete hain. Calling code decide kar sakta hai ki uske situation ke liye kaise recover karna hai, ya wo decide kar sakta hai ki is case mein `Err` value unrecoverable hai, to wo khud `panic!` call karke aapke recoverable error ko unrecoverable error mein badal sakta hai. Isliye, jab aap ek aisa function define kar rahe hain jo fail ho sakta hai, to **`Result` return karna ek accha default choice hai.**

Bahut hi rare situations mein, `Result` return karne ke bajaye code ka **panic** hona zyaada appropriate hai. Aaiye dekhte hain ki examples, prototype code, aur tests mein panic karna kyon sahi hai. Hum un situations par bhi discuss karenge jahan compiler nahi bata sakta ki failure impossible hai, lekin aap insaan ke naate bata sakte hain.

-----

### Examples, Prototype Code aur Tests 🧪

Jab aap kisi concept ko samjhane ke liye ek example likh rahe hote hain, to usmein robust error-handling code hone se example less clear ho sakta hai. Examples mein, ye mana jaata hai ki `unwrap` jaisa method jo panic kar sakta hai, ek **placeholder** hai us tareeke ke liye jisse aap apni application mein errors ko handle karna chahte hain.

Isi tarah, `unwrap` aur `expect` methods prototyping ke liye bahut kaam aate hain, jab aap abhi errors ko handle karne ka final decision lene ke liye ready nahi hain. Ye aapke code mein saaf markers chhod dete hain ki kab aap apne program ko aur robust banane ke liye taiyar hain.

Agar ek test mein ek method call fail ho jaata hai, to aap chahte hain ki poora test hi fail ho jaaye, bhale hi wo method test kiye jaane wali functionality na ho. Kyunki `panic!` se hi ek test ko as a failure mark kiya jaata hai, `unwrap` ya `expect` call karna bilkul wahi hai jo hona chahiye.

-----

### Aise Cases Jab Aapke Paas Compiler Se Zyaada Jankari Ho 🧠

`unwrap` ko call karna tab bhi appropriate hai jab aapke paas kuch aisa logic ho jo ensure karta hai ki `Result` mein `Ok` value hi hogi, lekin wo logic compiler nahi samajhta. Aapke paas abhi bhi ek `Result` value hogi jise aapko handle karna hai, lekin aap ensure kar sakte hain ki `Err` variant kabhi nahi aayega, to `unwrap` call karna bilkul theek hai. Yahan ek example hai:

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1".parse().unwrap();
```

Hum ek **hardcoded string** ko parse karke ek `IpAddr` instance bana rahe hain. Hum dekh sakte hain ki `127.0.0.1` ek valid IP address hai, isliye yahan `unwrap` use karna acceptable hai. Lekin, ek hardcoded, valid string `parse` method ke return type ko nahi badalta: humein abhi bhi ek `Result` value milti hai, aur compiler abhi bhi humein `Result` ko handle karne ke liye majboor karega jaise ki `Err` variant ek possibility hai. Agar IP address string user se aata aur fail hone ki possibility hoti, to humein `Result` ko zyaada robust tareeke se handle karna chahiye tha.

-----

### Error Handling ke liye Guidelines 💡

Ye advisable hai ki aapke code ko tab **panic** karna chahiye jab ye possible ho ki aapka code ek **kharab halat (bad state)** mein aa sakta hai. Is context mein, ek kharab halat tab hoti hai jab koi **assumption, guarantee, contract, ya invariant** toot gaya ho, jaise ki jab aapke code ko invalid, contradictory, ya missing values diye jaate hain—plus ek ya ek se zyaada in baaton ka hona:

  * Ye kharab halat aisi nahi hai jiske aksar hone ki umeed ho.
  * Is point ke baad aapka code is kharab halat mein na hone par depend karta hai.
  * Is jaankari ko aapke types mein encode karne ka koi accha tareeka nahi hai.

Agar koi aapke code ko call karta hai aur aisi values deta hai jinka koi matlab nahi hai, to sabse accha choice ye ho sakta hai ki `panic!` call karke us library ko use karne wale insaan ko unke code mein bug ke baare mein alert kiya jaaye taaki wo use development ke dauran fix kar sakein.

Jab ek kharab halat tak pahuncha jaata hai, lekin uske hone ki umeed hoti hai, to **`Result` return karna zyaada appropriate hai** na ki `panic!` call karna. Examples mein, ek parser ko malformed data diya jaana ya ek HTTP request ka status return karna jo indicate karta hai ki aapne rate limit hit kar di hai, shamil hai. Aise cases mein, aapko `Result` return karke ye indicate karna chahiye ki failure ek expected possibility hai.

Jab aapka code values par operations perform karta hai, to use pehle values ko verify karna chahiye aur agar values valid nahi hain to panic karna chahiye. Ye safety reasons ke liye hai: invalid data par operate karne se aapka code vulnerabilities ke samne expose ho sakta hai. Yahi main reason hai ki jab aap out-of-bounds memory access ki koshish karte hain to standard library `panic!` call karta hai.

-----

### Validation ke liye Custom Types Banana 🏗️

Aaiye Rust ke type system ka upyog karke valid value ensure karne ke idea ko ek kadam aage le jaate hain aur validation ke liye ek **custom type** banate hain. Chapter 2 ke guessing game ko yaad karein, jismein humne user se 1 aur 100 ke beech ek number guess karne ko kaha tha. Humne kabhi validate nahi kiya ki user ka guess un numbers ke beech tha. Agar ye bilkul critical hota ki program sirf 1 aur 100 ke beech ki values par hi operate kare, to har function mein check karna **tedious** hota.

Iske bajaye, hum ek naya type bana sakte hain aur validations ko ek function mein daal sakte hain taaki har jagah validations repeat na karne paden. Isse functions ke liye naye type ka use karna aur unhein mili values par confident rehna safe ho jaata hai. **Listing 9-9** ek `Guess` type define karne ka ek tareeka dikhata hai jo `Guess` ka instance tabhi banayega jab `new` function ko 1 aur 100 ke beech ki value milti hai.

```rust
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }

    pub fn value(&self) -> u32 {
        self.value
    }
}
```

\<span class="caption"\>Listing 9-9: Ek `Guess` type jo sirf 1 aur 100 ke beech ki values ko hi accept karega\</span\>

Hum ek `Guess` struct define karte hain jismein ek `value` field hai. Phir hum `Guess` par ek associated function `new` implement karte hain jo `Guess` values ke instances banata hai. `new` function `value` ko test karta hai aur agar ye 1 aur 100 ke beech nahi hai, to hum `panic!` call karte hain, jo programmer ko alert karta hai ki unhein ek bug fix karne ki zaroorat hai.

Hum ek `value` naam ka method bhi implement karte hain. Is tarah ke method ko **getter** kehte hain. Ye **public method** isliye zaroori hai kyunki `Guess` struct ka `value` field **private** hai. `value` field ka private hona important hai, taaki `Guess` struct ka use karne wala code `value` ko directly set na kar sake: code ko `Guess::new` function ka upyog karna hi hoga, jisse ensure hota hai ki `Guess` ki `value` hamesha check ki gayi hai.

-----

## Summary 📝

Rust ke error handling features aapko zyaada **reliable** code likhne mein madad karne ke liye design kiye gaye hain. **`panic!` macro** signal deta hai ki aapka program aisi halat mein hai jise wo handle nahi kar sakta aur process ko rokne deta hai. **`Result` enum** Rust ke type system ka use karke indicate karta hai ki operations aise tareeke se fail ho sakte hain jise aapka code recover kar sakta hai. Sahi situations mein `panic!` aur `Result` ka upyog karne se aapka code inevitable problems ke samne zyaada reliable banega.

Ab jab aapne dekha hai ki standard library `Option` aur `Result` enums ke saath generics ka kaise use karti hai, hum generics ke baare mein baat karenge aur aap unhein apne code mein kaise use kar sakte hain.
