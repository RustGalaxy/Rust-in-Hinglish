# Introduction  

> Note: Yeh book ki edition wahi hai jo [The Rust Programming Language][nsprust] ke naam se **print aur ebook format** me [No Starch Press][nsp] se available hai.  

[nsprust]: https://nostarch.com/rust  
[nsp]: https://nostarch.com/  

Welcome to *The Rust Programming Language*!  
Yeh ek **introductory book** hai jo Rust programming language ke baare me hai.  
Rust aapko help karta hai **faster aur reliable software** likhne me.  

Normally, programming languages me ek trade-off hota hai:  
- High-level ergonomics (easy to use features)  
- Low-level control (jaise memory usage manage karna)  

Rust iss conflict ko challenge karta hai. Yeh dono ka balance provide karta hai:  
- Powerful technical capabilities  
- Great developer experience  

Iska matlab yeh hai ki aap low-level details control kar sakte ho (memory, concurrency) **without extra headache** jo normally aise kaam ke saath aata hai.  

---

## Who Rust Is For  

Rust bahut saare logon ke liye ideal hai. Chalo kuch groups dekhte hain.  

### Teams of Developers  

Rust ek productive tool ban raha hai large teams ke liye.  
Systems programming me low-level code me kaafi subtle bugs aate hain, aur dusre languages me yeh sirf **heavy testing aur expert code reviews** se hi pakde jaa sakte hain.  

Rust ka **compiler gatekeeper ki tarah kaam karta hai**. Yeh code ko tab tak compile nahi karta jab tak aise tricky bugs (including concurrency bugs) fix na ho.  

Iska fayda: Team bugs chase karne ke bajaye **program logic pe focus kar sakti hai**.  

Rust saath me modern developer tools bhi lata hai:  
- **Cargo** → dependency manager + build tool (dependencies add/manage karna easy aur consistent banata hai)  
- **Rustfmt** → consistent coding style enforce karta hai  
- **Rust Language Server (RLS)** → IDE integration (code completion, inline error messages)  

In tools ki wajah se developers productive hote hain even while writing **systems-level code**.  

---

### Students  

Rust students ke liye bhi perfect hai, jo systems concepts seekhna chahte hain.  
Rust use karke logon ne **Operating Systems development** aur aur bhi topics seekhe hain.  

Rust community welcoming hai aur student questions happily answer karti hai.  
Is book jaise efforts ka main aim hai systems concepts ko easy aur accessible banana, especially unke liye jo programming me naye hain.  

---

### Companies  

Aaj ke time me **hundreds of companies** (chhoti aur badi) Rust ko production me use kar rahi hain:  

- Command line tools  
- Web services  
- DevOps tooling  
- Embedded devices  
- Audio/video analysis & transcoding  
- Cryptocurrencies  
- Bioinformatics  
- Search engines  
- IoT applications  
- Machine Learning  
- Firefox web browser ke kuch major parts bhi Rust me likhe gaye hain  

---

### Open Source Developers  

Rust unke liye bhi hai jo **Rust language, community, tools aur libraries** me contribute karna chahte hain.  
Agar aap contribute karna chahte ho, Rust community aapka welcome karti hai!  

---

### People Who Value Speed and Stability  

Rust unke liye hai jo **speed + stability** chahte hain.  

- **Speed** → Programs fast run hote hain aur fast likhne me madad milti hai.  
- **Stability** → Compiler checks ki wajah se code safe aur stable hota hai, even jab features add/refactor kiye jaate hain.  

Dusre languages me, legacy code fragile hota hai aur devs changes karne se darte hain.  
Rust ka goal hai: **safe code = fast code**.  

---

## Who This Book Is For  

Yeh book assume karti hai ki aapne kisi aur programming language me coding kiya hai (kaunsi language → matter nahi karta).  

Agar aap bilkul programming me naye ho, to yeh book best nahi hai. Aapko ek aur introductory programming book start karni chahiye.  

---

## How to Use This Book  

Yeh book ko best tarike se **front se back** sequentially read kiya jaa sakta hai, kyunki chapters ek dusre pe build hote hain.  

Book me do tarah ke chapters hain:  
1. **Concept chapters** → Rust ke features samjhaye jaate hain  
2. **Project chapters** → Small programs banaye jaate hain taaki concepts apply ho sake  

- Chapter 1 → Install Rust, Hello World program, Cargo  
- Chapter 2 → Hands-on introduction (agar aap turant shuru karna chahte ho)  
- Chapter 3 → Basic Rust features (jo dusre languages me bhi hote hain)  
- Chapter 4 → Ownership system (Rust ka unique feature)  
- Chapter 5 → Structs aur methods  
- Chapter 6 → Enums, `match`, `if let`  
- Chapter 7 → Module system aur privacy rules (API design)  
- Chapter 8 → Common data structures (vectors, strings, hash maps)  
- Chapter 9 → Error handling philosophy  
- Chapter 10 → Generics, traits, lifetimes  
- Chapter 11 → Testing  
- Chapter 12 → Project: `grep` tool ka ek subset banayenge  
- Chapter 13 → Closures aur iterators  
- Chapter 14 → Cargo deep dive + library sharing  
- Chapter 15 → Smart pointers aur traits  
- Chapter 16 → Concurrency models (fearless multithreading)  
- Chapter 17 → Rust idioms vs OOP principles  
- Chapter 18 → Patterns aur pattern matching  
- Chapter 19 → Advanced topics (unsafe Rust, lifetimes, traits, closures, etc.)  
- Chapter 20 → Project: Low-level multithreaded web server  

Appendices:  
- A → Keywords  
- B → Operators & symbols  
- C → Derivable traits (std library)  
- D → Macros  

⚡ Important: Rust seekhte waqt sabse important skill hai → **compiler ke error messages padhna aur samajhna**.  
Is book me hum deliberately aise code examples bhi dikhayenge jo compile nahi hote, taaki aap error messages ko samajh sako.  

---

## Source Code  

Is book ka source code yahan available hai:  
👉 [GitHub repo][book]  

[book]: https://github.com/rust-lang/book/tree/master/second-edition/src
