## Hello, World!

Ab jab aapne Rust install kar liya hai, chalo apna pehla Rust program likhte hain.
Jab bhi hum koi nayi language seekhte hain, tradition hai ki ek chota sa program likhe jo terminal pe **`Hello, world!`** print kare.
Yahi hum yahan karenge!

> **Note:** Ye book assume karti hai ki aapko command line ki basic knowledge hai. Rust kisi bhi specific editor, IDE, ya file location ki demand nahi karta. Agar aap prefer karte ho IDE use karna, to apne favorite IDE me kaam kar sakte ho. Aajkal kai IDEs me Rust support available hai, check kar lo IDE documentation.

### Creating a Project Directory

Sabse pehle ek directory banate hain apne Rust code ke liye. Rust ko farak nahi padta ki code kaha hai, lekin book ke exercises ke liye hum suggest karte hain ki **projects** directory banaye aur saare projects waha rakhe.

Linux/macOS terminal commands:

```text
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Windows CMD commands:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

Windows PowerShell commands:

```powershell
> mkdir $env:USERPROFILE\projects
> cd $env:USERPROFILE\projects
> mkdir hello_world
> cd hello_world
```

### Writing and Running a Rust Program

Ab ek new source file banate hain aur uska naam **main.rs** rakhte hain. Rust files hamesha *.rs* extension se end hoti hain. Agar filename me do words ho to underscore use karo, jaise **hello\_world.rs**.

Ab **main.rs** open karo aur code likho:

```rust
fn main() {
    println!("Hello, world!");
}
```

Ye program terminal pe `Hello, world!` print karega.

Linux/macOS me compile aur run karne ke liye:

```text
$ rustc main.rs
$ ./main
Hello, world!
```

Windows me:

```powershell
> rustc main.rs
> .\main.exe
Hello, world!
```

Agar output sahi print ho gaya, congratulations! Ab aap officially Rust programmer ho 😎

### Anatomy of a Rust Program

Hello, world! program ka breakdown:

```rust
fn main() {

}
```

* Ye define karta hai ek function. `main` function har executable Rust program me pehla run hota hai.
* `()` me parameters hote agar koi hote.
* Function body curly braces `{}` me wrap hoti hai.
* Style ke liye opening `{` function declaration ke same line me hona chahiye.

Function body me:

```rust
    println!("Hello, world!");
```

* 4 spaces se indent karo (tab nahi).
* `println!` ek **macro** call karta hai (function call nahi).
* `"Hello, world!"` string ko print karta hai.
* Semicolon `;` se line end hoti hai.

### Compiling and Running Are Separate Steps

Rust me **compile aur run alag steps** hote hain.

Compile karne ke liye:

```text
$ rustc main.rs
```

* Ye binary executable generate karega.

Linux/macOS/PowerShell me executable check karne ke liye:

```text
$ ls
main  main.rs
```

Windows CMD me:

```cmd
> dir /B
main.exe
main.pdb
main.rs
```

Run karne ke liye:

```text
$ ./main  # Windows: .\main.exe
```

Rust *ahead-of-time compiled* language hai. Aap executable kisi ko de sakte ho, bina Rust installed ke bhi run ho jayega.

Ab `rustc` simple programs ke liye kaafi hai, lekin bade projects me aapko options manage karne aur code share karne me **Cargo tool** help karega.

Next, hum Cargo introduce karenge aur real-world Rust programs likhna start karenge.
