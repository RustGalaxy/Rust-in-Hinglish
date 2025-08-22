## Macros

Humne is kitaab mein `println!` jaise macros ka istemal kiya hai, lekin humne poori tarah se yeh explore nahi kiya hai ki macro kya hai aur yeh kaise kaam karta hai. **Macro** term Rust mein features ke ek parivaar ko refer karta hai: `macro_rules!` ke saath **declarative** macros aur teen tarah ke **procedural** macros:

  * Custom `#[derive]` macros jo `derive` attribute ke saath jode gaye code ko specify karte hain, jinka istemal structs aur enums par hota hai.
  * Attribute-like macros jo custom attributes define karte hain jinka istemal kisi bhi item par kiya ja sakta hai.
  * Function-like macros jo function calls jaise dikhte hain lekin unke argument ke roop mein diye gaye tokens par operate karte hain.

Hum in sab ke baare mein ek-ek karke baat karenge, lekin pehle, chaliye dekhte hain ki jab hamare paas functions hain toh humein macros ki zaroorat hi kyun hai.

-----

### Macros aur Functions ke Beech ka Fark

Mool roop se, macros code likhne ka ek tareeka hai jo doosra code likhta hai, jise **metaprogramming** kehte hain. `derive` attribute, jise humne pehle dekha hai, aapke liye alag-alag traits ka implementation generate karta hai. Humne `println!` aur `vec!` macros ka bhi istemal kiya hai. Yeh sabhi macros aapke dwara manually likhe gaye code se zyada code produce karne ke liye **expand** hote hain.

Metaprogramming aapke dwara likhe aur maintain kiye jaane wale code ki matra ko kam karne ke liye upyogi hai, jo ki functions ka bhi ek kaam hai. Lekin, macros ke paas kuch atirikt shaktiyan hoti hain jo functions ke paas nahi hoti.

Ek function signature ko yeh declare karna hota hai ki function ke paas kitne aur kis type ke parameters hain. Doosri taraf, macros ek **variable number of parameters** le sakte hain: hum `println!("hello")` ko ek argument ke saath ya `println!("hello {}", name)` ko do arguments ke saath call kar sakte hain. Saath hi, macros compiler dwara code ka arth samjhne se **pehle** expand hote hain, isliye ek macro, udaharan ke liye, diye gaye type par ek trait implement kar sakta hai. Ek function aisa nahi kar sakta, kyunki use runtime par call kiya jaata hai aur ek trait ko compile time par implement karne ki zaroorat hoti hai.

Ek function ke bajaye macro implement karne ka nuksaan yeh hai ki macro definitions function definitions se zyada jatil hoti hain kyunki aap Rust code likh rahe hain jo Rust code likhta hai. Is indirection ke kaaran, macro definitions aam taur par function definitions se padhne, samjhne aur maintain karne mein adhik kathin hoti hain.

Macros aur functions ke beech ek aur mahatvapurn antar yeh hai ki aapko macros ko file mein call karne se **pehle** unhe define karna ya scope mein laana hota hai, jabki functions ko aap kahin bhi define aur call kar sakte hain.

-----

### General Metaprogramming ke liye `macro_rules!` ke Saath Declarative Macros

Rust mein macros ka sabse zyada istemal hone wala roop **declarative macro** hai. Inhe kabhi-kabhi "macros by example," "`macro_rules!` macros," ya sirf "macros" bhi kaha jaata hai. Apne core mein, declarative macros aapko Rust ke `match` expression jaisa kuch likhne ki anumati dete hain. Macros bhi ek value ko patterns ke saath compare karte hain jo khaas code se jude hote hain: is sthiti mein, value macro ko pass kiya gaya literal Rust source code hota hai; patterns us source code ki sanrachna se compare kiye jaate hain; aur har pattern se juda code, match hone par, macro ko pass kiye gaye code ko replace kar deta hai. Yeh sab compilation ke dauran hota hai.

Ek macro define karne ke liye, aap `macro_rules!` construct ka istemal karte hain. Chaliye dekhte hain ki `vec!` macro kaise define kiya jaata hai. Listing 20-35 `vec!` macro ki thodi si saral ki gayi definition dikhata hai.

\<Listing number="20-35" file-name="src/lib.rs" caption="`vec!` macro definition ka ek saral version"\>

```rust,noplayground
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

\</Listing\>

`#[macro_export]` annotation batata hai ki jab bhi is macro ko define karne wala crate scope mein laaya jaata hai, toh yeh macro uplabdh ho jaana chahiye.

Hum macro definition `macro_rules!` aur macro ke naam (bina exclamation mark ke) ke saath shuru karte hain. Yahan ek arm hai jiska pattern `( $( $x:expr ),* )` hai, jiske baad `=>` aur is pattern se juda code block hai.

Chaliye is pattern ke tukdon ka matlab samjhein:

  * Pehle, hum poore pattern ko enclose karne ke liye parentheses ka istemal karte hain.
  * Hum ek dollar sign (`$`) ka istemal karte hain.
  * Iske baad parentheses ka ek set hai jo `$()` ke andar ke pattern se match hone wali values ko capture karta hai.
  * `$()` ke andar `$x:expr` hai, jo kisi bhi Rust expression se match karta hai aur us expression ko `$x` naam deta hai.
  * `$()` ke baad wala comma batata hai ki har match ke beech ek literal comma separator aana chahiye.
  * `*` specify karta hai ki pattern apne se pehle aane wali cheez ke zero ya adhik baar match karta hai.

Jab hum is macro ko `vec![1, 2, 3];` ke saath call karte hain, toh `$x` pattern teen baar teen expressions `1`, `2`, aur `3` ke saath match karta hai.

Ab is arm se jude code ke body mein pattern ko dekhein: `$()*` ke andar `temp_vec.push()` har us hisse ke liye generate hota hai jo pattern mein `$()` se match hota hai. `$x` har match kiye gaye expression se replace ho jaata hai. Jab hum is macro ko `vec![1, 2, 3];` ke saath call karte hain, toh is macro call ko replace karne wala generated code yeh hoga:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

-----

### Attributes se Code Generate Karne ke liye Procedural Macros

Macros ka doosra roop **procedural macro** hai, jo ek function ki tarah kaam karta hai. Procedural macros kuch code ko input ke roop mein accept karte hain, us code par operate karte hain, aur kuch code ko output ke roop mein produce karte hain. Teen tarah ke procedural macros hote hain: custom `derive`, attribute-like, aur function-like.

Procedural macros banate samay, definitions ko unke apne crate mein ek special crate type ke saath rehna chahiye. Ek procedural macro define karne wala function ek `TokenStream` ko input ke roop mein leta hai aur ek `TokenStream` ko output ke roop mein produce karta hai. `TokenStream` type `proc_macro` crate dwara define kiya gaya hai jo Rust ke saath aata hai aur tokens ke sequence ko represent karta hai.

-----

### Ek Custom `derive` Macro Kaise Likhein

Chaliye ek `hello_macro` naam ka crate banate hain jo `HelloMacro` naam ka ek trait define karta hai. Hum apne users ko har type ke liye `HelloMacro` trait implement karwane ke bajaye, ek procedural macro pradan karenge taaki users apne type ko `#[derive(HelloMacro)]` se annotate karke `hello_macro` function ka default implementation prapt kar sakein.

Ek user hamare crate ka istemal karke is tarah ka code likh payega (Listing 20-37):

\<Listing number="20-37" file-name="src/main.rs" caption="Hamare procedural macro ka istemal karte samay ek user jo code likh payega"\>

```rust,ignore,does_not_compile
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}
```

\</Listing\>

Jab hum kaam poora kar lenge, toh yeh code `Hello, Macro! My name is Pancakes!` print karega.

Agle kadam procedural macro ko define karna hai. Is samay, procedural macros ko unke apne crate mein hona chahiye. Convention ke anusaar, `foo` naam ke crate ke liye, ek custom `derive` procedural macro crate ko `foo_derive` kaha jaata hai. Chaliye apne `hello_macro` project ke andar `hello_macro_derive` naam ka ek naya crate shuru karte hain.

Humein `hello_macro_derive` crate ko ek procedural macro crate ke roop mein declare karna hoga. Humein `syn` aur `quote` crates se functionality ki bhi zaroorat hogi. `hello_macro_derive` ke liye `Cargo.toml` file mein nimnalikhit jodein:

```toml
[lib]
proc-macro = true

[dependencies]
syn = "2.0"
quote = "1.0"
```

Procedural macro define karna shuru karne ke liye, Listing 20-40 mein diye gaye code ko apne `hello_macro_derive` crate ke *src/lib.rs* file mein rakhein.

\<Listing number="20-40" file-name="hello\_macro\_derive/src/lib.rs" caption="Code jo zyada-tar procedural macro crates ko Rust code process karne ke liye zaroori hoga"\>

```rust,ignore,does_not_compile
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a representation of Rust code as a syntax tree
    // that we can manipulate
    let ast = syn::parse(input).unwrap();

    // Build the trait implementation
    impl_hello_macro(&ast)
}
```

\</Listing\>

Humne teen naye crates introduce kiye hain: `proc_macro`, `syn`, aur `quote`.

  * `proc_macro` crate Rust ke saath aata hai aur yeh compiler ka API hai jo humein apne code se Rust code ko padhne aur manipulate karne deta hai.
  * `syn` crate ek string se Rust code ko ek data structure mein parse karta hai jis par hum operations kar sakte hain.
  * `quote` crate `syn` data structures ko wapas Rust code mein badal deta hai.

`hello_macro_derive` function tab call kiya jaayega jab koi user apne type par `#[derive(HelloMacro)]` specify karega. Yeh isliye sambhav hai kyunki humne `hello_macro_derive` function ko `proc_macro_derive` se annotate kiya hai aur `HelloMacro` naam specify kiya hai, jo hamare trait naam se match karta hai.

Ab jab hamare paas annotated Rust code ko `TokenStream` se `DeriveInput` instance mein badalne ke liye code hai, chaliye woh code generate karte hain jo annotated type par `HelloMacro` trait ko implement karta hai, jaisa ki Listing 20-42 mein dikhaya gaya hai.

\<Listing number="20-42" file-name="hello\_macro\_derive/src/lib.rs" caption="Parsed Rust code ka istemal karke `HelloMacro` trait ko implement karna"\>

```rust,ignore
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}
```

\</Listing\>

Hum `ast.ident` ka istemal karke annotated type ka naam (identifier) wala ek `Ident` struct instance prapt karte hain. `quote!` macro humein woh Rust code define karne deta hai jise hum return karna chahte hain. `quote!` macro kuch templating mechanics bhi pradan karta hai: hum `#name` enter kar sakte hain, aur `quote!` use `name` variable ki value se replace kar dega.

Yahan istemal kiya gaya `stringify!` macro Rust mein built-in hai. Yeh ek Rust expression leta hai aur compile time par us expression ko ek string literal mein badal deta hai.

-----

### Attribute-Like Macros

**Attribute-like macros** custom `derive` macros ke samaan hain, lekin `derive` attribute ke liye code generate karne ke bajaye, yeh aapko naye attributes banane ki anumati dete hain. Yeh zyada flexible bhi hain: `derive` sirf structs aur enums ke liye kaam karta hai; attributes doosre items par bhi lagaye ja sakte hain, jaise ki functions. Ek web application framework ka istemal karte samay functions ko annotate karne wale `route` naam ke attribute ka ek udaharan yahan hai:

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Is `#[route]` attribute ko framework dwara ek procedural macro ke roop mein define kiya jaayega. Macro definition function ka signature is tarah dikhega:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Yahan, hamare paas do `TokenStream` type ke parameters hain. Pehla attribute ke contents ke liye hai: `GET, "/"` wala hissa. Doosra us item ki body hai jis par attribute laga hai: is case mein, `fn index() {}` aur function ki baaki body.

-----

### Function-Like Macros

**Function-like macros** aise macros define karte hain jo function calls jaise dikhte hain. `macro_rules!` macros ki tarah, yeh functions se zyada flexible hote hain. Lekin, `macro_rules!` macros ko sirf match-like syntax ka istemal karke define kiya ja sakta hai. Function-like macros ek `TokenStream` parameter lete hain, aur unki definition us `TokenStream` ko Rust code ka istemal karke manipulate karti hai. Ek function-like macro ka ek udaharan `sql!` macro hai jise is tarah call kiya ja sakta hai:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Yeh macro apne andar ke SQL statement ko parse karega aur check karega ki yeh syntactically sahi hai ya nahi, jo ki `macro_rules!` macro se kahin zyada jatil processing hai. `sql!` macro is tarah define kiya jaayega:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

-----

## Summary

Waah\! Ab aapke paas apne toolbox mein kuch aise Rust features hain jinka aap shayad aksar istemal na karein, lekin aapko pata hoga ki yeh bahut hi khaas paristhitiyon mein uplabdh hain. Humne kai jatil vishayon ko introduce kiya hai taaki jab aap unhe error message suggestions mein ya doosre logon ke code mein dekhein, toh aap in concepts aur syntax ko pehchaan sakein. Is chapter ko samadhan tak pahunchne ke liye ek reference ke roop mein istemal karein.

Aage, hum is kitaab mein discuss ki gayi har cheez ko practice mein laayenge aur ek aur project karenge\!