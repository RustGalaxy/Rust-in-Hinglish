## Heap par Data Point Karne ke Liye `Box<T>` ka Upyog

Sabse seedha-saadha smart pointer ek **box** hai, jiska type `Box<T>` likha jaata hai. Boxes aapko `stack` ke bajaye `heap` par data store karne dete hain. `stack` par sirf `heap` ke data ka pointer rehta hai. Stack aur heap ke beech ka fark samajhne ke liye Chapter 4 ko dobara dekhein.

-----

Boxes mein data ko stack ke bajaye heap par store karne ke alawa, koi performance overhead nahi hota. Lekin unke paas bahut saari extra capabilities bhi nahi hoti hain. Aap unhe aksar in situations mein istemaal karenge:

  * Jab aapke paas ek aisa type ho jiska **size compile time par pata nahi lagaya ja sakta**, aur aap us type ki value ko ek aise context mein use karna chahte hain jahan ek exact size ki zaroorat hoti hai.
  * Jab aapke paas **bahut saara data** ho aur aap ownership transfer karna chahte hain, lekin yeh sunishchit karna chahte hain ki data copy na ho.
  * Jab aap ek value ko **own karna chahte hain** aur aapko sirf is baat se matlab hai ki woh ek aisi type hai jo ek particular trait ko implement karti hai, na ki kisi specific type ki ho.

Hum pehli situation ko "Enabling Recursive Types with Boxes" section mein demonstrate karenge. Doosre case mein, bade data ki ownership transfer karne mein lamba samay lag sakta hai kyunki data stack par copy hota hai. Performance behtar banane ke liye, hum us bade data ko heap par ek box mein store kar sakte hain. Phir, stack par sirf chhota sa pointer data copy hota hai, jabki data heap par ek hi jagah rehta hai. Teesra case **trait object** kehlata hai, jise Chapter 17 mein विस्तार se samjhaya gaya hai.

### `Box<T>` ka Upyog Heap par Data Store Karne ke Liye

Is use case par charcha karne se pehle, hum syntax aur `Box<T>` mein store ki gayi values ke saath interact karna dekhenge.

Neeche diya gaya code dikhata hai ki ek `i32` value ko heap par store karne ke liye box ka upyog kaise karein:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

\<span class="caption"\>Listing 15-1: Ek box ka upyog karke `i32` value ko heap par store karna\</span\>

Hum variable `b` ko ek `Box` ki value dete hain jo `5` ko point karta hai, aur yeh `5` heap par allocate hota hai. Yeh program `b = 5` print karega; is case mein, hum box ke andar data ko waise hi access kar sakte hain jaise woh stack par hota. Kisi bhi owned value ki tarah, jab ek box scope se bahar jaata hai (jaise `b` `main` ke end mein), woh deallocate ho jaayega. Deallocation box (jo stack par stored hai) aur uske data (jo heap par stored hai) dono ke liye hota hai.

Ek single value ko heap par rakhna bahut upyogi nahi hai, isliye aap is tarah se boxes ka istemaal aksar nahi karenge.

-----

### Boxes se Recursive Types ko Enable Karna

Compile time par, Rust ko yeh jaanna zaroori hota hai ki ek type kitni jagah leti hai. Ek type jiska size compile time par nahi jaana ja sakta, woh hai ek **recursive type**, jahan ek value apne hisse ke roop mein usi type ki doosri value rakh sakti hai. Kyunki values ki yeh nesting anant (infinitely) tak jaa sakti hai, Rust ko nahi pata ki ek recursive type ki value ke liye kitni jagah chahiye. Halaanki, **boxes ka size hamesha pata hota hai**, isliye ek recursive type definition mein box daal kar, aap recursive types bana sakte hain.

Chaliye ek example ke taur par **cons list** ko explore karte hain, jo functional programming languages mein ek aam data type hai.

#### Cons List ke Baare Mein Aur Jaankari

Ek **cons list** Lisp programming language se aane wala ek data structure hai. Har item mein do elements hote hain: current item ki value aur agla item. List ka aakhri item, jise `Nil` kehte hain, sirf ek value rakhta hai aur uske aage koi item nahi hota.

Rust mein cons list ka aam taur par istemaal nahi hota hai. Zyadaatar jab aapko items ki list chahiye hoti hai, to `Vec<T>` ek behtar vikalp hai. Lekin cons list ek aasan example hai yeh samajhne ke liye ki boxes kaise recursive data types ko define karne mein madad karte hain.

Neeche ek cons list ke liye `enum` ki definition di gayi hai. Dhyan dein ki yeh code abhi compile nahi hoga kyunki `List` type ka size pata nahi hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
enum List {
    Cons(i32, List),
    Nil,
}
```

\<span class="caption"\>Listing 15-2: `i32` values ki cons list represent karne ki pehli koshish\</span\>

Agar hum is code ko compile karne ki koshish karte hain, to humein yeh error milta hai:

```text
error[E0072]: recursive type `List` has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^ recursive type has infinite size
2 |     Cons(i32, List),
  |               ----- recursive without indirection
  |
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to
  make `List` representable
```

\<span class="caption"\>Listing 15-4: Ek recursive enum ko define karne ki koshish karte waqt milne wala error\</span\>

Error batata hai ki is type ka "infinite size" hai. Iska kaaran yeh hai ki humne `List` ko ek aise variant (`Cons`) ke saath define kiya hai jo seedhe apne andar khud ka hi ek aur value (`List`) rakhta hai. Iske parinaamswaroop, Rust yeh pata nahi laga sakta ki ek `List` value ko store karne ke liye kitni jagah chahiye. Compiler `Cons` variant ko dekhta hai, jismein ek `i32` aur ek `List` hai. `List` ka size pata karne ke liye, woh phir se `Cons` variant ko dekhta hai, jismein ek `i32` aur ek `List` hai, aur yeh prakriya anant tak chalti rehti hai.

#### `Box<T>` ka Upyog Karke ek Known Size wala Recursive Type Banana

Compiler ke error mein ek madadgaar sujhav bhi shaamil hai: "insert indirection". **Indirection** ka matlab hai ki value ko seedhe store karne ke bajaye, hum data structure ko badal kar us value ka pointer store karenge.

Kyunki `Box<T>` ek pointer hai, Rust ko hamesha pata hota hai ki ek `Box<T>` ko kitni jagah chahiye: ek pointer ka size is baat par nirbhar nahi karta ki woh kitne data ki taraf point kar raha hai. Iska matlab hai ki hum `Cons` variant ke andar seedhe `List` value ke bajaye ek `Box<T>` daal sakte hain. `Box<T>` agle `List` value ki taraf point karega jo `Cons` variant ke andar hone ke bajaye heap par hoga.

Hum `List` enum ki definition ko neeche diye gaye code mein badal sakte hain, jo compile ho jaayega:

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```

\<span class="caption"\>Listing 15-5: `List` ki definition jo `Box<T>` ka upyog karti hai taaki uska size pata ho\</span\>

Ab `Cons` variant ko ek `i32` ke size aur box ke pointer data ke size ke barabar jagah chahiye hogi. `Nil` variant koi value store nahi karta, isliye use kam jagah chahiye. Ab hum jaante hain ki koi bhi `List` value ek `i32` ke size aur ek box ke pointer ke size ke barabar jagah legi. Ek box ka upyog karke, humne anant, recursive chain ko tod diya hai, isliye ab compiler `List` value ko store karne ke liye zaroori size ka pata laga sakta hai. .

Boxes sirf indirection aur heap allocation pradaan karte hain; unke paas koi aur special capabilities nahi hoti. Unka koi performance overhead bhi nahi hota, isliye woh cons list jaise मामलों mein upyogi ho sakte hain jahan indirection hi hamari zaroorat hai.

`Box<T>` type ek smart pointer hai kyunki yeh `Deref` trait ko implement karta hai, jo `Box<T>` values ko references ki tarah treat karne deta hai. Jab ek `Box<T>` value scope se bahar jaati hai, to heap data jise box point kar raha hota hai, `Drop` trait ke implementation ke kaaran clean up ho jaata hai. Chaliye in do traits ko aur detail mein dekhte hain.
