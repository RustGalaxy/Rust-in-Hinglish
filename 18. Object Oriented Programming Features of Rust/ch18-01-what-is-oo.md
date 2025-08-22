
## Object-Oriented Languages ki Visheshtayein

Programming community mein is baat par koi aam sahamati nahi hai ki kisi bhasha ko object-oriented maane jaane ke liye usmein kaun se features hone chahiye. Rust kai programming paradigms se prabhavit hai, jismein OOP bhi shaamil hai. Shayad, OOP bhashaon mein kuch aam visheshtayein (characteristics) hoti hain, jaise ki objects, encapsulation, aur inheritance. Aaiye dekhen ki inmein se har ek visheshta ka kya matlab hai aur kya Rust use support karta hai.

-----

### Objects Data aur Behavior Rakhte Hain

"Design Patterns: Elements of Reusable Object-Oriented Software" naamak kitaab, jise aam taur par "The Gang of Four" ki kitaab kaha jaata hai, OOP ko is tarah paribhashit karti hai:

> Object-oriented programs objects se bane hote hain. Ek **object** data aur us data par kaam karne wale procedures, dono ko package karta hai. Un procedures ko aam taur par **methods** ya **operations** kaha jaata hai.

Is paribhasha ka upyog karte hue, Rust object-oriented hai: `structs` aur `enums` mein data hota hai, aur `impl` blocks structs aur enums par methods pradan karte hain. Bhale hi methods waale structs aur enums ko *objects* nahi kaha jaata, ve wahi kaaryakshamata (functionality) pradan karte hain. 🧱

-----

### Encapsulation jo Implementation Details ko Chhupata Hai

OOP se juda ek aur pehlu **encapsulation** ka vichar hai, jiska matlab hai ki ek object ke implementation details us object ka upyog karne wale code ke liye uplabdh nahi hote. Isliye, ek object ke saath interact karne ka ekmatra tareeka uske public API ke madhyam se hai. Yah programmer ko object ke internals ko badalne aur refactor karne ki anumati deta hai, bina us code ko badalne ki zaroorat ke jo us object ka upyog karta hai.

Humne Chapter 7 mein encapsulation ko control karna seekha tha: hum `pub` keyword ka upyog karke yah tay kar sakte hain ki hamare code mein kaun se modules, types, functions, aur methods public hone chahiye, aur default roop se sab kuch private hota hai. Udaharan ke liye, hum ek `AveragedCollection` struct define kar sakte hain jismein integers ki ek list aur unka average maintain kiya jaata hai.

```rust
// Filename: src/lib.rs
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
```

*Listing 18-1: Ek `AveragedCollection` struct jo integers ki list aur collection mein items ka average maintain karta hai*

Struct ko `pub` mark kiya gaya hai taaki doosra code iska istemal kar sake, lekin struct ke andar ke fields private rehte hain. Yah is maamle mein mahatvapurna hai kyunki hum yah sunishchit karna chahte hain ki jab bhi list mein koi value jodi ya hatayi jaaye, to average bhi update ho. Hum yah struct par `add`, `remove`, aur `average` methods implement karke karte hain.

```rust
// Filename: src/lib.rs
impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            }
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

*Listing 18-2: `AveragedCollection` par public methods `add`, `remove`, aur `average` ka implementation*

Public methods `add`, `remove`, aur `average` hi ek `AveragedCollection` ke instance mein data ko access ya modify karne ke ekmatra tareeke hain. `list` aur `average` fields ko private rakha gaya hai taaki bahari code `list` field mein seedhe items jod ya hata na sake; anyatha, `average` field `list` ke badalne par out of sync ho sakta hai.

Kyunki humne `AveragedCollection` struct ke implementation details ko encapsulate kar diya hai, hum bhavishya mein aasani se data structure jaise pehluon ko badal sakte hain. Agar encapsulation ek bhasha ko object-oriented maane jaane ke liye ek aavashyak pehlu hai, to Rust is avashyakta ko poora karta hai. ✅

-----

### Inheritance ek Type System aur Code Sharing ke Roop Mein

**Inheritance** ek aisi mechanism hai jiske tahat ek object doosre object ki definition se tatvon (elements) ko inherit kar sakta hai, is prakaar bina unhe dobara define kiye parent object ke data aur behavior ko prapt kar leta hai.

Agar ek bhasha ko object-oriented hone ke liye inheritance ki avashyakta hai, to **Rust aisi bhasha nahi hai**. Aisa koi tareeka nahi hai jisse ek struct ko define kiya ja sake jo parent struct ke fields aur method implementations ko inherit kare.

Halaanki, agar aapko apne programming toolbox mein inheritance ki aadat hai, to aap Rust mein doosre samadhanon ka upyog kar sakte hain. Aap do mukhya kaaranon se inheritance ko chunte hain:

1.  **Code Reuse (Code ka Punah Upyog):** Aap ek type ke liye vishesh behavior implement kar sakte hain, aur inheritance aapko us implementation ko ek alag type ke liye dobara upyog karne ki anumati deta hai. Rust mein aap yah **default trait method implementations** ka upyog karke kar sakte hain. Yah ek parent class ke method ke samaan hai jo inheriting child class ke liye bhi uplabdh hota hai.

2.  **Type System (Polymorphism):** Ek child type ko unhi jagahon par istemal karne ki anumati dena jahan parent type ka istemal hota hai. Ise **polymorphism** bhi kaha jaata hai, jiska matlab hai ki aap runtime par kai objects ko ek doosre ke liye substitute kar sakte hain agar unmein kuch visheshtayein samaan hon.

> ### Polymorphism
>
> Kai logon ke liye, polymorphism inheritance ka synonym hai. Lekin yah vastav mein ek adhik samanya concept hai jo us code ko sandarbhit karta hai jo kai prakaar ke data ke saath kaam kar sakta hai.
>
> Rust iske bajaye alag-alag sambhavit types par abstract karne ke liye **generics** aur un types par badhyatayein (constraints) lagane ke liye **trait bounds** ka upyog karta hai. Ise kabhi-kabhi **bounded parametric polymorphism** kaha jaata hai.

Rust ne inheritance na pradan karke ek alag set of tradeoffs chuna hai. Inheritance aksar zaroorat se zyada code share karne ke jokhim mein hota hai. Subclasses ko hamesha apne parent class ki sabhi visheshtaon ko share nahi karna chahiye. Isse program ka design kam lacheela ho sakta hai.

In kaaranon se, Rust polymorphism ko saksham karne ke liye inheritance ke bajaye **trait objects** ka upyog karne ka alag drishtikon apnata hai. Chaliye dekhen ki trait objects kaise kaam karte hain.