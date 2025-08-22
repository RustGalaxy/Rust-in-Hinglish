# `pub` se Visibility Control

Humne **Listing 7-5** ki errors ko `network` aur `network::server` ko respectively `src/network/mod.rs` aur `src/network/server.rs` files mein move karke solve kiya. `cargo build` to successfully run ho gaya tha, lekin ab bhi warnings aa rahi thi ki `client::connect`, `network::connect` aur `network::server::connect` functions use nahi ho rahe hain.

Ye warnings kyu aayi? Hum to library bana rahe hain jiske functions users ke liye hain, na ki hamare apne project ke andar use karne ke liye. To in `connect` functions ka unused rehna matter nahi karna chahiye.

Is problem ko samajhne ke liye, hum `communicator` library ko ek alag project se call karke dekhte hain. Uske liye, hum *src/main.rs* file banakar ek binary crate banayenge.

```rust,ignore
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

Yahan `extern crate` command se hum `communicator` library ko scope mein laate hain. Ab hamare package mein do crates hain. `cargo` *src/main.rs* ko ek binary crate ka root file maanta hai, jo library crate (*src/lib.rs*) se alag hai. Ye pattern kaafi common hai: **most functionality library crate mein hoti hai, aur binary crate us library ko use karta hai.** Isse dusre programs bhi library crate use kar sakte hain aur kaam divide ho jaata hai.

`communicator` library ke bahar se dekha jaye to, hamare saare modules us module ke andar hain jiska naam crate ke naam jaisa hai, yani `communicator`. Crate ke top-level module ko hum **root module** kehte hain.

Agar hum kisi external crate ko apne project ke submodule ke andar bhi use kar rahe hain, to `extern crate` command hamesha root module (*src/main.rs* ya *src/lib.rs*) mein jaana chahiye. Fir hum apne submodules mein external crates se items ko top-level modules ki tarah refer kar sakte hain.

Ab hamari binary crate library ke `connect` function ko `client` module se call karti hai. Lekin `cargo build` chalane par warnings ke baad ek error bhi aayegi:

```text
error[E0603]: module `client` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Ye error bolti hai ki `client` module private hai, aur yahi warnings ka **main reason** hai. Rust mein saara code by default private hota hai. Koi bhi dusra code use use nahi kar sakta. Agar aap koi private function apne program mein use nahi karte hain, to Rust aapko warn karega ki wo function unused hai.

Jab aap `client::connect` jaise kisi function ko **public** kar dete hain, to na sirf aap use apni binary crate se call kar sakte hain, balki **unused warning** bhi chali jaayegi. Function ko public karne se Rust ko pata chalta hai ki us function ko bahar ka code use karega, aur Rust is external usage ko us function ka "use" maanta hai.

-----

### Function ko Public Banana

Kisi function ko public banane ke liye, hum uske declaration ke aage **`pub` keyword** lagate hain. Hum `client::connect` ki unused warning aur `module client is private` error ko fix karte hain. *src/lib.rs* ko change karke `client` module ko public banao:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
pub mod client;

mod network;
```

Ab `pub` keyword `mod` se pehle lagaya hai. Ab build karke dekho:

```text
error[E0603]: function `connect` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Hooray\! Ab ek alag error aayi hai. Ye **`function connect is private`** bol rahi hai, to chalo *src/client.rs* mein `client::connect` ko bhi public bana dete hain:

\<span class="filename"\>Filename: src/client.rs\</span\>

```rust
pub fn connect() {
}
```

Ab `cargo build` run karo:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

Code compile ho gaya, aur `client::connect` ki warning chali gayi\!

Unused code warnings hamesha ye nahi batati ki item ko public banana hai. Agar aap in functions ko apni **public API** ka part nahi banana chahte the, to ye warnings aapko bata sakti hain ki aapko ab is code ki zaroorat nahi hai aur aap use safely delete kar sakte hain. Ya fir, agar aapne galti se library mein se is function ke saare calls hata diye hain, to ye warning ek bug bhi ho sakti hai.

Lekin is case mein, hum chahte hain ki baaki do functions bhi public API ka part bane, to unhe bhi `pub` mark karte hain. *src/network/mod.rs* ko change karo:

\<span class="filename"\>Filename: src/network/mod.rs\</span\>

```rust,ignore
pub fn connect() {
}

mod server;
```

Ab compile karo:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / pub fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

Humm, ab bhi unused function ki warning hai, even though `network::connect` ko `pub` set kar diya hai. Kyunki ye function module ke andar public hai, lekin `network` module, jismein ye function hai, **public nahi hai**. Humein *src/lib.rs* ko change karke `network` ko bhi public karna hoga:

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
pub mod client;

pub mod network;
```

Ab compile karne par, wo warning chali jaayegi. Bas ek warning bachi hai - wo khud se fix karne ki koshish karo\!

-----

## Privacy Rules

Overall, visibility ke rules simple hain:

  * Agar koi item **public** hai, to uske **kisi bhi parent module** se access kiya ja sakta hai.
  * Agar koi item **private** hai, to use sirf uske **immediate parent module** aur us parent ke **child modules** se hi access kiya ja sakta hai.

-----

## Privacy Examples

Chalo kuch aur privacy examples dekhte hain. Ek naya library project banao aur **Listing 7-6** ka code uske *src/lib.rs* mein daalo.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust,ignore
mod outermost {
    pub fn middle_function() {}

    fn middle_secret_function() {}

    mod inside {
        pub fn inner_function() {}

        fn secret_function() {}
    }
}

fn try_me() {
    outermost::middle_function();
    outermost::middle_secret_function();
    outermost::inside::inner_function();
    outermost::inside::secret_function();
}
```

Compile karne se pehle guess karo ki `try_me` function ki kaun si lines mein errors aayegi.

#### Errors ko Samjho

`try_me` function hamare project ke root module mein hai. `outermost` module private hai, lekin dusra privacy rule kehta hai ki `try_me` function `outermost` module ko access kar sakta hai, kyunki `outermost` aur `try_me` dono current (root) module mein hain.

  * `outermost::middle_function()` call work karega, kyunki `middle_function` public hai aur `try_me` uske parent module `outermost` se use access kar raha hai, jo ki accessible hai. ✅
  * `outermost::middle_secret_function()` call **compilation error** dega. Kyunki `middle_secret_function` private hai. Root module na to `middle_secret_function` ka current module (`outermost` hai), aur na hi uska child module hai. ❌
  * `inside` module private hai aur uske koi child modules nahi hain, isliye use sirf uska current module `outermost` hi access kar sakta hai. Iska matlab hai ki `try_me` function `outermost::inside::inner_function()` ya `outermost::inside::secret_function()` ko call nahi kar sakta. ❌

#### Errors ko Fix Karna

Yahan kuch suggestions hain code ko fix karne ke liye. Guess karo ki kya ye errors fix karenge ya nahi, aur fir compile karke dekho:

  * Agar `inside` module public hota to kya hota?
  * Agar `outermost` public hota aur `inside` private hota to kya hota?
  * Agar `inner_function` ki body mein aap `::outermost::middle_secret_function()` call karte to kya hota? (Starting mein `::` ka matlab hai ki aap root module se refer karna chahte hain.)

Aage hum `use` keyword se items ko scope mein laane ke baare mein baat karenge.
