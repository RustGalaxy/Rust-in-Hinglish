## Slice Type

Ek aur data type jo **ownership** nahi rakhta, woh hai **slice**. Slices aapko poore collection ke bajaye, uske elements ke ek contiguous sequence (lagatar anukram) ko refer karne dete hain.

Yahaan ek chhota sa programming problem hai: ek function likhein jo ek string leta hai aur us string mein use milne wala pehla word return karta hai. Agar function ko string mein koi space nahi milta, toh poori string hi ek word hai, isliye poori string return honi chahiye.

Chaliye is function ke signature ke bare mein sochte hain:

```rust,ignore
fn first_word(s: &String) -> ?
```

Yeh function, `first_word`, mein parameter ke roop mein `&String` hai. Humein ownership nahi chahiye, isliye yeh theek hai. Lekin humein kya return karna chahiye? Hamare paas string ke **part** ke bare mein baat karne ka koi tarika nahi hai. Halanki, hum word ke end ka index return kar sakte hain. Chaliye Listing 4-7 mein yeh try karte hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

\<span class="caption"\>Listing 4-7: `first_word` function jo `String` parameter mein ek byte index value return karta hai\</span\>

Kyunki humein `String` ke har element ko ek-ek karke check karna hai ki kya woh ek space hai, hum apni `String` ko `as_bytes` method ka upyog karke bytes ke ek array mein convert karenge:

```rust,ignore
let bytes = s.as_bytes();
```

Next, hum `iter` method ka upyog karke bytes ke array par ek iterator banate hain:

```rust,ignore
for (i, &item) in bytes.iter().enumerate() {
```

Hum iterators ke bare mein Chapter 13 mein aur detail mein discuss karenge. Abhi ke liye, yeh jaan lein ki `iter` ek method hai jo ek collection ke har element ko return karta hai aur `enumerate` `iter` ke result ko wrap karta hai aur har element ko ek tuple ke hisse ke roop mein return karta hai. `enumerate` se return kiya gaya tuple ka pehla element index hai, aur doosra element element ka ek reference hai. Yeh index ko khud se calculate karne se thoda zyada convenient hai.

Kyunki `enumerate` method ek tuple return karta hai, hum us tuple ko destructure karne ke liye patterns ka upyog kar sakte hain, jaise Rust mein har jagah. Toh `for` loop mein, hum ek pattern specify karte hain jismein tuple mein index ke liye `i` aur single byte ke liye `&item` hota hai. Kyunki humein `.iter().enumerate()` se element ka ek reference milta hai, hum pattern mein `&` ka upyog karte hain.

`for` loop ke andar, hum byte literal syntax ka upyog karke space ko represent karne wale byte ko search karte hain. Agar humein ek space milta hai, toh hum position return karte hain. Varna, hum `s.len()` ka upyog karke string ki length return karte hain:

```rust,ignore
    if item == b' ' {
        return i;
    }
}

s.len()
```

Ab hamare paas string mein pehle word ke end ka index pata karne ka ek tarika hai, lekin ek problem hai. Hum akele `usize` return kar rahe hain, lekin yeh sirf `&String` ke context mein hi ek meaningful number hai. Doosre shabdon mein, kyunki yeh `String` se ek alag value hai, koi guarantee nahi hai ki yeh future mein valid rahegi. Listing 4-8 mein diye gaye program ko consider karein jo Listing 4-7 se `first_word` function ka upyog karta hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# fn first_word(s: &String) -> usize {
#     let bytes = s.as_bytes();
#
#     for (i, &item) in bytes.iter().enumerate() {
#         if item == b' ' {
#             return i;
#         }
#     }
#
#     s.len()
# }
#
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word ko value 5 milegi

    s.clear(); // yeh String ko khali kar deta hai, isse "" ke barabar banata hai

    // word abhi bhi yahan value 5 rakhta hai, lekin ab koi string nahi hai jiske saath
    // hum meaningfully value 5 ka upyog kar sakein. word ab poori tarah se invalid hai!
}
```

\<span class="caption"\>Listing 4-8: `first_word` function ko call karne se result store karna aur phir `String` ke contents ko badalna\</span\>

Yeh program bina kisi error ke compile hota hai aur agar hum `s.clear()` call karne ke baad `word` ka upyog karte hain, toh bhi aisa hi hoga. Kyunki `word` `s` ki state se bilkul bhi connected nahi hai, `word` mein abhi bhi value `5` hai. Hum us value `5` ka upyog `s` variable ke saath karke pehla word nikalne ki koshish kar sakte hain, lekin yeh ek bug hoga kyunki `s` ke contents badal gaye hain jab se humne `5` ko `word` mein save kiya tha.

`word` mein index ka `s` mein data ke saath out of sync hone ke bare mein chinta karna tedious aur error prone hai\! In indices ko manage karna aur bhi fragile hai agar hum `second_word` function likhte hain. Uska signature is tarah dikhna chahiye:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Ab hum ek starting aur ek ending index track kar rahe hain, aur hamare paas aur bhi values hain jo ek particular state mein data se calculate ki gayi thin, lekin us state se bilkul bhi bandhi hui nahi hain. Ab hamare paas teen unrelated variables hain jo aas-paas ghoom rahe hain aur unhe sync mein rakhne ki zaroorat hai.

Khushkismati se, Rust ke paas is problem ka ek solution hai: **string slices**.

-----

### `String Slices` (String Slices)

Ek **string slice** ek `String` ke hisse ka reference hai, aur yeh is tarah dikhta hai:

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

Yeh poore `String` ka reference lene jaisa hi hai, lekin `[0..5]` bit ke saath. Poore `String` ke reference ke bajaye, yeh `String` ke ek hisse ka reference hai. `start..end` syntax ek range hai jo `start` par shuru hoti hai aur `end` tak chalti hai, lekin usko shamil nahi karti.

Hum brackets ke andar ek range ka upyog karke slices bana sakte hain, `[starting_index..ending_index]` specify karke, jahan `starting_index` slice mein pehli position hai aur `ending_index` slice mein aakhri position se ek zyada hai. Internally, slice data structure starting position aur slice ki length ko store karti hai, jo `ending_index` minus `starting_index` ke barabar hoti hai. Toh `let world = &s[6..11];` ke case mein, `world` ek slice hoga jismein `s` ke 7th byte ka ek pointer aur `5` ki length value hogi.

\<span class="caption"\>Figure 4-6: String slice `String` ke ek hisse ko refer karta hua\</span\>

Rust ke `..` range syntax ke saath, agar aap pehle index (zero) se shuru karna chahte hain, toh aap do periods se pehle value ko chhod sakte hain. Doosre shabdon mein, yeh dono barabar hain:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Isi tarah, agar aapke slice mein `String` ka aakhri byte shamil hai, toh aap aakhri number ko chhod sakte hain. Iska matlab hai ki yeh dono barabar hain:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Aap poori string ka slice lene ke liye dono values ko bhi chhod sakte hain. Toh yeh dono barabar hain:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> **Note:** String slice range indices ko valid UTF-8 character boundaries par hona chahiye. Agar aap multibyte character ke beech mein ek string slice banane ki koshish karte hain, toh aapka program error ke saath exit ho jayega. String slices ko introduce karne ke uddeshya se, hum is section mein sirf ASCII maan rahe hain; UTF-8 handling par aur poori charcha Chapter 8 ke "Storing UTF-8 Encoded Text with Strings" section mein hai.

Is sabhi jankari ko dhyan mein rakhte hue, chaliye `first_word` ko ek slice return karne ke liye dobara likhte hain. Jo type "string slice" ko signify karta hai use `&str` ke roop mein likha jata hai:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

Humein word ke end ka index usi tarike se milta hai jaisa humne Listing 4-7 mein kiya tha, ek space ki pehli occurrence ko dekh kar. Jab humein ek space milta hai, toh hum string ke shuruat aur space ke index ka upyog karke ek string slice return karte hain.

Ab jab hum `first_word` ko call karte hain, toh humein ek single value wapas milti hai jo underlying data se bandhi hui hai. Value slice ke starting point ke reference aur slice mein elements ki sankhya se banti hai.

`second_word` function ke liye bhi ek slice return karna kaam karega:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Ab hamare paas ek straightforward API hai jisse gadbad karna bahut mushkil hai, kyunki compiler yeh ensure karega ki `String` mein references valid rahenge. Listing 4-8 mein program mein bug ko yaad karein, jab humein pehle word ke end ka index mila tha lekin phir string ko clear kar diya tha jisse hamara index invalid ho gaya tha? Woh code logically galat tha lekin koi immediate errors nahi dikhaye the. Problems baad mein tab aati jab hum khali string ke saath pehle word index ka upyog karne ki koshish karte rehte. Slices is bug ko impossible bana dete hain aur humein hamare code mein problem hone ke bare mein bahut jaldi bata dete hain. `first_word` ke slice version ka upyog karne se compile-time error aayegi:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // error!
}
```

Yahaan compiler error hai:

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let word = first_word(&s);
  |                                - immutable borrow occurs here
5 |
6 |     s.clear(); // error!
  |     ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

Borrowing rules se yaad karein ki agar hamare paas kisi cheez ka immutable reference hai, toh hum ek mutable reference bhi nahi le sakte. Kyunki `clear` ko `String` ko truncate karne ki zaroorat hoti hai, yeh ek mutable reference lene ki koshish karta hai, jo fail ho jata hai. Rust ne hamare API ko upyog karne mein aasan hi nahi banaya hai, balki isne compile time par errors ke poore class ko bhi eliminate kar diya hai\!

-----

#### `String Literals Are Slices` (String Literals Slices Hain)

Yaad karein ki humne string literals ke bare mein baat ki thi jo binary ke andar store hote hain. Ab jab hum slices ke bare mein jante hain, hum string literals ko sahi se samajh sakte hain:

```rust
let s = "Hello, world!";
```

Yahan `s` ka type `&str` hai: yeh us binary ke specific point ko point karne wala ek slice hai. Isliye string literals immutable hain; `&str` ek immutable reference hai.

-----

#### `String Slices as Parameters` (Parameters ke roop mein String Slices)

Yeh jankar ki aap literals aur `String` values ke slices le sakte hain, humein `first_word` par ek aur sudhar karne ka mauka milta hai, aur woh hai iska signature:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Ek zyada experienced Rustacean Listing 4-9 mein dikhaya gaya signature likhega, kyunki yeh humein `String` values aur `&str` values dono par ek hi function ka upyog karne ki permission deta hai.

```rust,ignore
fn first_word(s: &str) -> &str {
```

\<span class="caption"\>Listing 4-9: `s` parameter ke type ke liye string slice ka upyog karke `first_word` function ko behtar banana\</span\>

Agar hamare paas ek string slice hai, toh hum use sidhe pass kar sakte hain. Agar hamare paas ek `String` hai, toh hum poore `String` ka ek slice pass kar sakte hain. Ek function ko `String` ke reference ke bajaye string slice lene ke liye define karna hamare API ko aur bhi general aur useful banata hai, bina kisi functionality ko khoye:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# fn first_word(s: &str) -> &str {
#     let bytes = s.as_bytes();
#
#     for (i, &item) in bytes.iter().enumerate() {
#         if item == b' ' {
#             return &s[0..i];
#         }
#     }
#
#     &s[..]
# }
fn main() {
    let my_string = String::from("hello world");

    // first_word `String`s ke slices par kaam karta hai
    let word = first_word(&my_string[..]);

    let my_string_literal = "hello world";

    // first_word string literals ke slices par kaam karta hai
    let word = first_word(&my_string_literal[..]);

    // Kyunki string literals pehle se hi string slices hain,
    // yeh bhi kaam karta hai, bina slice syntax ke!
    let word = first_word(my_string_literal);
}
```

-----

### `Other Slices` (Doosre Slices)

String slices, jaisa ki aap soch sakte hain, strings ke liye specific hain. Lekin ek zyada general slice type bhi hai. Is array ko consider karein:

```rust
let a = [1, 2, 3, 4, 5];
```

Jis tarah hum ek string ke hisse ko refer karna chahte hain, hum ek array ke hisse ko bhi refer karna chahte hain. Hum aisa is tarah karenge:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];
```

Is slice ka type `&[i32]` hai. Yeh usi tarah kaam karta hai jaise string slices karte hain, pehle element ke reference aur ek length ko store karke. Aap is tarah ke slice ka upyog sabhi tarah ke doosre collections ke liye karenge. Jab hum Chapter 8 mein vectors ke bare mein baat karenge, tab hum in collections par detail mein charcha karenge.

-----

### Summary

Ownership, borrowing, aur slices ke concepts Rust programs mein compile time par memory safety ensure karte hain. Rust language aapko doosri systems programming languages ki tarah hi aapke memory usage par control deti hai, lekin data ke owner ka scope se bahar jane par us data ko automatically clean karna, iska matlab hai ki aapko is control ko pane ke liye extra code likhne aur debug karne ki zaroorat nahi hai.

Ownership Rust ke bahut se doosre parts ko kaise kaam karte hain, is par asar dalta hai, isliye hum is kitab ke baaki hisse mein in concepts par aur aage baat karenge. Chaliye Chapter 5 par chalte hain aur data ke pieces ko `struct` mein ek saath group karne ko dekhte hain.
