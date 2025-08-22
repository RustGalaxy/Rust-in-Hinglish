# Smart Pointers

Ek **pointer** ek general concept hai, ek aise variable ke liye jismein memory ka ek address hota hai. Yeh address kisi doosre data ko refer karta hai, ya uski taraf "point" karta hai. Rust mein sabse aam tarah ka pointer **reference** hai, jiske baare mein aapne Chapter 4 mein padha tha. References ko `&` symbol se dikhaya jaata hai aur woh us value ko **borrow** karte hain jis par woh point karte hain. Unke paas data ko refer karne ke alawa koi special capabilities nahi hoti. Saath hi, unka koi overhead (extra performance cost) nahi hota aur yeh woh pointer hain jo hum sabse zyada istemaal karte hain.

---

Doosri taraf, **Smart pointers** aise data structures hain jo na sirf ek pointer ki tarah kaam karte hain, balki unke paas additional **metadata** (extra jaankari) aur **capabilities** (shamtaayein) bhi hoti hain. Smart pointers ka concept sirf Rust tak seemit nahi hai: iski shuruaat C++ mein hui thi aur yeh doosri languages mein bhi maujood hai. Rust mein, standard library mein define kiye gaye alag-alag smart pointers references se zyada functionality pradaan karte hain. Ek example jo hum is chapter mein explore karenge, woh hai ***reference counting*** smart pointer type. Yeh pointer aapko data ke multiple owners rakhne ki anumati deta hai, owners ki sankhya ka track rakhta hai, aur jab koi owner nahi bachta, to data ko clean up kar deta hai.

Rust mein, jo ownership aur borrowing ke concept ka istemaal karta hai, references aur smart pointers ke beech ek aur fark yeh hai ki references aise pointers hain jo sirf data ko **borrow** karte hain; iske vipreet, kai maamlon mein, smart pointers us data ko **own** karte hain jis par woh point karte hain.

---

Hum is kitaab mein pehle hi kuch smart pointers se mil chuke hain, jaise Chapter 8 mein `String` aur `Vec<T>`, haalanki us samay humne unhe smart pointers nahi kaha tha. Yeh dono types smart pointers maane jaate hain kyunki woh kuch memory ko own karte hain aur aapko use manipulate karne dete hain. Unke paas metadata (jaise unki capacity) aur extra capabilities ya guarantees (jaise `String` yeh sunishchit karta hai ki uska data hamesha valid UTF-8 rahega) bhi hoti hain.

Smart pointers aam taur par **structs** ka use karke implement kiye jaate hain. Ek smart pointer ko ek saadharan struct se alag karne wali visheshta yeh hai ki smart pointers `Deref` aur `Drop` traits ko implement karte hain. **`Deref` trait** ek smart pointer struct ke instance ko ek reference ki tarah behave karne deta hai, taaki aap aisa code likh sakein jo references ya smart pointers dono ke saath kaam kare. **`Drop` trait** aapko us code ko customize karne deta hai jo tab chalta hai jab smart pointer ka instance scope se bahar jaata hai. Is chapter mein, hum dono traits par charcha karenge aur batayenge ki woh smart pointers ke liye itne mahatvapurna kyun hain.

---

Yeh dekhte hue ki smart pointer pattern Rust mein aksar istemaal hone wala ek general design pattern hai, yeh chapter har maujooda smart pointer ko cover nahi karega. Kai libraries ke apne smart pointers hote hain, aur aap apne khud ke bhi likh sakte hain. Hum standard library mein sabse aam smart pointers ko cover karenge:

* `Box<T>`, heap par values allocate karne ke liye.
* `Rc<T>`, ek reference counting type jo multiple ownership ko enable karta hai.
* `Ref<T>` aur `RefMut<T>`, jinhe `RefCell<T>` ke zariye access kiya jaata hai, ek aisa type jo compile time ke bajaye runtime par borrowing rules laagoo karta hai.

Iske alawa, hum ***interior mutability*** pattern ko cover karenge, jahan ek immutable type ek andarooni value ko mutate karne ke liye ek API deta hai. Hum ***reference cycles*** par bhi charcha karenge: ki woh kaise memory leak kar sakte hain aur unhe kaise roka ja sakta hai.

Chaliye, shuru karte hain! 🚀
