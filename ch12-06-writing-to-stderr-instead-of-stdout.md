## Error Messages ko Standard Output ke Bajaye Standard Error par Likhna 📝

Filhaal, hum `println!` function ka upyog karke apna saara output **terminal** par likh rahe hain. Zyaadatar terminals do tarah ke output provide karte hain: **standard output (`stdout`)** general jaankari ke liye aur **standard error (`stderr`)** error messages ke liye. Ye alag-alag output streams users ko program ke successful output ko ek file mein bhejne ki suvidha dete hain, jabki error messages screen par hi print hote rehte hain.

`println!` function sirf standard output par print kar sakta hai, isliye humein standard error par print karne ke liye kuch aur use karna hoga.

-----

### Check karna ki Errors Kahan Likhi Jaa Rahi Hain 🔎

Sabse pehle, hum dekhenge ki `minigrep` dwara print kiya gaya content currently standard output par kaise likha ja raha hai. Hum standard output stream ko ek file par redirect karke aisa karenge, jabki jaanbujhkar ek error bhi cause karenge. Hum standard error stream ko redirect nahi karenge, taaki standard error par bheja gaya koi bhi content screen par dikhta rahe.

Command line programs se ye umeed ki jaati hai ki wo error messages ko standard error stream par bhejenge, taaki hum error messages ko screen par dekh saken, bhale hi hum standard output stream ko ek file par redirect kar dein. Lekin hamara program filhaal sahi nahi hai: hum abhi dekhenge ki ye error message output ko ek file mein save karta hai.

Is behavior ko demonstrate karne ka tareeka hai program ko `>` aur filename, *output.txt*, ke saath run karna, jismein hum standard output stream ko redirect karna chahte hain. Hum koi arguments pass nahi karenge, jisse ek error cause hona chahiye:

```text
$ cargo run > output.txt
```

`>` syntax shell ko batata hai ki standard output ke contents ko screen ke bajaye *output.txt* mein likho. Hamein screen par wo error message nahi dikha jiski hum umeed kar rahe the, iska matlab hai ki wo file mein chala gaya hai. *output.txt* mein ye hai:

```text
Problem parsing arguments: not enough arguments
```

Haan, hamara error message standard output par print ho raha hai. Is tarah ke error messages ka standard error par print hona zyaada useful hai, taaki ek successful run ka data hi file mein jaayega. Hum ise badlenge.

-----

### Errors ko Standard Error par Print karna 🖨️

Hum error messages ko print karne ke tareeke ko badalne ke liye **Listing 12-24** ke code ka upyog karenge. `eprintln!` macro standard error stream par print karta hai, to chaliye `main` mein un do jagahon ko badalte hain jahan hum errors print karne ke liye `println!` ko call kar rahe the, taaki unki jagah `eprintln!` ka upyog ho.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {}", e);

        process::exit(1);
    }
}
```

\<span class="caption"\>Listing 12-24: `eprintln!` ka upyog karke error messages ko standard output ke bajaye standard error par likhna\</span\>

`println!` ko `eprintln!` mein badalne ke baad, chaliye program ko phir se usi tarah run karte hain, bina kisi arguments ke aur `>` ke saath standard output ko redirect karte hue:

```text
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Ab humein screen par error dikhti hai aur *output.txt* mein kuch bhi nahi hai, jo command line programs se expected behavior hai.

Chaliye program ko phir se aise arguments ke saath run karte hain jo error cause nahi karte hain, lekin phir bhi standard output ko ek file par redirect karte hain:

```text
$ cargo run to poem.txt > output.txt
```

Hamein terminal par koi output nahi dikhega, aur *output.txt* mein hamare results honge:

\<span class="filename"\>Filename: output.txt\</span\>

```text
Are you nobody, too?
How dreary to be somebody!
```

Ye demonstrate karta hai ki hum ab successful output ke liye standard output aur error output ke liye standard error ka upyog kar rahe hain.

## Summary 📝

Is chapter ne ab tak aapne seekhe hue kuch major concepts ko recap kiya aur Rust mein common **I/O operations** kaise perform karte hain ye cover kiya hai. Command line arguments, files, environment variables, aur errors print karne ke liye `eprintln!` macro ka upyog karke, aap ab command line applications likhne ke liye taiyar hain. Pichle chapters ke concepts ka upyog karke, aapka code well organized hoga, data ko effective tareeke se appropriate data structures mein store karega, errors ko acche se handle karega, aur well tested hoga.

Aage, hum Rust ke kuch features ko explore karenge jo functional languages se influenced hain: **closures aur iterators**.
