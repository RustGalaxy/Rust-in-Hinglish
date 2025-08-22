## Ownership kya hai?

Rust ka sabse unique feature **ownership** hai. Isse samjhana aasan hai, lekin iske poori language par gehre asar hain.

Har program ko chalte waqt computer ki memory ko manage karna hota hai. Kuch languages mein **garbage collection** hoti hai jo program chalte samay lagatar un memory ko dhundhti hai jo ab use nahi ho rahi hain; doosri languages mein, programmer ko explicitly memory allocate aur free karni padti hai. Rust ka approach alag hai: memory ko **ownership** ke system se manage kiya jata hai, jisme kuch rules hote hain jise compiler compile time par check karta hai. Ownership ke features se aapke program ki speed par koi asar nahi padta.

Kyunki **ownership** ek naya concept hai, iski aadat dalne mein thoda samay lagta hai. Achhi baat yeh hai ki jaise-jaise aap Rust aur ownership system ke rules ke saath experience gain karte hain, aap naturally hi safe aur efficient code likh payenge.

Jab aap ownership samajh jayenge, toh aapko Rust ke unique features ko samajhne mein ek solid foundation mil jayegi. Is chapter mein, hum kuch common data structure, **strings** ke examples ke saath ownership ko samjhenge.

> **The Stack and The Heap (Stack aur Heap)**
>
> Bahut si programming languages mein, aapko stack aur heap ke bare mein jyada nahi sochna padta. Lekin Rust jaisi **systems programming language** mein, yeh matter karta hai ki value stack par hai ya heap par, aur iska asar language ke behavior aur aapke decisions par padta hai. Is chapter mein aage ownership ko stack aur heap ke reference mein samjhaya jayega, isliye yahan ek chhota sa explanation hai.
>
> Stack aur heap dono hi memory ke hisse hain jo aapke code ko runtime par use karne ke liye available hote hain, lekin unki structure alag hoti hai. **Stack** values ko us order mein store karta hai jisme woh receive hoti hain aur unhe ulte order mein remove karta hai. Isse **last in, first out** kehte hain. Ek plate ke stack ke bare mein sochein: jab aap aur plates jodte hain, toh aap unhe pile ke upar rakhte hain, aur jab aapko ek plate chahiye hoti hai, toh aap sabse upar wali plate uthate hain. Beech ya bottom se plates ko add ya remove karna theek se kaam nahi karega\! Data add karne ko **pushing onto the stack** aur data remove karne ko **popping off the stack** kehte hain.
>
> Stack fast hai kyunki woh data ko ek particular way mein access karta hai: use hamesha upar se hi data milta hai, isliye use naye data ke liye ya data lene ke liye jagah search nahi karni padti. Ek aur property jo stack ko fast banati hai woh yeh hai ki stack par sabhi data ka ek fixed, known size hona chahiye.
>
> Jo data compile time par unknown size ka ho ya jiska size badal sakta ho, use **heap** par store kiya ja sakta hai. Heap kam organized hota hai: jab aap data ko heap par rakhte hain, toh aap kuch amount ki space maangte hain. Operating system heap mein kahin ek empty spot dhundhta hai jo kafi bada ho, use "in use" mark karta hai, aur ek **pointer** return karta hai, jo us location ka address hota hai. Is process ko **allocating on the heap** kehte hain, ya short mein **"allocating"** bhi kehte hain. Values ko stack par push karna allocating nahi mana jata hai. Kyunki pointer ka size fixed aur known hota hai, aap pointer ko stack par store kar sakte hain, lekin jab aapko actual data chahiye hota hai, toh aapko pointer ko follow karna padta hai.
>
> Imagine kijiye aap ek restaurant mein hain. Jab aap enter karte hain, aap apne group mein logon ki sankhya batate hain, aur staff ek khali table dhundhta hai jo sabko fit kar sake aur aapko wahan le jata hai. Agar aapke group mein koi late aata hai, toh woh puch sakte hain ki aap kahan baithe hain aapko dhundhne ke liye.
>
> Heap mein data ko access karna stack se slow hota hai kyunki aapko wahan tak pahunchne ke liye ek pointer ko follow karna padta hai. Modern processors tab fast hote hain jab woh memory mein kam jump karte hain. Analogy ko aage badhate hue, ek restaurant mein ek server ko kai tables se orders lene hain. Sabse efficient tarika yeh hai ki ek table se sare orders le liye jayein, phir doosre table par jaya jaye. Table A se ek order, phir table B se ek order, phir A se dobara ek, aur phir B se dobara ek lena, yeh ek bahut slow process hoga. Isi tarah, ek processor apna kaam tab behtar kar sakta hai agar woh data par kaam karta hai jo doosre data ke paas hai (jaise stack par hota hai) na ki bahut door (jaise heap par ho sakta hai). Heap par badi amount ki space allocate karna bhi time le sakta hai.
>
> Jab aapka code ek function ko call karta hai, toh function mein pass ki gayi values (jismein heap par data ke pointers shamil ho sakte hain) aur function ke local variables stack par push ho jate hain. Jab function khatam hota hai, toh woh values stack se pop ho jati hain.
>
> Yeh track karna ki code ke kaun se parts heap par kaun se data ka upyog kar rahe hain, heap par duplicate data ko minimize karna, aur unused data ko heap se clean karna taaki aapki space khatam na ho jaye, yeh sab woh problems hain jise ownership solve karta hai. Ek baar jab aap ownership samajh jayenge, toh aapko stack aur heap ke bare mein baar-baar nahi sochna padega, lekin yeh janna ki ownership heap data ko manage karne ke liye hai, yeh explain karne mein help kar sakta hai ki yeh is tarah se kyun kaam karta hai.

-----

### Ownership Rules (Ownership Ke Niyam)

Sabse pehle, chaliye ownership rules ko dekhte hain. In rules ko dhyan mein rakhein jab hum inko explain karne wale examples par kaam karenge:

  * Rust mein har value ka ek variable hota hai jise uska **owner** kehte hain.
  * Ek samay mein sirf ek owner ho sakta hai.
  * Jab owner **scope se bahar** chala jata hai, toh value ko **drop** kar diya jayega.

-----

### Variable Scope (Variable ka Scope)

Humne Chapter 2 mein Rust program ka ek example dekha tha. Ab jab hum basic syntax se aage badh gaye hain, toh hum examples mein saare `fn main() {` code ko shamil nahi karenge, isliye agar aap saath mein follow kar rahe hain, toh aapko neeche diye gaye examples ko ek `main` function ke andar manually rakhna padega. Natijan, hamare examples thode aur concise honge, jisse hum boilerplate code ke bajaye actual details par focus kar payenge.

Ownership ke pehle example ke roop mein, hum kuch variables ke **scope** ko dekhenge. Ek **scope** ek program ke andar woh range hoti hai jismein koi item valid hota hai. Chaliye maan lete hain ki hamare paas ek variable hai jo is tarah dikhta hai:

```rust
let s = "hello";
```

Variable **`s`** ek **string literal** ko refer karta hai, jahan string ki value hamare program ke text mein hardcoded hoti hai. Variable us point se valid hota hai jahan use declare kiya jata hai aur current scope ke end tak valid rehta hai. Listing 4-1 mein woh comments hain jo batate hain ki variable `s` kahan valid hai.

```rust
{                     // s yahan valid nahi hai, abhi tak declare nahi hua hai
    let s = "hello";  // s yahan se aage valid hai

    // do stuff with s
}                     // yeh scope ab khatam ho gaya hai, aur s ab valid nahi hai
```

\<span class="caption"\>Listing 4-1: Ek variable aur woh scope jismein woh valid hai\</span\>

Doosre shabdon mein, yahan do important time points hain:

  * Jab `s` **scope ke andar aata hai**, toh woh valid hota hai.
  * Yeh tab tak valid rehta hai jab tak woh **scope se bahar** nahi chala jata.

Is point par, scopes aur variables ke valid hone ke beech ka relationship doosri programming languages jaisa hi hai. Ab hum is samajh par **`String`** type ko introduce karke aage badhenge.

-----

### The `String` Type (String Type)

Ownership ke rules ko samjhane ke liye, humein ek data type chahiye jo Chapter 3 ke "Data Types" section mein cover kiye gaye types se zyada complex ho. Pehle cover kiye gaye types sabhi stack par store hote hain aur jab unka scope khatam hota hai toh stack se pop ho jate hain, lekin hum aise data ko dekhna chahte hain jo heap par store hota hai aur explore karna chahte hain ki Rust ko kab pata chalta hai ki us data ko clean karna hai.

Hum yahan `String` ko example ke roop mein use karenge aur `String` ke un hisson par dhyan denge jo ownership se related hain. Yeh aspects standard library dwara provide kiye gaye aur aapke dwara banaye gaye doosre complex data types par bhi apply hote hain. Hum Chapter 8 mein `String` ke bare mein aur detail mein discuss karenge.

Humne pehle hi string literals dekhe hain, jahan ek string value hamare program mein hardcoded hoti hai. String literals convenient hain, lekin har situation ke liye suitable nahi hain jismein hum text use karna chahte hain. Ek reason yeh hai ki woh **immutable** hote hain. Doosra yeh hai ki har string value compile time par humein pata nahi ho sakti: example ke liye, kya hoga agar hum user input lena chahte hain aur use store karna chahte hain? In situations ke liye, Rust ke paas ek doosra string type hai, **`String`**. Yeh type heap par allocate hota hai aur is tarah se text ki ek aisi amount store kar sakta hai jo compile time par humein pata nahi hoti. Aap `from` function ka use karke ek string literal se ek `String` bana sakte hain, jaise is tarah:

```rust
let s = String::from("hello");
```

Double colon (`::`) ek operator hai jo humein is particular `from` function ko `String` type ke andar namespace karne ki permission deta hai, na ki `string_from` jaisa koi naam use karke. Hum is syntax ke bare mein Chapter 5 ke "Method Syntax" section mein aur Chapter 7 mein modules ke saath namespacing ke bare mein baat karte waqt aur discuss karenge.

Is tarah ki string **mutable** ho sakti hai:

```rust
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() ek literal ko ek String mein append karta hai

println!("{}", s); // Yeh `hello, world!` print karega
```

Toh, yahan kya difference hai? `String` mutable kyun ho sakti hai jabki literals nahi ho sakte? Difference yeh hai ki yeh do types memory ko kaise deal karte hain.

-----

### Memory and Allocation (Memory aur Allocation)

String literal ke case mein, hum contents ko compile time par jante hain, isliye text sidhe final executable mein hardcoded hota hai. Isliye string literals fast aur efficient hote hain. Lekin yeh properties sirf string literal ki immutability se aati hain. Unfortunately, hum har text ke piece ke liye, jiska size compile time par unknown hai aur jiska size program chalte waqt badal sakta hai, binary mein memory ka blob nahi daal sakte.

`String` type ke saath, ek mutable, growable text ke piece ko support karne ke liye, humein heap par memory ki ek amount allocate karne ki zaroorat hoti hai, jo compile time par unknown hai, takki contents ko hold kar sake. Iska matlab hai:

  * Memory ko runtime par operating system se request kiya jana chahiye.
  * Jab hum apne `String` ka kaam poora kar lein, toh is memory ko operating system ko wapas karne ka ek tarika chahiye.

Pehla hissa hum karte hain: jab hum `String::from` ko call karte hain, toh iska implementation use zaroorat wali memory request karta hai. Yeh lagbhag programming languages mein universal hai.

Lekin, doosra hissa alag hai. **Garbage collector (GC)** wali languages mein, GC track rakhta hai aur un memory ko clean karta hai jo ab use nahi ho rahi hain, aur humein iske bare mein sochna nahi padta. GC ke bina, yeh hamari zimmedari hai ki hum identify karein ki memory ab use nahi ho rahi hai aur use explicitly wapas karne ke liye code call karein, jaisa humne use request karne ke liye kiya tha. Aisa sahi se karna historically ek mushkil programming problem rahi hai. Agar hum bhool jate hain, toh hum memory waste karenge. Agar hum bahut jaldi karte hain, toh hamare paas ek invalid variable hoga. Agar hum isse do baar karte hain, toh woh bhi ek bug hai. Humein exactly ek `allocate` ko exactly ek `free` ke saath pair karna chahiye.

Rust ek alag rasta apnata hai: memory automatically wapas kar di jati hai ek baar jab woh variable jiske paas ownership hai scope se bahar chala jata hai. Yahaan hamare scope example ka ek version hai jo string literal ke bajaye `String` ka upyog karta hai:

```rust
{
    let s = String::from("hello"); // s yahan se aage valid hai

    // do stuff with s
}                                  // yeh scope ab khatam ho gaya hai, aur s ab valid nahi hai
```

Ek natural point hai jahan hum apne `String` ko zaroorat wali memory operating system ko wapas kar sakte hain: jab `s` scope se bahar chala jata hai. Jab ek variable scope se bahar jata hai, toh Rust hamare liye ek special function call karta hai. Is function ko **`drop`** kehte hain, aur yahi woh jagah hai jahan `String` ka author memory wapas karne ke liye code dal sakta hai. Rust closing curly bracket par `drop` ko automatically call karta hai.

> Note: C++ mein, kisi item ke lifetime ke end par resources ko deallocate karne ke is pattern ko kabhi-kabhi **Resource Acquisition Is Initialization (RAII)** kehte hain. Rust mein `drop` function aapke liye familiar hoga agar aapne RAII patterns ka upyog kiya hai.

Is pattern ka Rust code likhne ke tarike par ek gehra asar padta hai. Abhi yeh simple lag sakta hai, lekin code ka behavior aur bhi complex situations mein unexpected ho sakta hai jab hum chahte hain ki multiple variables us data ka upyog karein jise humne heap par allocate kiya hai. Aaiye ab un situations ko explore karte hain.

-----

#### Ways Variables and Data Interact: Move (Variables aur Data ke Interact karne ke Tarike: Move)

Rust mein multiple variables ek hi data ke saath alag-alag tarikon se interact kar sakte hain. Chaliye Listing 4-2 mein ek integer ka upyog karke ek example dekhte hain.

```rust
let x = 5;
let y = x;
```

\<span class="caption"\>Listing 4-2: Variable `x` ki integer value ko `y` ko assign karna\</span\>

Hum shayad guess kar sakte hain ki yeh kya kar raha hai: "value `5` ko `x` se bind karo; phir `x` mein value ki ek copy banao aur use `y` se bind karo." Ab hamare paas do variables hain, `x` aur `y`, aur dono `5` ke barabar hain. Yahi ho raha hai, kyunki integers simple values hain jinka size known aur fixed hota hai, aur yeh do `5` values stack par push ho jati hain.

Ab chaliye `String` version ko dekhte hain:

```rust
let s1 = String::from("hello");
let s2 = s1;
```

Yeh pichhle code jaisa hi lagta hai, isliye hum maan sakte hain ki yeh usi tarah kaam karega: yaani, doosri line `s1` mein value ki ek copy banayegi aur use `s2` se bind karegi. Lekin aisa nahi hota.

Dekhein Figure 4-1 ko yeh dekhne ke liye ki `String` andar se kaise kaam karta hai. Ek `String` teen parts se banta hai, jo left mein dikhaye gaye hain: memory ka ek pointer jo string ke contents ko hold karta hai, ek length, aur ek capacity. Data ka yeh group stack par store hota hai. Right mein heap par memory hai jo contents ko hold karti hai.

\<span class="caption"\>Figure 4-1: `"hello"` value ko hold karne wale `String` ka memory mein representation jo `s1` se bind hai\</span\>

**Length** yeh hai ki `String` ke contents currently kitni memory, bytes mein, use kar rahe hain. **Capacity** memory ki total amount hai, bytes mein, jo `String` ne operating system se receive ki hai. Length aur capacity ke beech ka difference matter karta hai, lekin is context mein nahi, isliye abhi ke liye, capacity ko ignore karna theek hai.

Jab hum `s1` ko `s2` ko assign karte hain, toh `String` data copy ho jata hai, matlab hum pointer, length, aur capacity ko copy karte hain jo stack par hain. Hum heap par us data ko copy nahi karte jise pointer refer karta hai. Doosre shabdon mein, memory mein data ka representation Figure 4-2 jaisa dikhta hai.

\<span class="caption"\>Figure 4-2: `s2` variable ka memory mein representation jismein `s1` ke pointer, length, aur capacity ki copy hai\</span\>

Yeh representation Figure 4-3 jaisa **nahi** dikhta, jaisa memory dikhegi agar Rust heap data ko bhi copy karta. Agar Rust aisa karta, toh `s2 = s1` operation runtime performance ke terms mein bahut expensive ho sakta tha agar heap par data bada hota.

\<span class="caption"\>Figure 4-3: `s2 = s1` kya kar sakta hai iski ek aur possibility agar Rust heap data ko bhi copy karta\</span\>

Pehle, humne kaha tha ki jab ek variable scope se bahar jata hai, toh Rust automatically `drop` function ko call karta hai aur us variable ke liye heap memory ko clean karta hai. Lekin Figure 4-2 dikhata hai ki dono data pointers ek hi location ko point kar rahe hain. Yeh ek problem hai: jab `s2` aur `s1` scope se bahar jayenge, toh woh dono ek hi memory ko free karne ki koshish karenge. Ise **double free error** kehte hain aur yeh un memory safety bugs mein se ek hai jiska humne pehle zikr kiya tha. Memory ko do baar free karne se memory corruption ho sakta hai, jisse security vulnerabilities ho sakti hain.

Memory safety ko ensure karne ke liye, Rust mein is situation mein ek aur detail hoti hai. Allocated memory ko copy karne ki koshish karne ke bajaye, Rust `s1` ko ab valid nahi manta hai aur, isliye, jab `s1` scope se bahar jata hai toh use kuch bhi free karne ki zaroorat nahi padti. Dekhein ki jab aap `s2` banne ke baad `s1` ko use karne ki koshish karte hain toh kya hota hai; yeh kaam nahi karega:

```rust,ignore
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```

Aapko aisi error milegi kyunki Rust aapko invalidated reference ko use karne se rokta hai:

```text
error[E0382]: use of moved value: `s1`
 --> src/main.rs:5:28
  |
3 |     let s2 = s1;
  |         -- value moved here
4 | 
5 |     println!("{}, world!", s1);
  |                            ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`, which does
  not implement the `Copy` trait
```

Agar aapne doosri languages mein **shallow copy** aur **deep copy** terms suni hain, toh pointer, length, aur capacity ko copy karne ka concept, data ko copy kiye bina, shayad shallow copy jaisa lagega. Lekin kyunki Rust pehle variable ko bhi invalidate kar deta hai, isse shallow copy ke bajaye, **move** ke roop mein jana jata hai. Is example mein, hum kahenge ki `s1` ko `s2` mein **move** kar diya gaya tha. Toh jo actually hota hai woh Figure 4-4 mein dikhaya gaya hai.

\<span class="caption"\>Figure 4-4: `s1` ke invalidate hone ke baad memory mein representation\</span\>

Isse hamari problem solve ho jati hai\! Sirf `s2` valid hone ke saath, jab woh scope se bahar jata hai, toh woh akela memory ko free karega, aur hamara kaam ho gaya.

Iske alawa, isse ek design choice bhi imply hoti hai: Rust kabhi bhi aapke data ki "deep" copies automatically nahi banayega. Isliye, koi bhi **automatic** copying ko runtime performance ke terms mein sasta mana ja sakta hai.

-----

#### Ways Variables and Data Interact: Clone (Variables aur Data ke Interact karne ke Tarike: Clone)

Agar hum **`String`** ke heap data ki deep copy karna chahte hain, na ki sirf stack data ki, toh hum **`clone`** naam ka ek common method use kar sakte hain. Hum Chapter 5 mein method syntax discuss karenge, lekin kyunki methods bahut si programming languages mein ek common feature hain, aapne unhe shayad pehle hi dekha hoga.

Yahan `clone` method ka ek example hai:

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

Yeh bilkul sahi kaam karta hai aur explicitly woh behavior produce karta hai jo Figure 4-3 mein dikhaya gaya hai, jahan heap data copy ho jata hai.

Jab aap `clone` ka call dekhte hain, toh aap jante hain ki kuch arbitrary code execute ho raha hai aur woh code expensive ho sakta hai. Yeh ek visual indicator hai ki kuch alag ho raha hai.

-----

#### Stack-Only Data: Copy (Sirf Stack Data: Copy)

Ek aur baat hai jiske bare mein humne abhi tak baat nahi ki hai. Integer ka upyog karne wala yeh code, jiska ek hissa Listing 4-2 mein dikhaya gaya tha, kaam karta hai aur valid hai:

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

Lekin yeh code usse contradict karta hua lagta hai jo humne abhi seekha: hamare paas `clone` ka koi call nahi hai, lekin `x` abhi bhi valid hai aur `y` mein move nahi hua.

Reason yeh hai ki integers jaise types jinka compile time par size known hota hai, poori tarah se stack par store hote hain, isliye actual values ki copies banana fast hota hai. Iska matlab hai ki hamare paas `y` variable banane ke baad `x` ko valid rehne se rokne ka koi reason nahi hai. Doosre shabdon mein, yahan deep aur shallow copying mein koi difference nahi hai, isliye `clone` call karne se usual shallow copying se kuch alag nahi hoga aur hum isse chhod sakte hain.

Rust mein ek special annotation hai jise **`Copy` trait** kehte hain, jise hum integers jaise types par laga sakte hain jo stack par store hote hain (hum Chapter 10 mein traits ke bare mein aur baat karenge). Agar ek type mein `Copy` trait hai, toh assignment ke baad ek purana variable ab bhi use kiya ja sakta hai. Rust humein ek type par `Copy` trait annotate nahi karne dega agar type, ya uske kisi bhi parts ne `Drop` trait implement kiya ho. Agar value ke scope se bahar jane par type ko kuch special karne ki zaroorat hai aur hum us type mein `Copy` annotation jodte hain, toh humein compile-time error milegi. Apne type mein `Copy` annotation kaise jodein, iske bare mein janne ke liye Appendix C mein “Derivable Traits” dekhein.

Toh kaun se types `Copy` hain? Aap sure hone ke liye diye gaye type ke documentation ko check kar sakte hain, lekin ek general rule ke roop mein, simple scalar values ka koi bhi group `Copy` ho sakta hai, aur kuch bhi jise allocation ki zaroorat hai ya woh kisi resource ka roop hai, woh `Copy` nahi hota. Yahan kuch types hain jo `Copy` hain:

  * Sabhi integer types, jaise `u32`.
  * The Boolean type, `bool`, `true` aur `false` values ke saath.
  * Sabhi floating point types, jaise `f64`.
  * The character type, `char`.
  * Tuples, lekin sirf agar unmein aise types hon jo `Copy` bhi hon. Jaise, `(i32, i32)` `Copy` hai, lekin `(i32, String)` nahi hai.

-----

### Ownership and Functions (Ownership aur Functions)

Ek value ko ek function mein pass karne ke semantics ek value ko ek variable mein assign karne jaise hi hain. Ek variable ko ek function mein pass karne se move ya copy hoga, jaisa assignment karta hai. Listing 4-3 mein kuch annotations ke saath ek example hai jo batata hai ki variables kahan scope mein aate aur bahar jate hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let s = String::from("hello");  // s scope mein aata hai

    takes_ownership(s);             // s ki value function mein move ho jati hai...
                                    // ... aur isliye yahan ab valid nahi hai

    let x = 5;                      // x scope mein aata hai

    makes_copy(x);                  // x function mein move hoga,
                                    // lekin i32 Copy hai, isliye uske baad bhi
                                    // x ka upyog karna theek hai

} // Yahan, x scope se bahar chala jata hai, phir s. Lekin kyunki s ki value move ho gayi thi, kuch
  // special nahi hota hai.

fn takes_ownership(some_string: String) { // some_string scope mein aata hai
    println!("{}", some_string);
} // Yahan, some_string scope se bahar chala jata hai aur `drop` ko call kiya jata hai.
  // Backing memory free ho jati hai.

fn makes_copy(some_integer: i32) { // some_integer scope mein aata hai
    println!("{}", some_integer);
} // Yahan, some_integer scope se bahar chala jata hai. Kuch special nahi hota hai.
```

\<span class="caption"\>Listing 4-3: Ownership aur scope ke saath annotated functions\</span\>

Agar hum `takes_ownership` ke call ke baad `s` ko use karne ki koshish karte, toh Rust compile-time error throw karta. Yeh static checks humein galtiyon se bachate hain. `main` mein `s` aur `x` ka upyog karne wala code jodkar dekhein ki aap unka upyog kahan kar sakte hain aur kahan ownership rules aapko aisa karne se rokte hain.

-----

### Return Values and Scope (Return Values aur Scope)

Values ko return karna bhi ownership transfer kar sakta hai. Listing 4-4, Listing 4-3 jaisi hi annotations ke saath ek example hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let s1 = gives_ownership();        // gives_ownership apni return
                                       // value ko s1 mein move karta hai

    let s2 = String::from("hello");      // s2 scope mein aata hai

    let s3 = takes_and_gives_back(s2); // s2 ko
                                       // takes_and_gives_back mein move kiya jata hai, jo bhi
                                       // apni return value ko s3 mein move karta hai
} // Yahan, s3 scope se bahar chala jata hai aur drop ho jata hai. s2 scope se bahar jata hai
  // lekin move ho gaya tha, isliye kuch nahi hota. s1 scope se bahar jata hai aur drop ho jata hai.

fn gives_ownership() -> String {       // gives_ownership apni
                                       // return value ko us function mein move karega
                                       // jo use call karta hai

    let some_string = String::from("hello"); // some_string scope mein aata hai

    some_string                        // some_string return ho jata hai aur
                                       // calling function mein move ho jata hai
}

// takes_and_gives_back ek String lega aur ek return karega
fn takes_and_gives_back(a_string: String) -> String { // a_string scope mein aata hai

    a_string  // a_string return ho jata hai aur calling function mein move ho jata hai
}
```

\<span class="caption"\>Listing 4-4: Return values ka ownership transfer karna\</span\>

Ek variable ki ownership har baar same pattern follow karti hai: ek value ko doosre variable ko assign karna use move karta hai. Jab ek variable jismein heap par data shamil hai scope se bahar jata hai, toh value `drop` dwara clean kar di jayegi jab tak ki data ko kisi doosre variable ki ownership ke liye move nahi kar diya gaya ho.

Har function ke saath ownership lena aur phir ownership wapas karna thoda tedious ho sakta hai. Kya hoga agar hum ek function ko ek value use karne dena chahte hain lekin ownership lena nahi chahte? Yeh bahut annoying hai ki jo kuch bhi hum pass karte hain use wapas bhi pass karna padta hai agar hum use dobara use karna chahte hain, function ke body se kisi bhi data ke alawa jise hum return karna chahte hain.

Tuple ka upyog karke multiple values return karna possible hai, jaisa Listing 4-5 mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() ek String ki length return karta hai

    (s, length)
}
```

\<span class="caption"\>Listing 4-5: Parameters ka ownership return karna\</span\>

Lekin yeh bahut zyada formality ka kaam hai, jabki yeh ek aam concept hona chahiye. Khushkismati se, Rust mein is concept ke liye ek feature hai, jise **references** kehte hain.
