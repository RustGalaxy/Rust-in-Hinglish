## Ek Single-Threaded Web Server Banana

Hum ek single-threaded web server ko chala kar shuruaat karenge. Shuru karne se pehle, chaliye web server banane mein istemal hone wale protocols ka ek chhota sa overview dekhte hain. In protocols ki details is kitaab ke scope se bahar hain, lekin ek sankshipt overview aapko zaroori jaankari de dega.

Web server mein do mukhya protocols istemal hote hain: **Hypertext Transfer Protocol (HTTP)** aur **Transmission Control Protocol (TCP)**. Dono protocols *request-response* protocols hain, iska matlab hai ki ek **client** request shuru karta hai aur ek **server** requests ko sunta hai aur client ko response deta hai. Un requests aur responses ka content protocols dwara define kiya jaata hai.

TCP ek lower-level protocol hai jo batata hai ki jaankari ek server se doosre tak kaise pahunchti hai, lekin yeh nahi batata ki woh jaankari kya hai. HTTP, TCP ke upar bana hai aur requests aur responses ke content ko define karta hai. Takneeki roop se HTTP ko doosre protocols ke saath istemal karna sambhav hai, lekin zyada-tar maamlon mein, HTTP apna data TCP par hi bhejta hai. Hum TCP aur HTTP requests aur responses ke raw bytes ke saath kaam karenge.

-----

### TCP Connection ko Sunna

Hamare web server ko ek TCP connection sunne ki zaroorat hai, isliye yeh hamara pehla kaam hoga. Standard library ek `std::net` module pradan karti hai jo humein yeh karne deti hai. Chaliye hamesha ki tarah ek naya project banate hain:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Ab, *src/main.rs* mein Listing 21-1 ka code daalein. Yeh code local address `127.0.0.1:7878` par aane wale TCP streams ke liye sunega. Jab ise ek incoming stream milega, yeh `Connection established!` print karega.

\<Listing number="21-1" file-name="src/main.rs" caption="Incoming streams ke liye sunna aur stream milne par ek message print karna"\>

```rust,no_run
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        println!("Connection established!");
    }
}
```

\</Listing\>

`TcpListener` ka istemal karke, hum `127.0.0.1:7878` address par TCP connections ke liye sun sakte hain. Is address mein, colon se pehle ka hissa ek IP address hai jo aapke computer ko represent karta hai (yeh har computer par samaan hota hai), aur `7878` port hai.

Is scenario mein `bind` function `new` function ki tarah kaam karta hai, yaani yeh ek naya `TcpListener` instance return karega. Function ka naam `bind` isliye hai kyunki networking mein, sunne ke liye ek port se connect hone ko "port se bind karna" kehte hain.

`bind` function ek `$Result<T, E>$` return karta hai, jo batata hai ki binding fail ho sakti hai. Hum abhi ke liye errors handle karne ki chinta nahi karenge; iske bajaye, hum `unwrap` ka istemal karenge taaki error aane par program ruk jaaye.

`TcpListener` par `incoming` method ek iterator return karta hai jo humein streams ka ek sequence deta hai (khaas taur par, `TcpStream` type ke streams). Ek single **stream** client aur server ke beech ek open connection ko represent karta hai. Ek **connection** poore request aur response process ka naam hai. Isliye, hum `TcpStream` se padhenge yeh dekhne ke liye ki client ne kya bheja hai aur phir client ko data wapas bhejne ke liye stream par apna response likhenge.

Chaliye is code ko chala kar dekhte hain\! Terminal mein `cargo run` likhein aur phir ek web browser mein *127.0.0.1:7878* load karein. Browser mein ek error message dikhega jaise “Connection reset” kyunki server abhi koi data wapas nahi bhej raha hai. Lekin jab aap apne terminal ko dekhenge, toh aapko kai messages dikhenge jo tab print hue jab browser server se connect hua\!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Kabhi-kabhi aapko ek browser request ke liye multiple messages print hote dikhenge; iska kaaran ho sakta hai ki browser page ke saath-saath doosre resources, jaise ki browser tab mein dikhne wala *favicon.ico* icon, ke liye bhi request kar raha hai.

Sabse zaroori baat yeh hai ki humne safaltapoorvak ek TCP connection ka handle prapt kar liya hai\!

Jab aap code ke ek particular version ko chala kar poora kar lein, toh program ko \<kbd\>ctrl\</kbd\>-\<kbd\>C\</kbd\> daba kar rokna na bhoolein.

-----

### Request ko Padhna

Chaliye browser se request padhne ki functionality implement karte hain\! Hum connections process karne ke liye ek naya function shuru karenge. Is naye `handle_connection` function mein, hum TCP stream se data padhenge aur use print karenge. Code ko Listing 21-2 jaisa badlein.

\<Listing number="21-2" file-name="src/main.rs" caption="`TcpStream` se padhna aur data print karna"\>

```rust,no_run
use std::{
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Request: {:#?}", http_request);
}
```

\</Listing\>

`handle_connection` function mein, hum ek naya `BufReader` instance banate hain. Hum `http_request` naam ka ek variable banate hain taaki browser dwara server ko bheje gaye request ki lines ko collect kar sakein.

Browser ek HTTP request ke ant ka sanket do newline characters ek ke baad ek bhej kar deta hai, isliye stream se ek request paane ke liye, hum tab tak lines lete hain jab tak humein ek empty string wali line na mil jaaye.

Chaliye is code ko aazmate hain\! Program shuru karein aur web browser mein ek request karein. Aapke terminal mein program ka output ab kuch is tarah dikhega:

```console
$ cargo run
   ...
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 ...",
    "Accept: text/html,...",
    ...
]
```

Aapke browser ke aadhar par, aapko thoda alag output mil sakta hai. Chaliye is request data ko thoda tod kar samjhein ki browser hamare program se kya pooch raha hai.

#### HTTP Request par ek Gehri Nazar

HTTP ek text-based protocol hai, aur ek request ka format is prakaar hota hai:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

Pehli line **request line** hai jismein client kya request kar raha hai uske baare mein jaankari hoti hai. Iska pehla hissa istemal kiya ja raha **method** batata hai, jaise `GET` ya `POST`. Hamare client ne ek `GET` request ka istemal kiya, jiska matlab hai ki woh jaankari maang raha hai.

Request line ka agla hissa `/` hai, jo client dwara request kiye gaye **URI (Uniform Resource Identifier)** ko batata hai. Aakhri hissa client dwara istemal kiya gaya HTTP version hai, aur phir request line ek CRLF sequence (`\r\n`) mein samapt hoti hai.

Baaki lines jo `Host:` se shuru ho rahi hain, woh headers hain. `GET` requests mein koi body nahi hoti.

Ab jab hum jaante hain ki browser kya maang raha hai, chaliye kuch data wapas bhejte hain\!

-----

### Ek Response Likhna

Hum client ke request ke jawab mein data bhejne ki functionality implement karne ja rahe hain. Responses ka format nimnalikhit hota hai:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

Pehli line ek **status line** hai jismein response mein istemal kiya gaya HTTP version, ek numeric status code jo request ke parinaam ko summarize karta hai, aur ek reason phrase jo status code ka text description pradan karta hai, shaamil hai.

Yahan ek chhota sa safal HTTP response hai. Status code 200 standard success response hai. Chaliye ise stream par likhte hain\! `handle_connection` function se, `println!` hatayein jo request data print kar raha tha aur use Listing 21-3 ke code se badal dein.

\<Listing number="21-3" file-name="src/main.rs" caption="Stream par ek chhota sa safal HTTP response likhna"\>

```rust,no_run
// in handle_connection
let status_line = "HTTP/1.1 200 OK";
let contents = "";
let response = format!("{status_line}\r\nContent-Length: 0\r\n\r\n{contents}");
stream.write_all(response.as_bytes()).unwrap();
```

\</Listing\>

`stream` par `write_all` method ek `&[u8]` leta hai aur un bytes ko seedhe connection par bhej deta hai.

In badlavon ke saath, chaliye apna code chalayein aur ek request karein. Jab aap web browser mein *127.0.0.1:7878* load karenge, toh aapko ek error ke bajaye ek khali page milna chahiye. Aapne abhi-abhi ek HTTP request receive karne aur ek response bhejne ka code haath se likha hai\!

-----

### Asli HTML Return Karna

Chaliye ek khali page se zyada return karne ki functionality implement karte hain. Apne project directory ke root mein ek nayi file *hello.html* banayein, *src* directory mein nahi. Aap koi bhi HTML daal sakte hain; Listing 21-4 ek sambhavna dikhata hai.

\<Listing number="21-4" file-name="hello.html" caption="Response mein return karne ke liye ek sample HTML file"\>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Hello!</h1>
    <p>Hi from Rust</p>
  </body>
</html>
```

\</Listing\>

Ise server se return karne ke liye, hum `handle_connection` ko Listing 21-5 mein dikhaye gaye anusaar modify karenge taaki HTML file ko padha ja sake, use response mein body ke roop mein joda ja sake, aur bheja ja sake.

\<Listing number="21-5" file-name="src/main.rs" caption="*hello.html* ke contents ko response ki body ke roop mein bhejna"\>

```rust,no_run
use std::fs;
// ... in handle_connection
let status_line = "HTTP/1.1 200 OK";
let contents = fs::read_to_string("hello.html").unwrap();
let length = contents.len();

let response =
    format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

stream.write_all(response.as_bytes()).unwrap();
```

\</Listing\>

Humne `use` statement mein `fs` joda hai taaki standard library ke filesystem module ko scope mein laaya ja sake.

Iske baad, hum `format!` ka istemal karke file ke contents ko success response ki body ke roop mein jodte hain. Ek valid HTTP response sunishchit karne ke liye, hum `Content-Length` header jodte hain jo hamare response body ke size par set hota hai, is maamle mein `hello.html` ka size.

Is code ko `cargo run` ke saath chalayein aur apne browser mein *127.0.0.1:7878* load karein; aapko apna HTML render hua dikhna chahiye\!

Abhi, hamara server bahut seemit hai. Hum apne responses ko request ke aadhar par customize karna chahte hain aur sirf `/` ke liye ek sahi request par HTML file wapas bhejna chahte hain.

-----

### Request ko Validate Karna aur Chuninda Jawab Dena

Abhi, hamara web server client ke request ki parwah kiye bina file mein HTML return karega. Chaliye yeh check karne ke liye functionality jodte hain ki browser `/` request kar raha hai ya nahi, aur agar browser kuch aur request karta hai toh ek error return karein. Iske liye humein `handle_connection` ko modify karna hoga, jaisa ki Listing 21-6 mein dikhaya gaya hai.

\<Listing number="21-6" file-name="src/main.rs" caption="`/*` ke requests ko doosre requests se alag handle karna"\>

```rust,no_run
// in handle_connection
let buf_reader = BufReader::new(&mut stream);
let request_line = buf_reader.lines().next().unwrap().unwrap();

if request_line == "GET / HTTP/1.1" {
    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
} else {
    // some other request
}
```

\</Listing\>

Hum `request_line` ko check karte hain yeh dekhne ke liye ki kya yeh `/` path ke liye ek GET request ke barabar hai. Agar hai, toh `if` block hamari HTML file ke contents return karta hai.

Ab chaliye `else` block mein Listing 21-7 ka code jodte hain taaki status code 404 ke saath ek response return kiya ja sake, jo batata hai ki request ke liye content nahi mila.

\<Listing number="21-7" file-name="src/main.rs" caption="Status code 404 aur ek error page ke saath jawab dena agar `/*` ke alawa kuch aur request kiya gaya ho"\>

```rust,no_run
// in the else block
let status_line = "HTTP/1.1 404 NOT FOUND";
let contents = fs::read_to_string("404.html").unwrap();
let length = contents.len();

let response =
    format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

stream.write_all(response.as_bytes()).unwrap();
```

\</Listing\>

Yahan, hamare response mein status line mein status code 404 aur reason phrase `NOT FOUND` hai. Response ki body file *404.html* mein HTML hogi. Aapko error page ke liye *hello.html* ke paas ek *404.html* file banani hogi.

In badlavon ke saath, apna server phir se chalayein. *127.0.0.1:7878* request karne par *hello.html* ke contents return hone chahiye, aur koi bhi doosra request, jaise *127.0.0.1:7878/foo*, *404.html* se error HTML return karna chahiye.

-----

### Thoda sa Refactoring

Is samay, `if` aur `else` blocks mein bahut repetition hai: dono files padh rahe hain aur files ke contents ko stream par likh rahe hain. Fark sirf status line aur filename mein hai. Chaliye code ko aur concise (sankshipt) banate hain, un differences ko alag `if` aur `else` lines mein daal kar jo status line aur filename ki values ko variables mein assign karenge. Listing 21-9 is bade `if` aur `else` blocks ko replace karne ke baad ka code dikhata hai.

\<Listing number="21-9" file-name="src/main.rs" caption="`if` aur `else` blocks ko refactor karke sirf woh code rakhna jo dono cases ke beech alag hai"\>

```rust,no_run
// in handle_connection
let buf_reader = BufReader::new(&mut stream);
let request_line = buf_reader.lines().next().unwrap().unwrap();

let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
    ("HTTP/1.1 200 OK", "hello.html")
} else {
    ("HTTP/1.1 404 NOT FOUND", "404.html")
};

let contents = fs::read_to_string(filename).unwrap();
let length = contents.len();

let response =
    format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

stream.write_all(response.as_bytes()).unwrap();
```

\</Listing\>

Ab `if` aur `else` blocks sirf status line aur filename ke liye appropriate values ek tuple mein return karte hain; phir hum **destructuring** ka istemal karke in do values ko `status_line` aur `filename` mein assign karte hain.

Pehle jo duplicated code tha, woh ab `if` aur `else` blocks ke bahar hai aur `status_line` aur `filename` variables ka istemal karta hai. Isse dono cases ke beech ka fark dekhna aasan ho jaata hai.

Bahut badhiya\! Ab hamare paas lagbhag 40 lines ke Rust code mein ek saral web server hai jo ek request ka jawab content ke ek page se deta hai aur baaki sabhi requests ka jawab 404 response se deta hai.

Abhi, hamara server ek single thread mein chalta hai, jiska matlab hai ki yeh ek samay mein sirf ek request serve kar sakta hai. Chaliye dekhte hain ki yeh ek samasya kaise ho sakti hai. Phir hum ise theek karenge taaki hamara server ek saath kai requests handle kar sake.