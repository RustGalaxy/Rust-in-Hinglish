## Custom Commands se Cargo ko Extend Karna

Cargo ko is tarah design kiya gaya hai ki aap use naye subcommands ke saath extend kar sakte hain, bina Cargo ko modify kiye. Agar aapke `$PATH` mein koi binary `cargo-something` naam se hai, to aap use `cargo something` chalaakar ek Cargo subcommand ki tarah run kar sakte hain. Is tarah ke custom commands `cargo --list` chalaane par bhi list hote hain. `cargo install` ka use karke extensions ko install karna aur phir unhe bilkul built-in Cargo tools ki tarah chala paana, Cargo ke design ka ek super convenient fayda hai! 👍

---
## Summary

Cargo aur [crates.io](https://crates.io) ke saath code share karna ek ahem hissa hai jo Rust ecosystem ko kai alag-alag kaamon ke liye upyogi banata hai. Rust ki standard library chhoti aur stable hai, lekin crates ko share karna, use karna, aur improve karna aasan hai, aur yeh language se alag timeline par ho sakta hai. Jo code aapke liye upyogi hai, use [crates.io](https://crates.io) par share karne se na jhijhkein; poori sambhavna hai ki woh kisi aur ke liye bhi upyogi hoga! 😊
