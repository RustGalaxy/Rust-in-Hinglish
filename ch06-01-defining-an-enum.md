# ENUM ko Define Karna

Chaliye ek aisi situation ko dekhte hain jise hum code mein express karna chahte hain aur samajhte hain ki aisi situation mein **enums** kyun useful hote hain aur structs se behtar kyun hain. Maan lijiye hum IP addresses ke saath kaam karna chahte hain. Filhal, IP addresses ke liye do bade standards use hote hain: version four aur version six. Hamare program mein jo IP address aayega uske liye yahi do possibilities hain: hum sabhi possible values ko **enumerate** kar sakte hain, aur yahi se enumeration ko uska naam mila hai.

Koi bhi IP address ya toh version four ya version six address ho sakta hai, lekin ek hi samay mein dono nahi. IP addresses ki yeh property enum data structure ko appropriate banati hai, kyunki enum values variants mein se sirf ek hi ho sakte hain. Version four aur version six dono addresses ab bhi fundamentally IP addresses hi hain, isliye jab code un situations ko handle kar raha ho jo kisi bhi IP address par lagu hoti hain, toh unhe ek hi type ke roop mein treat kiya jana chahiye.

Hum is concept ko code mein `IpAddrKind` enumeration ko define karke aur ek IP address ke possible kinds, `V4` aur `V6` ko list karke express kar sakte hain. Inhe enum ke **variants** ke roop mein jana jata hai:

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

Ab `IpAddrKind` ek custom data type hai jise hum apne code mein kahin bhi use kar sakte hain.

### Enum Values (Enum ki Values)

Hum `IpAddrKind` ke do variants mein se har ek ke instances is tarah bana sakte hain:

```rust
# enum IpAddrKind {
#      V4,
#      V6,
# }
#
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

Dhyan dein ki enum ke variants uske identifier ke under namespaced hain, aur hum dono ko alag karne ke liye double colon ka upyog karte hain. Iska fayda yeh hai ki ab dono values `IpAddrKind::V4` aur `IpAddrKind::V6` ek hi type ke hain: `IpAddrKind`. Hum, udharanan, ek function define kar sakte hain jo koi bhi `IpAddrKind` leta hai:

```rust
# enum IpAddrKind {
#      V4,
#      V6,
# }
#
fn route(ip_type: IpAddrKind) { }
```

Aur hum is function ko kisi bhi variant ke saath call kar sakte hain:

```rust
# enum IpAddrKind {
#      V4,
#      V6,
# }
#
# fn route(ip_type: IpAddrKind) { }
#
route(IpAddrKind::V4);
route(IpAddrKind::V6);
```

Enums ka upyog karne ke aur bhi fayde hain. Apne IP address type ke bare mein aur sochte hue, filhal hamare paas actual IP address **data** ko store karne ka koi tarika nahi hai; humein bas uski **kind** ka pata hai. Kyunki aapne Chapter 5 mein structs ke bare mein abhi hi sikha hai, aap is problem ko Listing 6-1 mein dikhaye gaye tarike se tackle kar sakte hain.

```rust
enum IpAddrKind {
    V4,
    V6,
}

struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};
```

\<span class="caption"\>Listing 6-1: IP address ke data aur `IpAddrKind` variant ko `struct` ka upyog karke store karna\</span\>

Yahaan, humne ek struct `IpAddr` define kiya hai jismein do fields hain: ek `kind` field jiska type `IpAddrKind` hai (jo humne pehle define kiya hai) aur ek `address` field jiska type `String` hai. Hamare paas is struct ke do instances hain. Pehle, `home` ki `kind` value `IpAddrKind::V4` hai, jiske saath `127.0.0.1` ka associated address data hai. Doosre instance, `loopback` ki `kind` value `IpAddrKind` ka doosra variant, `V6`, hai, aur uske saath `::1` address associated hai. Humne `kind` aur `address` values ko ek saath bundle karne ke liye ek struct ka upyog kiya hai, isliye ab variant value ke saath associated hai.

Hum is concept ko sirf ek enum ka upyog karke aur bhi concise tarike se express kar sakte hain, na ki ek struct ke andar ek enum ka upyog karke. Yeh data ko sidhe har enum variant mein daal kar kiya jata hai. `IpAddr` enum ki yeh nayi definition kehti hai ki `V4` aur `V6` dono variants ke paas associated `String` values hongi:

```rust
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```

Hum data ko sidhe enum ke har variant se attach karte hain, isliye ek extra struct ki zaroorat nahi hai.

Enum ka upyog karne ka ek aur fayda hai: har variant mein alag-alag types aur amount ka associated data ho sakta hai. Version four type ke IP addresses mein hamesha char numeric components honge jinki values 0 aur 255 ke beech hongi. Agar hum `V4` addresses ko char `u8` values ke roop mein store karna chahte hain lekin phir bhi `V6` addresses ko ek `String` value ke roop mein express karna chahte hain, toh hum ek struct ke saath aisa nahi kar pate. Enums is case ko aasaani se handle karte hain:

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

Humne version four aur version six IP addresses ko store karne ke liye data structures ko define karne ke kai alag-alag tarike dikhaye hain. Lekin, jaisa ki pata chalta hai, IP addresses ko store karna aur yeh encode karna ki woh kis tarah ke hain itna common hai ki **standard library mein ek definition hai jise hum use kar sakte hain\!** Chaliye dekhte hain ki standard library `IpAddr` ko kaise define karti hai: ismein bilkul wahi enum aur variants hain jo humne define aur use kiye hain, lekin yeh address data ko do alag-alag structs ke roop mein variants ke andar embed karta hai, jo har variant ke liye alag se define kiye gaye hain:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Yeh code batata hai ki aap enum variant ke andar kisi bhi tarah ka data daal sakte hain: jaise strings, numeric types, ya structs. Aap dusra enum bhi shamil kar sakte hain\! Aur, standard library types aksar utne complex nahi hote hain jitna aap soch sakte hain.

Dhyan dein ki halanki standard library mein `IpAddr` ki definition hai, hum ab bhi apni definition bana sakte hain aur use bina kisi conflict ke use kar sakte hain kyunki humne standard library ki definition ko apne scope mein nahi laya hai. Hum Chapter 7 mein types ko scope mein lane ke bare mein aur baat karenge.

Listing 6-2 mein ek aur enum ka example dekhte hain: ismein iske variants mein alag-alag tarah ke types embed hain.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

\<span class="caption"\>Listing 6-2: Ek `Message` enum jiske variants har ek alag amount aur types ki values store karte hain\</span\>

Is enum mein alag-alag types ke saath char variants hain:

  * `Quit` ke saath bilkul bhi data associated nahi hai.
  * `Move` mein ek anonymous struct shamil hai.
  * `Write` mein ek single `String` shamil hai.
  * `ChangeColor` mein teen `i32` values shamil hain.

Listing 6-2 mein diye gaye jaise variants ke saath ek enum ko define karna alag-alag tarah ke struct definitions ko define karne jaisa hi hai, bas farak yeh hai ki enum `struct` keyword ka upyog nahi karta hai aur saare variants `Message` type ke under ek saath group kiye gaye hain. Neeche diye gaye structs wahi data hold kar sakte hain jo upar diye gaye enum variants hold karte hain:

```rust
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

Lekin agar hum alag-alag structs ka upyog karte, jismein har ek ka apna type hota hai, toh hum aasaani se ek function define nahi kar pate jo in mein se kisi bhi tarah ke messages ko le sake, jaisa hum Listing 6-2 mein define kiye gaye `Message` enum ke saath kar sakte hain, jo ek single type hai.

Enums aur structs ke beech ek aur similarity hai: jaise hum `impl` ka upyog karke structs par methods define kar sakte hain, hum enums par bhi methods define kar sakte hain. Yahaan `call` naam ka ek method hai jise hum apne `Message` enum par define kar sakte hain:

```rust
# enum Message {
#     Quit,
#     Move { x: i32, y: i32 },
#     Write(String),
#     ChangeColor(i32, i32, i32),
# }
#
impl Message {
    fn call(&self) {
        // method body yahan define hoga
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

Method ki body `self` ka upyog karegi us value ko pane ke liye jis par humne method ko call kiya hai. Is example mein, humne ek variable `m` banaya hai jismein `Message::Write(String::from("hello"))` value hai, aur `m.call()` chalne par `call` method ki body mein `self` yahi hoga.

-----

### The `Option` Enum and Its Advantages Over Null Values (Option Enum aur Null Values par iske Fayde)

Pichhle section mein, humne dekha ki `IpAddr` enum ne humein Rust ke type system ka upyog karke apne program mein data se zyada information encode karne ki permission di. Yeh section `Option` ki ek case study ko explore karta hai, jo standard library dwara define kiya gaya ek aur enum hai. `Option` type ka upyog kai jagahon par hota hai kyunki yeh us bahut hi common scenario ko encode karta hai jismein ek value kuch ho sakti hai ya bilkul bhi nahi ho sakti hai. Is concept ko type system ke terms mein express karne ka matlab hai ki compiler check kar sakta hai ki kya aapne un sabhi cases ko handle kiya hai jinhein aapko handle karna chahiye; yeh functionality un bugs ko rok sakti hai jo doosri programming languages mein bahut common hain.

Programming language design ko aksar is terms mein socha jata hai ki aap kaun se features shamil karte hain, lekin jo features aap exclude karte hain woh bhi important hote hain. Rust mein **null** feature nahi hai jo bahut si doosri languages mein hota hai. Null ek aisi value hai jiska matlab hai ki wahan koi value nahi hai. Null wali languages mein, variables hamesha do states mein se ek mein ho sakte hain: null ya not-null.

2009 mein apne presentation **"Null References: The Billion Dollar Mistake"** mein, Tony Hoare, jo null ke inventor hain, ne yeh kaha:

> "Main ise apni billion-dollar mistake kehta hoon. Us samay, main ek object-oriented language mein references ke liye pehla comprehensive type system design kar raha tha. Mera lakshya yeh tha ki references ka sabhi upyog bilkul safe ho, jise compiler dwara automatically check kiya jaye. Lekin main ek null reference dalne ke temptation ko rok nahi paya, bas isliye kyunki isse implement karna bahut aasan tha. Isse anaginat errors, vulnerabilities, aur system crashes hue hain, jisse pichle chalis saalon mein shayad ek billion dollars ka nuksan hua hai."

Null values ke saath problem yeh hai ki agar aap ek null value ko not-null value ke roop mein upyog karne ki koshish karte hain, toh aapko kisi na kisi tarah ki error milegi. Kyunki yeh null ya not-null property har jagah hai, is tarah ki error karna bahut aasan hai.

Lekin, woh concept jo null express karne ki koshish kar raha hai woh ab bhi useful hai: ek null ek aisi value hai jo abhi invalid ya kisi karan se absent hai.

Problem asal mein concept ke saath nahi hai, balki particular implementation ke saath hai. Is tarah, Rust mein nulls nahi hain, lekin iske paas ek enum hai jo ek value ke maujood hone ya absent hone ke concept ko encode kar sakta hai. Is enum ko `Option<T>` kehte hain, aur isse standard library dwara is tarah define kiya gaya hai:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

`Option<T>` enum itna useful hai ki yeh prelude mein bhi shamil hai; aapko isse explicitly scope mein lane ki zaroorat nahi hai. Iske alawa, iske variants bhi hain: aap `Some` aur `None` ko sidhe `Option::` prefix ke bina upyog kar sakte hain. `Option<T>` enum ab bhi ek regular enum hai, aur `Some(T)` aur `None` ab bhi `Option<T>` type ke variants hain.

`<T>` syntax Rust ka ek feature hai jiske bare mein humne abhi tak baat nahi ki hai. Yeh ek **generic type parameter** hai, aur hum Chapter 10 mein generics ko aur detail mein cover karenge. Abhi ke liye, aapko bas yeh janne ki zaroorat hai ki `<T>` ka matlab hai ki `Option` enum ka `Some` variant kisi bhi type ke ek piece of data ko hold kar sakta hai. Yahaan number types aur string types ko hold karne ke liye `Option` values ka upyog karne ke kuch examples hain:

```rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

Agar hum `Some` ke bajaye `None` ka upyog karte hain, toh humein Rust ko batana padta hai ki hamare paas kis type ka `Option<T>` hai, kyunki compiler sirf `None` value ko dekh kar us type ka anuman nahi laga sakta jise `Some` variant hold karega.

Jab hamare paas ek `Some` value hoti hai, toh hum jante hain ki ek value maujood hai aur woh value `Some` ke andar held hai. Jab hamare paas ek `None` value hoti hai, toh kuch mayne mein, iska matlab null jaisa hi hai: hamare paas ek valid value nahi hai. Toh phir `Option<T>` hona null se behtar kyun hai?

Saadha shabdon mein, kyunki `Option<T>` aur `T` (jahan `T` koi bhi type ho sakta hai) alag-alag types hain, compiler humein ek `Option<T>` value ko aise upyog nahi karne dega jaise ki woh definitely ek valid value ho. Udaharan ke liye, yeh code compile nahi hoga kyunki yeh ek `i8` ko ek `Option<i8>` mein jodane ki koshish kar raha hai:

```rust,ignore
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
```

Agar hum is code ko chalate hain, toh humein is tarah ka error message milta hai:

```text
error[E0277]: the trait bound `i8: std::ops::Add<std::option::Option<i8>>` is
not satisfied
 -->
  |
5 |     let sum = x + y;
  |                   ^ no implementation for `i8 + std::option::Option<i8>`
  |
```

Gaur se dekho\! Is error message ka matlab hai ki Rust nahi samajhta ki ek `i8` aur ek `Option<i8>` ko kaise joda jaye, kyunki woh alag-alag types hain. Jab hamare paas Rust mein `i8` jaisa koi type hota hai, toh compiler ensure karega ki hamare paas hamesha ek valid value ho. Hum us value ka upyog karne se pehle null ke liye check kiye bina confident rah sakte hain. Sirf tab jab hamare paas ek `Option<i8>` (ya jo bhi type ki value ke saath hum kaam kar rahe hain) hota hai, tabhi humein possibly value nahi hone ke bare mein chinta karni padti hai, aur compiler ensure karega ki hum value ka upyog karne se pehle us case ko handle karein.

Dusre shabdon mein, aapko ek `Option<T>` ko ek `T` mein convert karna padta hai usse pehle ki aap uske saath `T` operations kar sakein. Aam taur par, yeh null ke saath sabse common issues mein se ek ko pakadne mein madad karta hai: yeh manna ki koi cheez null nahi hai jabki woh asal mein null ho.

Galat tarike se not-null value maan lene ki chinta na karke aap apne code mein aur bhi confident ho sakte hain. Ek aisi value rakhne ke liye jo possibly null ho sakti hai, aapko us value ka type `Option<T>` banakar explicitly opt in karna padta hai. Phir, jab aap us value ka upyog karte hain, toh aapko explicitly us case ko handle karna padta hai jab value null hoti hai. Har jagah jahan ek value ka type `Option<T>` nahi hai, aap surakshit roop se maan sakte hain ki value null nahi hai. Yeh Rust ke liye ek deliberate design decision tha taaki null ki vyapta ko seemit kiya ja sake aur Rust code ki safety badhayi ja sake.

Toh, jab aapke paas `Option<T>` type ki value hoti hai toh aap ek `Some` variant se `T` value ko kaise nikalte hain taaki aap us value ka upyog kar sakein? `Option<T>` enum mein badi sankhya mein methods hain jo alag-alag situations mein useful hote hain; aap unhein iske documentation mein check kar sakte hain. `Option<T>` par methods se familiar hona aapki Rust journey mein behad useful hoga.

Aam taur par, ek `Option<T>` value ka upyog karne ke liye, aap chahte hain ki aapke paas aisa code ho jo har variant ko handle karega. Aap chahte hain ki kuch code tabhi chale jab aapke paas ek `Some(T)` value ho, aur is code ko inner `T` ka upyog karne ki permission hoti hai. Aap chahte hain ki koi aur code tab chale agar aapke paas ek `None` value ho, aur us code ke paas koi `T` value available nahi hoti hai. `match` expression ek control flow construct hai jo enums ke saath upyog hone par bilkul yahi karta hai: yeh alag-alag code chalayega is baat par depend karte hue ki uske paas enum ka kaun sa variant hai, aur woh code matching value ke andar ke data ka upyog kar sakta hai.
