# Functional Language Features: Iterators and Closures

Rust ka design kayi existing languages aur techniques se inspired hai, aur ek significant influence hai **functional programming**. Functional style mein programming mein aksar functions ko values ki tarah use kiya jaata hai — unhein arguments mein pass karke, doosre functions se return karke, ya unhein variables ko assign karke baad mein execute karne ke liye, aur aise hi aur bhi tarike se.

Is chapter mein, hum is par debate nahi karenge ki functional programming kya hai ya kya nahi, balki Rust ke kuch features par discuss karenge jo kayi aisi languages ke features se similar hain jinhein functional kaha jaata hai.

Specifically, hum cover karenge:

* **Closures**, jo ek function-jaisa construct hai jise aap ek variable mein store kar sakte hain.
* **Iterators**, jo elements ki ek series ko process karne ka ek tareeka hai.
* In dono features ka upyog karke **Chapter 12** ke I/O project ko kaise improve karein.
* In dono features ki **performance**. (Spoiler alert: ye aapke sochte se zyaada fast hain!)

Rust ke kuch aur features, jaise ki **pattern matching** aur **enums**, jinhein humne doosre chapters mein cover kiya hai, wo bhi functional style se inspired hain. Closures aur iterators ko master karna idiomatic aur fast Rust code likhne ka ek zaroori hissa hai, isliye hum is poore chapter ko un par devote karenge.
