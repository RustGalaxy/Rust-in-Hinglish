# Generic Types, Traits, and Lifetimes

Har programming language mein concepts ke **duplication** ko effectively handle karne ke liye tools hote hain. Rust mein, aisa hi ek tool **generics** hai. Generics **concrete types** ya doosri properties ke liye **abstract stand-ins** hote hain. Jab hum code likh rahe hote hain, to hum generics ka behavior ya unka doosre generics se relation express kar sakte hain, bina ye jaane ki compile ya run hone par unki jagah kya hoga.

Bilkul waise hi jaise ek function alag-alag concrete values par ek hi code ko run karne ke liye anjaan values ke saath **parameters** leta hai, functions ek concrete type jaise `i32` ya `String` ke bajaye kisi generic type ke parameters le sakte hain. Humne already **Chapter 6** mein `Option<T>`, **Chapter 8** mein `Vec<T>` aur `HashMap<K, V>`, aur **Chapter 9** mein `Result<T, E>` ke saath generics ka use kiya hai. Is chapter mein, aap seekhenge ki kaise aap apne khud ke types, functions aur methods ko generics ke saath define kar sakte hain\!

Sabse pehle, hum code duplication ko ek function mein extract karke hatana review karenge. Phir, hum isi technique ka upyog karke do functions se ek generic function banayenge jo sirf apne parameter types mein alag hain. Hum struct aur enum definitions mein generic types ka upyog karna bhi samjhayenge.

-----

### Function ko Extract karke Duplication hatana ✂️

Generics ke syntax mein dive karne se pehle, aaiye pehle dekhte hain ki non-generic types ke duplication ko ek function mein extract karke kaise hataya jaata hai. Phir hum isi technique ko ek generic function ko extract karne ke liye apply karenge\! Jis tarah aap duplicate code ko pehchan kar ek function mein extract karte hain, usi tarah aap aise duplicate code ko pehchanana shuru karenge jo generics ka use kar sakta hai.

Ek chhota program dekhte hain jo ek list mein sabse bada number dhoondhta hai, jaisa **Listing 10-1** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
#  assert_eq!(largest, 100);
}
```

\<span class="caption"\>Listing 10-1: Numbers ki list mein sabse bada number dhoondhne ka code\</span\>

Ye code `number_list` variable mein integers ki ek list store karta hai aur list ka pehla number `largest` naam ke variable mein rakhta hai. Phir ye list ke saare numbers par iterate karta hai, aur agar current number `largest` mein store kiye gaye number se bada hai, to ye us variable mein us number ko replace kar deta hai. Saare numbers ko consider karne ke baad, `largest` mein sabse bada number hona chahiye, jo is case mein **100** hai.

Agar humein do alag-alag lists mein sabse bada number dhoondhna ho, to hum **Listing 10-1** ke code ko duplicate kar sakte hain aur program mein do alag-alag jagahon par usi logic ka upyog kar sakte hain, jaisa **Listing 10-2** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```

\<span class="caption"\>Listing 10-2: *Do* lists mein sabse bada number dhoondhne ka code\</span\>

Bhale hi ye code kaam karta hai, lekin code ko **duplicate** karna **tedious** aur **error prone** hai. Jab hum isko change karna chahte hain, to humein kayi jagahon par code ko update karna padta hai.

Is duplication ko hatane ke liye, hum ek **abstraction** bana sakte hain. Hum ek function define karenge jo usko diye gaye integers ki kisi bhi list par operate karega. Ye solution hamare code ko zyaada clear banata hai aur humein list mein sabse bada number dhoondhne ke concept ko abstract tareeke se express karne deta hai.

**Listing 10-3** mein, humne sabse bada number dhoondhne wale code ko `largest` naam ke function mein extract kar liya hai. **Listing 10-1** ke code ke ulat, jo sirf ek hi list mein sabse bada number dhoondh sakta hai, ye program do alag-alag lists mein sabse bada number dhoondh sakta hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn largest(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 100);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 6000);
}
```

\<span class="caption"\>Listing 10-3: Do lists mein sabse bada number dhoondhne ke liye abstracted code\</span\>

`largest` function mein `list` naam ka ek parameter hai, jo `i32` values ke kisi bhi concrete slice ko represent karta hai jise hum function mein pass kar sakte hain. Iske natije mein, jab hum function ko call karte hain, to code un specific values par run hota hai jo hum pass karte hain.

In sabka saaransh, yahan wo steps hain jo humne **Listing 10-2** ke code ko **Listing 10-3** mein change karne ke liye uthaye:

1.  **Duplicate code ko identify karein.**
2.  **Duplicate code ko function ki body mein extract karein** aur function signature mein us code ke inputs aur return values specify karein.
3.  **Duplicated code ke do instances ko function call se replace karein.**

Aage, hum generics ke saath isi steps ka upyog alag-alag tareekon se code duplication ko kam karne ke liye karenge. Jis tarah function body specific values ke bajaye ek **abstract `list`** par operate kar sakti hai, usi tarah generics code ko **abstract types** par operate karne dete hain.

For example, maan lijiye hamare paas do functions the: ek jo `i32` values ke slice mein sabse bada item dhoondhta hai aur doosra jo `char` values ke slice mein sabse bada item dhoondhta hai. Hum us duplication ko kaise hatayenge? Chaliye pata karte hain\!
