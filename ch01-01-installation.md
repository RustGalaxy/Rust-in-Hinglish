## Installation

Sabse pehla step hai Rust install karna.
Hum Rust ko download karenge **`rustup`** ke through — yeh ek command line tool hai jo Rust versions aur tools ko manage karta hai.
Download ke liye internet connection zaroori hai.

> **Note:** Agar aap kisi wajah se `rustup` use nahi karna chahte, to [Rust installation page](https://www.rust-lang.org/install.html) par aur options diye gaye hain.

Neeche diye gaye steps se aapko Rust compiler ka **latest stable version** milega.
Is book ke saare examples aur outputs **stable Rust 1.21.0** pe based hain.

Rust ke stability guarantees ka matlab hai:

* Jo code is book me compile hota hai, woh future Rust versions pe bhi compile hoga.
* Bas thoda output difference ho sakta hai (kyunki Rust compiler ke error/warning messages time ke sath improve hote rehte hain).

Matlab, koi bhi **latest stable Rust version** aap install karoge, woh is book ke examples ke saath sahi chalega.

---

> ### Command Line Notation
>
> Book me jo bhi terminal commands dikhaye gaye hain woh `$` se start hote hain.
>
> * `$` ka matlab hai command ka start — aapko `$` type nahi karna hai.
> * Jo lines `$` se start nahi hoti, woh commands ka **output** hoti hain.
> * Agar PowerShell ka example hoga to `>` use kiya jaayega instead of `$`.

---

### Installing `rustup` on Linux or macOS

Linux/macOS me terminal open karo aur yeh command run karo:

```text
$ curl https://sh.rustup.rs -sSf | sh
```

Yeh command ek script download karta hai aur `rustup` install kar deta hai,
jo Rust ka **latest stable version** install karega.

Installation ke time password maanga jaa sakta hai.
Agar installation successful ho jaata hai to yeh message milega:

```text
Rust is installed now. Great!
```

👉 Tip: Script ko run karne se pehle aap chahe to usko download karke inspect kar sakte ho.

* Installation script automatically Rust ko aapke **system PATH** me add kar dega (next login ke baad).
* Agar aap bina restart kiye Rust use karna chahte ho, to yeh command run karo:

```text
$ source $HOME/.cargo/env
```

Ya phir apne *\~/.bash\_profile* me yeh line add kar do:

```text
$ export PATH="$HOME/.cargo/bin:$PATH"
```

⚡ Extra Note: Rust ko compile karne ke liye ek **linker** bhi chahiye hota hai.
Usually yeh already installed hota hai, lekin agar aapko compile karte waqt linker error aaye to iska matlab hai linker missing hai.
C compilers ke saath linker aata hai, isliye ek C compiler install karna useful hoga (aur kuch Rust packages ko C code ki zaroorat hoti hai).

---

### Installing `rustup` on Windows

Windows par [https://www.rust-lang.org/install.html][install] par jao aur installation instructions follow karo.

Installation ke time aapse kaha jaayega ki aapko **C++ build tools for Visual Studio 2013 or later** chahiye.
Sabse easy way hai install karna: [Build Tools for Visual Studio 2017][visualstudio] (Other Tools and Frameworks section me available hai).

[install]: https://www.rust-lang.org/install.html
[visualstudio]: https://www.visualstudio.com/downloads/

Book me aage jitne commands diye gaye hain woh **cmd.exe aur PowerShell dono me chalenge**. Agar koi difference hoga, woh clearly bataya jaayega.

---

### Updating and Uninstalling

Rust ko latest version me update karna easy hai:

```text
$ rustup update
```

Rust aur `rustup` uninstall karne ke liye yeh command run karo:

```text
$ rustup self uninstall
```

---

### Troubleshooting

Check karne ke liye ki Rust properly install hua hai ya nahi, yeh command run karo:

```text
$ rustc --version
```

Aapko kuch aisa output dekhne ko milega:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Agar aapko yeh information milti hai, matlab Rust sahi install hua hai ✅

Agar nahi mila (specially Windows pe), to check karo ki Rust aapke `%PATH%` me hai ya nahi.
Agar sab sahi hai aur phir bhi problem aa rahi hai, to help ke liye ye resources use kar sakte ho:

* [#rust IRC channel (irc.mozilla.org)][irc] (web se access ke liye [Mibbit][mibbit])
* [Rust Users forum][users]
* [Stack Overflow][stackoverflow]

[irc]: irc://irc.mozilla.org/#rust
[mibbit]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[users]: https://users.rust-lang.org/
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust

---

### Local Documentation

Installer ke sath Rust documentation bhi local copy me install hoti hai.
Offline docs open karne ke liye run karo:

```text
$ rustup doc
```

Jab bhi aapko kisi type ya function ke baare me doubt ho,
to **API documentation** open karke check kar lo ki woh kya karta hai aur kaise use hota hai.
