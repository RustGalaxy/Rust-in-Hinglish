# Hash Maps mein Keys aur Unki Values ko Store karna 🗺️

Hamara last common collection **hash map** hai. `HashMap<K, V>` type **keys** (`K` type ke) ko **values** (`V` type ke) se map karke store karta hai. Ye ek **hashing function** ke zariye hota hai, jo ye decide karta hai ki keys aur values ko memory mein kaise rakha jaaye. Bahut si programming languages is tarah ke data structure ko support karti hain, lekin unke alag-alag naam hote hain, jaise ki hash, map, object, hash table, ya associative array.

Hash maps tab useful hote hain jab aap data ko index se nahi, balki kisi bhi type ki key se look up karna chahte hain. Jaise, ek game mein aap har team ka score ek hash map mein track kar sakte hain, jahan har key team ka naam ho aur values unke scores. Team ka naam dekar aap uska score nikal sakte hain.

Is section mein hum hash maps ke basic API ko cover karenge, lekin standard library mein `HashMap<K, V>` par bahut saare aur bhi useful functions hain. Zyaada jaankari ke liye hamesha standard library documentation dekhein.

-----

### Naya Hash Map Banana 🏗️

Aap `new` se ek khali hash map bana sakte hain aur `insert` se elements add kar sakte hain. **Listing 8-20** mein, hum do teams, Blue aur Yellow, ke scores track kar rahe hain. Blue team 10 points se shuru karti hai, aur Yellow team 50 se.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

\<span class="caption"\>Listing 8-20: Ek naya hash map banana aur usmein keys aur values insert karna\</span\>

Gaur karein ki humein pehle standard library ke collections portion se `HashMap` ko `use` karna padta hai. Hamare teen common collections mein se, ye sabse kam use hota hai, isliye ye prelude mein automatically scope mein nahi aata hai. Hash maps mein standard library se kam support milta hai; jaise ki unhein construct karne ke liye koi built-in macro nahi hai.

Vectors ki tarah, hash maps apna data heap par store karte hain. Is `HashMap` mein keys `String` type ki hain aur values `i32` type ki hain. Vectors ki tarah, hash maps **homogeneous** hote hain: saari keys ka type same hona chahiye, aur saari values ka type bhi same hona chahiye.

Hash map banane ka ek aur tareeka hai **vector of tuples** par `collect` method ka upyog karna, jahan har tuple mein ek key aur uski value hoti hai. `collect` method data ko kayi collection types mein ikattha karta hai, jismein `HashMap` bhi shamil hai. Jaise, agar hamare paas team names aur initial scores do alag-alag vectors mein hote, to hum `zip` method ka upyog karke tuples ka ek vector bana sakte the jahan "Blue" ko 10 ke saath joda jaata, aur aise hi aage. Fir hum `collect` method ka upyog karke us vector of tuples ko ek hash map mein convert kar sakte the, jaisa **Listing 8-21** mein dikhaya gaya hai.

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

\<span class="caption"\>Listing 8-21: Teams aur scores ki list se ek hash map banana\</span\>

Yahan `HashMap<_, _>` type annotation zaroori hai kyunki `collect` kayi alag-alag data structures mein ho sakta hai aur Rust ko pata nahi chalta ki aap kaunsa chahte hain. Key aur value types ke parameters ke liye, hum underscores ka use karte hain, aur Rust vectors ke data ke types ke aadhar par hash map ke types ko infer kar sakta hai.

-----

### Hash Maps aur Ownership 🤝

Jo types **`Copy` trait** ko implement karte hain, jaise `i32`, unki values hash map mein copy ho jaati hain. **`String`** jaise owned values ke liye, values **move** ho jaati hain aur hash map un values ka owner ban jaata hai, jaisa **Listing 8-22** mein dikhaya gaya hai.

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value ab invalid hain, unhein use karke dekhiye ki compiler kya error deta hai!
```

\<span class="caption"\>Listing 8-22: Dikhana ki keys aur values insert hone ke baad hash map unka owner ban jaata hai\</span\>

Hum `field_name` aur `field_value` variables ko `insert` call ke baad use nahi kar sakte, kyunki wo hash map mein move ho chuke hain.

Agar hum references ko hash map mein insert karte hain, to values move nahi hongi. जिन values ko references point kar rahe hain wo tab tak valid hone chahiye jab tak hash map valid hai.

-----

### Hash Map mein Values ko Access karna 🔑

Hum `get` method ko key dekar hash map se value nikal sakte hain, jaisa **Listing 8-23** mein dikhaya gaya hai.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

\<span class="caption"\>Listing 8-23: Hash map mein stored Blue team ke score ko access karna\</span\>

Yahan, `score` ki value Blue team se associated hogi, aur result **`Some(&10)`** hoga. Result `Some` mein wrap kiya gaya hai, kyunki `get` ek `Option<&V>` return karta hai; agar hash map mein us key ke liye koi value nahi hai, to `get` `None` return karega.

Hum `for` loop ka upyog karke hash map ke har key/value pair par iterate kar sakte hain:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

Ye code har pair ko ek arbitrary order mein print karega.

-----

### Hash Map ko Update karna 🔄

Har key ke saath ek samay mein sirf ek hi value associated ho sakti hai. Jab aap hash map mein data change karna chahte hain, to aapko decide karna hota hai ki jab ek key ke paas already ek value hai, to us case ko kaise handle karna hai. Aap purani value ko nayi se replace kar sakte hain, ya purani value rakhkar sirf tabhi nayi value add kar sakte hain jab key ke paas pehle se koi value nahi ho, ya purani aur nayi value ko combine kar sakte hain. Aaiye inmein se har ek ko dekhte hain\!

#### Value ko Overwrite karna 📝

Agar hum ek hash map mein ek key aur value insert karte hain aur fir usi key ko alag value ke saath insert karte hain, to us key se associated value replace ho jaayegi. **Listing 8-24** mein `insert` ko do baar call karne par bhi, hash map mein sirf ek key/value pair hoga.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

\<span class="caption"\>Listing 8-24: Ek particular key ke saath stored value ko replace karna\</span\>

Ye code `{"Blue": 25}` print karega. Original value `10` overwrite ho chuki hai.

#### Value sirf tab Insert karna jab Key ke paas koi Value na ho ✨

Ye common hai ki ek particular key ke paas value hai ya nahi ye check karna, aur agar nahi hai, to uske liye ek value insert karna. Hash maps mein iske liye ek special API hai jise `entry` kehte hain, jo check karne wali key ko parameter ke roop mein leta hai. `entry` method ka return value ek `Entry` naam ka enum hota hai jo ek aisi value ko represent karta hai jo shayad exist ho ya nahi. **Listing 8-25** mein, hum `entry` API ka upyog kar rahe hain.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

\<span class="caption"\>Listing 8-25: `entry` method ka upyog karke value sirf tab insert karna jab key ke paas pehle se koi value na ho\</span\>

`Entry` par `or_insert` method ko is tarah se define kiya gaya hai ki wo corresponding `Entry` key ke liye mutable reference return kare agar key exist karti hai, aur agar nahi, to parameter ko nayi value ke roop mein insert karta hai aur nayi value ka mutable reference return karta hai.

**Listing 8-25** ka code run karne par `{"Yellow": 50, "Blue": 10}` print hoga.

#### Purani Value ke aadhar par Value ko Update karna 📈

Hash maps ka ek aur common use case hai key ki value ko look up karna aur fir use purani value ke aadhar par update karna. **Listing 8-26** mein, code count karta hai ki text mein har word kitni baar aata hai.

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

\<span class="caption"\>Listing 8-26: Words aur unke counts ko store karne wale hash map ka upyog karke words ki occurrences count karna\</span\>

Ye code `{"world": 2, "hello": 1, "wonderful": 1}` print karega. `or_insert` method asal mein value ke liye ek **mutable reference (`&mut V`)** return karta hai. Yahan hum us mutable reference ko `count` variable mein store karte hain, isliye us value ko assign karne ke liye, humein pehle asterisk (`*`) ka use karke `count` ko **dereference** karna padta hai.

-----

### Hashing Functions 🛡️

By default, `HashMap` ek **cryptographically secure hashing function** ka use karta hai jo Denial of Service (DoS) attacks ke khilaf resistance provide kar sakta hai. Ye sabse fast hashing algorithm nahi hai, lekin behtar security ke liye performance mein thodi kami ka trade-off worth it hai. Agar aapko lagta hai ki default hash function bahut slow hai, to aap ek alag **hasher** specify kar sakte hain. Hasher ek aisa type hai jo `BuildHasher` trait ko implement karta hai.

-----

### Summary 📋

Vectors, strings, aur hash maps aapko programs mein data store, access, aur modify karne ke liye bahut saari functionality denge. Ab aap kuch exercises solve kar sakte hain:

  * Integers ki list di gayi hai, vector ka upyog karein aur list ka **mean**, **median**, aur **mode** return karein. (**Hash map** mode ke liye helpful hoga).
  * Strings ko **pig latin** mein convert karein. Jaise, “first” “irst-fay” ban jaata hai. Vowels se shuru hone wale words “apple” “apple-hay” ban jaate hain. **UTF-8 encoding** ka dhyan rakhein.
  * Ek hash map aur vectors ka upyog karke, ek text interface banao jisse user ek company mein employees ko departments mein add kar sake. Fir user ko ek department ya poori company ke logon ki alphabetically sorted list nikalne do.

In exercises ke liye vectors, strings aur hash maps ke methods standard library API documentation mein diye gaye hain.

Ab hum aur complicated programs mein ja rahe hain jahan operations fail ho sakte hain, isliye error handling par baat karne ka ye perfect time hai. Hum agle chapter mein wahi karenge\!
