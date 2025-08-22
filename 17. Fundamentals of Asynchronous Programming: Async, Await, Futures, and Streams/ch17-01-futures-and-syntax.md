## Futures aur Async Syntax

Rust mein asynchronous programming ke mukhya tatva (key elements) hain **futures** aur Rust ke `async` aur `await` keywords.

Ek **future** ek aisi value hai jo shayad abhi taiyaar na ho, lekin bhavishya mein kisi samay taiyaar ho jayegi. (Yahi concept kai bhashaon mein aata hai, kabhi-kabhi doosre naamon se jaise ki *task* ya *promise*.) Rust ek building block ke roop mein `Future` trait pradan karta hai. Rust mein, futures aise types hain jo `Future` trait ko implement karte hain.

Aap `async` keyword ko blocks aur functions par laga sakte hain yeh batane ke liye ki unhe interrupt karke resume kiya ja sakta hai. Ek async block ya async function ke andar, aap `await` keyword ka istemal *ek future ka intezaar karne* ke liye kar sakte hain (yaani, uske taiyaar hone ka intezaar karna). Jis bhi jagah aap ek future ka `await` karte hain, woh us async block ya function ke liye pause aur resume karne ka ek potential spot hota hai. Ek future ko check karne ki prakriya ki uski value uplabdh hai ya nahi, **polling** kehlati hai.

-----

Jab hum async Rust likhte hain, to hum zyada tar `async` aur `await` keywords ka istemal karte hain. Rust unhe `Future` trait ka istemal karke barabar ke code mein compile karta hai, bilkul waise hi jaise vah `for` loops ko `Iterator` trait ka istemal karke barabar ke code mein compile karta hai.

Yeh sab thoda abstract lag sakta hai, isliye aaiye apna pehla async program likhte hain: ek chhota web scraper. Hum command line se do URLs pass karenge, dono ko concurrently fetch karenge, aur jo bhi pehle poora hoga, uska result return karenge.

## Hamara Pehla Async Program

Is chapter mein async seekhne par dhyan kendrit rakhne ke liye, humne `trpl` crate banaya hai (jo "The Rust Programming Language" ka short form hai). Yah aapke liye zaroori sabhi types, traits aur functions ko re-export karta hai, mukhya roop se [`futures`][futures-crate] aur [`tokio`][tokio] crates se. Tokio aaj Rust mein sabse zyada istemal hone wala async runtime hai, khaas karke web applications ke liye.

Ek naya binary project `hello-async` banayein aur `trpl` crate ko dependency ke roop mein jodein:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Ab hum `trpl` dwara pradan kiye gaye vibhinn tukdon ka istemal karke apna pehla async program likh sakte hain. Hum ek chhota command line tool banayenge jo do web pages ko fetch karta hai, har ek se `<title>` element nikalta hai, aur jo bhi page is poori prakriya ko pehle poora karta hai, uska title print karta hai.

### `page_title` Function ko Define Karna

Chaliye ek function likh kar shuruaat karte hain jo ek page URL ko parameter ke roop mein leta hai, us par ek request karta hai, aur title element ka text return karta hai (Listing 17-1).

```rust
// Filename: src/main.rs
use trpl::{get, Html};

async fn page_title(url: &str) -> Option<String> {
    let response_text = get(url).await.text().await;
    Html::parse(&response_text)
        .select_first("title")
        .map(|title| title.inner_html())
}
```

*Listing 17-1: Ek HTML page se title element prapt karne ke liye ek async function define karna*

Sabse pehle, hum `page_title` naamak ek function define karte hain aur use `async` keyword se mark karte hain. Phir hum `trpl::get` function ka istemal URL ko fetch karne ke liye karte hain aur response ka intezaar karne ke liye `await` keyword jodte hain. Response ka text prapt karne ke liye, hum uske `text` method ko call karte hain, aur ek baar phir `await` keyword ke saath uska intezaar karte hain.

Humein in dono futures ka spasht roop se `await` karna hoga, kyunki Rust mein futures **lazy** hote hain: ve tab tak kuch nahi karte jab tak aap unse `await` keyword ke saath kuch karne ke liye na kahein. (Vastav mein, agar aap future ka istemal nahi karte hain to Rust ek compiler warning dikhayega.)

Ek baar jab humein `response_text` mil jaata hai, to hum use `Html::parse` ka istemal karke `Html` type ke ek instance mein parse kar sakte hain. Hum `"title"` string pass karke document mein pehla `<title>` element prapt karne ke liye `select_first` method ka istemal kar sakte hain. Ant mein, hum `inner_html` call karke title ka content prapt karte hain, jo ek `String` hai.

Dhyan dein ki Rust ka `await` keyword us expression ke *baad* aata hai jiska aap intezaar kar rahe hain, na ki usse pehle. Yah ek **postfix** keyword hai. Iske parinaamsvaroop, hum `page_title` ke body ko `trpl::get` aur `text` function calls ko ek saath `await` ke saath chain karne ke liye badal sakte hain, jaisa ki Listing 17-2 mein dikhaya gaya hai.

```rust
// Filename: src/main.rs (chained version)
async fn page_title(url: &str) -> Option<String> {
    let response_text = trpl::get(url).await.text().await;
    // ...
}
```

*Listing 17-2: `await` keyword ke saath chaining*

Jab Rust `async` keyword se mark kiya gaya ek block dekhta hai, to vah use ek anokhe, anonymous data type mein compile karta hai jo `Future` trait ko implement karta hai. Is prakaar, `async fn` likhna ek aise function likhne ke barabar hai jo return type ka ek *future* return karta hai.

-----

### Ek Single Page ka Title Pata Karna

Shuruaat mein, hum sirf ek hi page ke liye title prapt karenge. Lekin, agar hum `main` function mein `await` ka istemal karne ki koshish karte hain, to code compile nahi hoga. Hum vishesh `main` function ko `async` ke roop mein mark nahi kar sakte.

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

`main` ko `async` mark na kar paane ka kaaran yah hai ki async code ko ek **runtime** ki zaroorat hoti hai: ek Rust crate jo asynchronous code ko execute karne ke details ko manage karta hai. Ek program ka `main` function ek runtime ko *initialize* kar sakta hai, lekin vah khud ek runtime *nahi* hai.

Kai bhashayein jo async support karti hain, ek runtime bundle karti hain, lekin Rust nahi karta. Iske bajaye, kai alag-alag async runtimes uplabdh hain. Yahan, hum `trpl` crate se `run` function ka istemal karenge, jo ek future ko argument ke roop mein leta hai aur use poora hone tak chalata hai. Parde ke peeche, `run` call karna ek runtime set up karta hai.

Isliye hum apne logic ko ek `async` block mein daalenge aur use `trpl::run` ko pass karenge, jaisa ki Listing 17-4 mein hai.

```rust,should_panic,noplayground
// Filename: src/main.rs
# use trpl::{Html, get};
# async fn page_title(url: &str) -> Option<String> {
#     let text = get(url).await.text().await;
#     Html::parse(&text)
#         .select_first("title")
#         .map(|title| title.inner_html())
# }
fn main() {
    let url = &std::env::args().nth(1).unwrap();
    trpl::run(async {
        let title = page_title(url).await;
        match title {
            Some(title) => println!("Title for {url} was\n\t{title}"),
            None => println!("Couldn't find a title for {url}"),
        }
    });
}
```

*Listing 17-4: `trpl::run` ke saath ek async block ka `await` karna*

Jab hum is code ko chalate hain, to humein apekshit vyavhaar milta hai:

```console
$ cargo run -- https://www.rust-lang.org
The title for https://www.rust-lang.org was
        Rust Programming Language
```

Har **await point**—yaani, har jagah jahan code `await` keyword ka istemal karta hai—ek aisi jagah hai jahan control runtime ko vaapas saunp diya jaata hai. Ise kaam karne ke liye, Rust compiler svachaalit roop se async code ke liye ek adrishya **state machine** banata aur manage karta hai. Antतः, kisi cheez ko is state machine ko execute karna hota hai, aur vah cheez ek runtime hai.

-----

### Do URLs ko Ek Doosre ke Khilaaf Race Karvana

Listing 17-5 mein, hum command line se pass kiye gaye do alag-alag URLs ke saath `page_title` ko call karte hain aur unki race karvate hain.

```rust,should_panic,noplayground
// Filename: src/main.rs
use trpl::{Either, Html};
//... (async fn page_title definition) ...

fn main() {
    let url1 = std::env::args().nth(1).unwrap();
    let url2 = std::env::args().nth(2).unwrap();

    trpl::run(async {
        let title_fut_1 = page_title(&url1);
        let title_fut_2 = page_title(&url2);

        let (title, url) = match trpl::race(title_fut_1, title_fut_2).await {
            Either::Left((title, url)) => (title, url),
            Either::Right((title, url)) => (title, url),
        };
        
        match title {
            Some(title) => println!("'{}' won the race with the title:\n\t{}", url, title),
            None => println!("'{}' won the race, but it had no title", url),
        }
    });
}
// Note: We also updated page_title to return `(Option<String>, &str)`
```

*Listing 17-5: Do URLs ke titles ki race karvana*

Hum har URL ke liye `page_title` call karke parinaami futures ko save karte hain. Yaad rakhein, ye abhi tak kuch nahi karte, kyunki futures lazy hote hain. Phir hum futures ko `trpl::race` mein pass karte hain, jo yah batane ke liye ek value return karta hai ki usmein pass kiye gaye futures mein se kaun sa pehle poora hota hai.

`race` `trpl::Either` naamak ek naya type return karta hai. `Either` type `Result` ke samaan hai, ismein do cases hote hain. `Result` ke vipreet, `Either` mein safalta ya asafalta ki koi dhaarana nahi hoti. Iske bajaye, yah "ek ya doosra" batane ke liye `Left` aur `Right` ka upyog karta hai.

Aapne ab ek chhota sa kaam karne wala web scraper bana liya hai\! Kuch URLs chunein aur command line tool chalayein. Aapne futures ke saath kaam karne ke mool baatein seekh li hain, isliye ab hum async ke saath kya kar sakte hain, ismein aur gehraai se ja sakte hain.