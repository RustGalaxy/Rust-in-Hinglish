## Release Profiles se Builds ko Customize Karna

Rust mein, ***release profiles*** pehle se define kiye gaye aur customize kiye jaa sakne wale profiles hote hain. Inmein alag-alag configurations hoti hain jisse ek programmer ko code compile karne ke various options par zyada control milta hai. Har profile ko doosron se alag, independently configure kiya jaata hai.

Cargo ke do main profiles hain: `dev` profile, jise Cargo tab use karta hai jab aap `cargo build` run karte hain, aur `release` profile, jise Cargo tab use karta hai jab aap `cargo build --release` run karte hain. `dev` profile development ke liye achhe defaults ke saath aata hai, aur `release` profile release builds ke liye achhe defaults ke saath aata hai.

Yeh profile names shayad aapko apne builds ke output se familiar lagenge:

```text
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
$ cargo build --release
    Finished release [optimized] target(s) in 0.0 secs
```

Is build output mein dikhaya gaya `dev` aur `release` yeh batata hai ki compiler alag-alag profiles ka istemaal kar raha hai.

Cargo ke paas har profile ke liye default settings hoti hain jo tab apply hoti hain jab project ki *Cargo.toml* file mein koi `[profile.*]` section nahi hota. Aap jis bhi profile ko customize karna chahte hain, uske liye `[profile.*]` section add karke, aap default settings ke kisi bhi hisse ko override kar sakte hain. Example ke liye, yahan `dev` aur `release` profiles ke liye `opt-level` setting ki default values di gayi hain:

\<span class="filename"\>Filename: Cargo.toml\</span\>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` setting yeh control karti hai ki Rust aapke code par kitne optimizations apply karega, iski range 0 se 3 tak hoti hai. Zyada optimizations apply karne se compiling time badh jaata hai, isliye agar aap development mein hain aur apne code ko baar-baar compile kar rahe hain, to aap fast compiling chahenge, bhale hi resulting code thoda slow chale. Yahi kaaran hai ki `dev` ke liye default `opt-level` `0` hai. Jab aap apna code release karne ke liye taiyar hon, to compiling mein zyada time lagana behtar hai. Aap release mode mein sirf ek baar compile karenge, lekin compiled program ko kai baar run karenge, isliye release mode lambe compile time ke badle mein fast chalne wala code deta hai. Yahi vajah hai ki `release` profile ke liye default `opt-level` `3` hai.

Aap *Cargo.toml* mein kisi bhi default setting ke liye ek alag value add karke use override kar sakte hain. For example, agar hum development profile mein optimization level 1 use karna chahte hain, to hum apne project ki *Cargo.toml* file mein yeh do lines add kar sakte hain:

\<span class="filename"\>Filename: Cargo.toml\</span\>

```toml
[profile.dev]
opt-level = 1
```

Yeh code `0` ki default setting ko override kar deta hai. Ab jab hum `cargo build` run karenge, to Cargo `dev` profile ke defaults ke saath-saath `opt-level` ke liye hamare customization ka bhi istemaal karega. Kyunki humne `opt-level` ko `1` set kiya hai, Cargo default se zyada optimizations apply karega, lekin utne nahi jitne release build mein hote hain.

Har profile ke liye configuration options aur defaults ki poori list ke liye, [Cargo ka documentation](https://doc.rust-lang.org/cargo/) dekhein.
