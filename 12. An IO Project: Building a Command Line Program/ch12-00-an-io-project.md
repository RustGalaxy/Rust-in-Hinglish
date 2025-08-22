# An I/O Project: Building a Command Line Program

Ye chapter ab tak aapne jo skills seekhi hain unka **recap** hai, aur ismein hum kuch aur standard library features ko bhi explore karenge. Hum ek **command line tool** banayenge jo file aur command line input/output ke saath **interact** karta hai, taaki aap kuch Rust concepts ki practice kar saken jo ab aapne seekh liye hain.

Rust ki speed, safety, single binary output, aur cross-platform support isko **command line tools** banane ke liye ek **ideal language** banati hain. Isliye, hamare project ke liye, hum classic command line tool **`grep`** (**g**lobally search a **r**egular **e**xpression and **p**rint) ka apna version banayenge. Iske sabse simple use case mein, `grep` ek specify ki gayi file mein ek specify ki gayi string ko dhoondhta hai. Aisa karne ke liye, `grep` apne arguments ke roop mein ek **filename** aur ek **string** leta hai. Fir ye file ko padhta hai, un lines ko dhoondhta hai jismein string argument hota hai, aur un lines ko print karta hai.

Is dauran, hum dikhayenge ki hamare command line tool ko terminal ke features ka upyog kaise karwaya jaaye. Hum ek **environment variable** ki value padhenge taaki user ko hamare tool ke behavior ko **configure** karne diya jaaye. Hum **standard output (`stdout`)** ke bajaye **standard error console stream (`stderr`)** par bhi print karenge, taaki, for example, user successful output ko ek file mein redirect kar sake, jabki error messages screen par hi dekhta rahe.

Rust community ke ek member, Andrew Gallant, ne already `grep` ka ek fully featured, bahut fast version bana rakha hai, jise **`ripgrep`** kehte hain. Uske comparison mein, hamara `grep` version kaafi simple hoga, lekin ye chapter aapko wo background knowledge dega jo aapko `ripgrep` jaise ek **real-world project** ko samajhne ke liye chahiye.

Hamara `grep` project kayi concepts ko combine karega jo aapne ab tak seekhe hain:

* Code ko **modules** ka upyog karke organize karna (Chapter 7 se)
* **Vectors** aur **strings** ka upyog (Chapter 8)
* **Errors** ko handle karna (Chapter 9)
* Jahan zaroori ho, wahan **traits** aur **lifetimes** ka upyog (Chapter 10)
* **Tests** likhna (Chapter 11)

Hum **closures**, **iterators**, aur **trait objects** ko bhi briefly introduce karenge, jinhein hum **Chapters 13** aur **17** mein detail mein cover karenge.
