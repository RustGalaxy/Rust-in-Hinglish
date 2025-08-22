## Patterns aur Matching

**Patterns** 🧩 Rust mein ek khaas syntax hain jinka istemal types ke structure, chahe woh complex ho ya simple, ke khilaaf match karne ke liye hota hai. `match` expressions aur doosre constructs ke saath patterns ka istemal karne se aapko program ke control flow par zyada niyantran milta hai. Ek pattern nimnalikhit (following) cheezon ke combination se banta hai:

* Literals
* Destructured arrays, enums, structs, ya tuples
* Variables
* Wildcards
* Placeholders

Kuch udaharan patterns mein `x`, `(a, 3)`, aur `Some(Color::Red)` shaamil hain. Jin contexts mein patterns valid hote hain, yeh components data ke aakar (shape) ko describe karte hain. Hamara program phir values ko patterns ke khilaaf match karta hai yah dekhne ke liye ki code ke ek vishesh hisse ko aage chalane ke liye uske paas sahi aakar ka data hai ya nahi.

Ek pattern ka istemal karne ke liye, hum use kisi value se compare karte hain. Agar pattern value se match karta hai, to hum apne code mein value ke hisson ka istemal karte hain. Chapter 6 ke `match` expressions ko yaad karein jinhone patterns ka istemal kiya tha, jaise ki coin-sorting machine wala udaharan. Agar value pattern ke aakar mein fit baithti hai, to hum naam diye gaye tukdon (named pieces) ka istemal kar sakte hain. Agar nahi, to pattern se juda code nahi chalega.

Yah chapter patterns se judi sabhi cheezon par ek reference hai. Hum patterns ka istemal karne ke liye valid jagahon, **refutable** aur **irrefutable** patterns ke beech ke antar, aur alag-alag prakaar ke pattern syntax jo aap dekh sakte hain, ko cover karenge. Is chapter ke ant tak, aap jaan jayenge ki kai concepts ko spasht tareeke se vyakt karne ke liye patterns ka istemal kaise karein.