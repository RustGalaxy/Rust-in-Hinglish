# Error Handling

Rust ki reliability ka commitment **error handling** tak bhi jaata hai. Software mein errors aana ek aam baat hai, isliye Rust mein bahut se features hain jisse aap situations ko handle kar sakte hain jab kuch galat ho jaata hai. Kayi cases mein, Rust aapse ek error ke aane ki possibility ko acknowledge karne ko kehta hai aur usse pehle ki aapka code compile ho, kuch action lene ko kehta hai. Ye requirement aapke program ko zyaada **robust** banati hai, kyunki ye ensure karti hai ki aap apne code ko production mein deploy karne se pehle hi errors ko discover karenge aur unhein sahi tareeke se handle karenge.

Rust errors ko do main categories mein divide karta hai: **recoverable** aur **unrecoverable** errors.

---

### Recoverable Errors 🤕

Recoverable errors wo hote hain jinhein hum handle kar sakte hain. Jaise, agar koi file nahi milti (a `file not found` error), to us problem ko user ko report karna aur operation ko retry karna reasonable hai. Zyaadatar languages exceptions jaise mechanisms ka upyog karke is tarah ki errors ko handle karte hain.

Rust mein exceptions nahi hote. Iske bajaye, ye **`Result<T, E>` type** ka upyog karta hai recoverable errors ke liye.

---

### Unrecoverable Errors 💥

Unrecoverable errors hamesha **bugs ke symptoms** hote hain. Jaise ki, ek array ke end ke baad ki location ko access karne ki koshish karna. Aisi situation mein, program ko aage continue karne ka koi tareeka nahi hai.

Rust mein, jab program ko ek unrecoverable error milti hai, to ye **`panic!` macro** ka upyog karke execution rok deta hai.

Aage is chapter mein, hum pehle `panic!` ko call karne ke baare mein baat karenge aur fir `Result<T, E>` values ko return karne ke baare mein explore karenge. Iske alawa, hum is par bhi baat karenge ki kab error se recover karne ki koshish karni chahiye aur kab execution rok dena chahiye.
