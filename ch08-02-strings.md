# Strings ke saath UTF-8 Encoded Text Store karna 

Humne strings ke baare mein **Chapter 4** mein baat ki thi, lekin ab hum inhe aur gehrai se dekhenge. Naye Rust programmers strings mein teen reasons ki wajah se commonly atak jaate hain:

1.  Rust ka possible errors ko expose karne ka nature.
2.  Strings ka ek complicated data structure hona jaisa ki aksar programmers sochte nahi hain.
3.  **UTF-8** encoding.

Jab aap doosri programming languages se aate hain, to ye factors milkar thode mushkil lag sakte hain.

Strings ko collections ke context mein samajhna useful hai, kyunki strings ko bytes ke collection ke roop mein implement kiya gaya hai. Jab in bytes ko text ke roop mein interpret kiya jaata hai, to ye bahut hi useful functionality provide karte hain. Is section mein hum **`String`** par hone wale operations ke baare mein baat karenge, jaise ki banana, update karna aur padhna. Hum ye bhi discuss karenge ki `String` dusre collections se alag kaise hai, khaaskar ki **`String` mein indexing** kitni complicated hai, kyunki insaan aur computer string data ko alag-alag tarah se dekhte hain.

-----

### String Kya Hai? 🤔

Sabse pehle, hum **string** term ka matlab define karenge. Rust ki core language mein sirf ek hi string type hai, jo ki **string slice `str`** hai aur ye aksar apne borrowed form **`&str`** mein dikhta hai. String literals, jaise ki `"hello"`, program ke binary output mein store hote hain aur isliye ye string slices hote hain.

**`String` type**, jo Rust ki standard library dwara provide kiya gaya hai (na ki core language mein), ek **growable (size badhne wala), mutable, owned, UTF-8 encoded string type** hai. Jab Rust programmers "strings" ka refer karte hain, to unka matlab `String` aur string slice `&str` dono se hota hai. Is section mein hum `String` ke baare mein zyaada baat karenge, lekin dono types Rust ki standard library mein bahut zyaada use hote hain aur dono hi **UTF-8 encoded** hain.

Rust ki standard library mein aur bhi kayi string types hain, jaise ki `OsString`, `OsStr`, `CString`, aur `CStr`. Ye sabhi **owned** aur **borrowed** variants ko refer karte hain, jaise ki `String` aur `str`.

-----

### Nayi String Banana 🆕

`Vec<T>` ki tarah, `String` ke saath bhi kayi operations available hain, jaise ki **`new` function** ka upyog karke ek string banana, jaisa **Listing 8-11** mein dikhaya gaya hai.

```rust
let mut s = String::new();
```

\<span class="caption"\>Listing 8-11: Ek naya, khali `String` banana\</span\>

Ye line `s` naam ki ek nayi khali string banati hai, jismein hum baad mein data load kar sakte hain. Aksar, humare paas kuch initial data hota hai jiske saath hum string shuru karna chahte hain. Uske liye, hum **`to_string` method** ka use karte hain, jo kisi bhi aise type par available hai jo `Display` trait ko implement karta hai, jaisa ki string literals karte hain. **Listing 8-12** mein do examples hain.

```rust
let data = "initial contents";

let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();
```

\<span class="caption"\>Listing 8-12: String literal se `String` banane ke liye `to_string` method ka upyog\</span\>

Is code se `"initial contents"` wali ek string banti hai.

Hum **`String::from` function** ka upyog karke bhi string literal se `String` bana sakte hain. **Listing 8-13** ka code **Listing 8-12** mein `to_string` ka use karne wale code ke barabar hai.

```rust
let s = String::from("initial contents");
```

\<span class="caption"\>Listing 8-13: String literal se `String` banane ke liye `String::from` function ka upyog\</span\>

Yaad rakhein ki strings **UTF-8 encoded** hain, isliye hum unme koi bhi sahi tarah se encoded data shamil kar sakte hain, jaisa **Listing 8-14** mein dikhaya gaya hai.

```rust
let hello = String::from("السلام عليكم");
let hello = String::from("Dobrý den");
let hello = String::from("Hello");
let hello = String::from("שָׁלוֹם");
let hello = String::from("नमस्ते");
let hello = String::from("こんにちは");
let hello = String::from("안녕하세요");
let hello = String::from("你好");
let hello = String::from("Olá");
let hello = String::from("Здравствуйте");
let hello = String::from("Hola");
```

\<span class="caption"\>Listing 8-14: Alag-alag languages mein greetings ko strings mein store karna\</span\>

Ye sabhi valid `String` values hain.

-----

### String ko Update karna ✍️

Ek `String` ka size badh sakta hai aur uske contents change ho sakte hain, bilkul `Vec<T>` ki tarah, agar aap usmein aur data `push` karte hain. Iske alawa, aap `+` operator ya `format!` macro ka upyog karke `String` values ko aasaani se jod sakte hain.

#### `push_str` aur `push` se String mein data jorana ➕

Hum **`push_str` method** ka use karke ek string slice ko jodkar `String` ko bada kar sakte hain, jaisa **Listing 8-15** mein dikhaya gaya hai.

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

\<span class="caption"\>Listing 8-15: `push_str` method ka upyog karke ek string slice ko `String` mein jorna\</span\>

Iske baad, `s` mein `foobar` hoga. `push_str` method ek string slice leta hai kyunki humein zaroori nahi ki parameter ki ownership leni ho. Jaise, **Listing 8-16** mein dikhaya gaya hai ki agar hum `s1` mein `s2` ke contents ko jodne ke baad `s2` ka upyog na kar paate to ye unfortunate hota.

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {}", s2);
```

\<span class="caption"\>Listing 8-16: Ek string slice ke contents ko `String` mein jodne ke baad uska upyog\</span\>

Agar `push_str` method `s2` ki ownership le leta, to hum last line par uski value print nahi kar paate. Lekin, ye code waise hi kaam karta hai jaisa hum expect karte hain\!

**`push` method** ek single character ko parameter ke roop mein leta hai aur use `String` mein add karta hai. **Listing 8-17** mein `push` method ka use karke `String` mein letter *l* add kiya gaya hai.

```rust
let mut s = String::from("lo");
s.push('l');
```

\<span class="caption"\>Listing 8-17: `push` ka use karke `String` value mein ek character add karna\</span\>

Is code ke baad, `s` mein `lol` hoga.

#### `+` Operator ya `format!` Macro se Concatenation 🔗

Aksar aap do existing strings ko jorana chahenge. Iska ek tareeka **`+` operator** ka upyog karna hai, jaisa **Listing 8-18** mein dikhaya gaya hai.

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

\<span class="caption"\>Listing 8-18: Do `String` values ko ek nayi `String` value mein jorne ke liye `+` operator ka upyog\</span\>

Is code ke baad, `s3` mein `"Hello, world!"` hoga. `s1` is addition ke baad valid nahi rehta hai aur humne `s2` ka reference use kiya hai, iska reason us method ke signature mein hai jo `+` operator ke use par call hota hai. `+` operator **`add` method** ka use karta hai, jiska signature kuch is tarah ka hota hai:

```rust,ignore
fn add(self, s: &str) -> String {
```

Yahan, `add` `self` ki ownership leta hai, kyunki `self` ke paas `&` nahi hai. Iska matlab hai ki **Listing 8-18** mein `s1` `add` call mein **move** ho jaayega aur uske baad valid nahi rahega.

Agar humein kayi strings ko concatenate karna ho, to `+` operator ka behavior unmanageable ho jaata hai:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

Aise mein, `s` `"tic-tac-toe"` hoga. Itne saare `+` aur `"` characters se samajhna mushkil ho jaata hai ki kya ho raha hai. Zyaada complicated string combining ke liye, hum **`format!` macro** ka use kar sakte hain:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

Ye code bhi `s` ko `"tic-tac-toe"` set karta hai. `format!` macro `println!` ki tarah hi kaam karta hai, lekin output ko screen par print karne ki bajaye, ye contents ke saath ek `String` return karta hai. `format!` ka upyog karne wala code padhna bahut aasan hai aur ye kisi bhi parameter ki ownership nahi leta.

-----

### Strings mein Indexing 🕵️‍♀️

Bahut si programming languages mein, strings mein individual characters ko index se access karna ek valid aur common operation hai. Lekin, agar aap Rust mein indexing syntax ka upyog karke `String` ke parts ko access karne ki koshish karte hain, to aapko **error** milegi. **Listing 8-19** mein invalid code ko dekhiye.

```rust,ignore
let s1 = String::from("hello");
let h = s1[0];
```

\<span class="caption"\>Listing 8-19: `String` ke saath indexing syntax ka upyog karne ki koshish\</span\>

Is code se ye error aayegi:

```text
error[E0277]: the trait bound `std::string::String: std::ops::Index<{integer}>` is not satisfied
 -->
  |
3 |     let h = s1[0];
  |             ^^^^^ the type `std::string::String` cannot be indexed by `{integer}`
  |
  = help: the trait `std::ops::Index<{integer}>` is not implemented for `std::string::String`
```

Error aur note batate hain ki Rust strings **indexing support nahi karte**. Lekin kyon? Iska jawab dene ke liye, humein samajhna hoga ki Rust strings ko memory mein kaise store karta hai.

#### Internal Representation 💾

Ek `String` **`Vec<u8>`** ke upar ek wrapper hai. Aaiye hamare **Listing 8-14** se kuch properly encoded UTF-8 example strings ko dekhte hain. Pehle, ye:

```rust
let len = String::from("Hola").len();
```

Is case mein, `len` **4** hogi, iska matlab hai ki "Hola" string ko store karne wala vector 4 bytes lamba hai. In mein se har ek letter UTF-8 mein encode hone par 1 byte leta hai. Lekin is line ke baare mein kya?

```rust
let len = String::from("Здравствуйте").len();
```

Aap soch sakte hain ki string 12 lamba hai. Lekin, Rust ka jawab **24** hai: "Здравствуйте" ko UTF-8 mein encode karne ke liye itne bytes lagte hain, kyunki har Unicode scalar value 2 bytes storage leta hai. Isliye, string ke bytes mein index hamesha ek valid Unicode scalar value se correlate nahi karega. Isko demonstrate karne ke liye, is invalid Rust code ko dekhein:

```rust,ignore
let hello = "Здравствуйте";
let answer = &hello[0];
```

`answer` ki value kya honi chahiye? Kya ye `З` honi chahiye? UTF-8 mein encoded, `З` ka pehla byte `208` hai aur doosra `151` hai, to `answer` asal mein `208` hona chahiye, lekin `208` khud mein ek valid character nahi hai. Rust is code ko compile hi nahi hone deta aur development process mein hi early misunderstandings ko rokta hai.

#### Bytes, Scalar Values aur Grapheme Clusters\! Oh My\! 🤯

UTF-8 ke baare mein ek aur point ye hai ki Rust ke perspective se strings ko dekhne ke teen relevant tareeke hain: **bytes**, **scalar values**, aur **grapheme clusters** (jo insaan ki bhasha mein "letters" ke sabse kareeb hain).

Agar hum Hindi word **"नमस्ते"** ko Devanagari script mein dekhein, to ye `u8` values ke vector ke roop mein store hota hai jo is tarah dikhta hai:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Ye 18 bytes hain. Agar hum unhein Unicode scalar values ke roop mein dekhein, jo Rust ka `char` type hai, to wo bytes is tarah dikhte hain:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Yahan cheh `char` values hain, lekin chautha aur chatha letter nahi hain: wo diacritics hain jo akele mein koi matlab nahi rakhte. Finally, agar hum unhein grapheme clusters ke roop mein dekhein, to humein wo milega jise insaan Hindi word banane wale chaar letters kahenge:

```text
["न", "म", "स्", "ते"]
```

Rust raw string data ko interpret karne ke alag-alag tareeke provide karta hai, taaki har program ko apni zaroorat ke hisaab se interpretation chunne ka mauka mile.

-----

### Strings ko Slice karna ✂️

String ko index karna aksar ek bura idea hai kyunki ye clear nahi hai ki string-indexing operation ka return type kya hona chahiye: ek byte value, ek character, ek grapheme cluster, ya ek string slice. Isliye, Rust aapse zyaada specific hone ko kehta hai agar aapko sach mein indices ka upyog karke string slices banane ki zaroorat hai. Aap **`[]` ka upyog ek single number ki bajaye range** ke saath kar sakte hain:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Yahan, `s` ek `&str` hoga jismein string ke pehle 4 bytes honge. `s` `"Зд"` hoga.

Agar hum `&hello[0..1]` ka use karte to kya hota? Jawab: Rust run time par panic ho jaata, usi tarah jaise vector mein ek invalid index access kiya jaata hai:

```text
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', src/libcore/str/mod.rs:2188:4
```

-----

### Strings par Iterate karne ke Methods 🚶‍♂️

Khushkismati se, aap string ke elements ko doosre tareekon se access kar sakte hain.

Agar aapko individual Unicode scalar values par operations perform karne ki zaroorat hai, to sabse accha tareeka **`chars` method** ka upyog karna hai. "नमस्ते" par `chars` ko call karne se cheh `char` type ki values alag ho jaati hain aur return hoti hain, aur aap har element ko access karne ke liye result par iterate kar sakte hain:

```rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

Ye code ye print karega:

```text
न
म
स
्
त
े
```

**`bytes` method** har raw byte ko return karta hai, jo aapke domain ke liye appropriate ho sakta hai:

```rust
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

Ye code wo 18 bytes print karega jo is `String` ko banate hain:

```text
224
164
// --snip--
165
135
```

Lekin yaad rakhein ki valid Unicode scalar values ek se zyaada byte se bane ho sakte hain.

-----

### Strings Itne Simple Nahi Hain 😅

Strings complicated hain. Alag-alag programming languages is complexity ko programmer ko alag-alag tareeke se present karte hain. Rust ne `String` data ki sahi handling ko default behavior banane ka decision liya hai, jiska matlab hai ki programmers ko **UTF-8 data** ko shuruat se hi handle karne par zyaada dhyan dena padta hai. Ye trade-off strings ki zyaada complexity ko expose karta hai, lekin ye aapko non-ASCII characters se judi errors ko development ke baad mein handle karne se rokta hai.
