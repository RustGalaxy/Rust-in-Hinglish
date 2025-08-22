## Iterators se Items ki Series ko Process karna 🔄

**Iterator pattern** aapko items ki ek series par ek task perform karne deta hai. Ek iterator ka kaam har item par iterate karne ka logic aur ye determine karne ka logic hai ki sequence kab khatam hua. Jab aap iterators ka upyog karte hain, to aapko ye logic khud se implement nahi karna padta.

Rust mein, iterators **lazy** hote hain, matlab unka tab tak koi asar nahi hota jab tak aap unhein consume karne wale methods call na karein. For example, **Listing 13-13** mein, `v1` vector ke items par **`iter` method** call karke ek iterator banaya gaya hai. Ye code khud kuch nahi karta hai.

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();
```

\<span class="caption"\>Listing 13-13: Ek iterator banana\</span\>

Ek baar jab hum ek iterator bana lete hain, to hum uska upyog kai tareekon se kar sakte hain. **Listing 13-14** mein, iterator ka upyog ek `for` loop mein kiya gaya hai.

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

for val in v1_iter {
    println!("Got: {}", val);
}
```

\<span class="caption"\>Listing 13-14: `for` loop mein ek iterator ka upyog\</span\>

Iterators aapke liye ye sara logic handle kar lete hain, jisse repetitive code kam hota hai aur aapko same logic ko kayi alag-alag tarah ke sequences ke saath use karne ki flexibility milti hai.

-----

### `Iterator` Trait aur `next` Method ➡️

Saare iterators standard library mein define kiye gaye **`Iterator` trait** ko implement karte hain. `Iterator` trait ko implement karne ke liye sirf ek method define karna zaroori hai: **`next` method**, jo ek samay mein iterator ka ek item `Some` mein wrap karke return karta hai aur jab iteration khatam ho jaati hai, to `None` return karta hai.

Hum iterators par `next` method ko directly call kar sakte hain; **Listing 13-15** dikhati hai ki ek vector se banaye gaye iterator par `next` ko baar-baar call karne par kaunsi values return hoti hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

\<span class="caption"\>Listing 13-15: Ek iterator par `next` method ko call karna\</span\>

Gaur karein ki humein `v1_iter` ko **mutable** banana pada: `next` method ko call karne par iterator apni internal state ko change karta hai. `iter` method immutable references par iterator banata hai. Agar hum `v1` ki **ownership** lekar owned values return karne wala iterator banana chahte hain, to hum `iter` ke bajaye **`into_iter`** call kar sakte hain.

-----

### Iterator ko Consume karne wale Methods 🥣

`Iterator` trait mein kayi alag-alag methods hain jo default implementations ke saath aate hain. Kuch methods ko **consuming adaptors** kehte hain, kyunki unhein call karne se iterator use ho jaata hai. Ek example **`sum` method** hai, jo iterator ki ownership leta hai aur har item ko ek total mein jodta hai aur jab iteration complete ho jaati hai to total return karta hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    let total: i32 = v1_iter.sum();

    assert_eq!(total, 6);
}
```

\<span class="caption"\>Listing 13-16: `sum` method ko call karke iterator ke saare items ka total paana\</span\>

`sum` call ke baad hum `v1_iter` ka upyog nahi kar sakte kyunki `sum` uski ownership le leta hai.

-----

### Dusre Iterators Produce karne wale Methods ⛓️

Kuch aur methods ko **iterator adaptors** kehte hain, jo iterators ko alag-alag tarah ke iterators mein badalne dete hain. Aap complex actions ko ek readable tareeke se perform karne ke liye multiple iterator adaptors ko **chain** kar sakte hain. Lekin kyunki saare iterators lazy hote hain, aapko results paane ke liye ek consuming adaptor method call karna padta hai.

**Listing 13-17** `map` iterator adaptor method ko call karne ka ek example dikhati hai, jo har item par call karne ke liye ek closure leta hai aur ek naya iterator produce karta hai. Lekin ye code ek warning deta hai, kyunki ye lazy hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
let v1: Vec<i32> = vec![1, 2, 3];

v1.iter().map(|x| x + 1);
```

\<span class="caption"\>Listing 13-17: Naya iterator banane ke liye `map` iterator adaptor ko call karna\</span\>

Isse `warning: unused` milta hai. Isko fix karne ke liye, hum `collect` method ka upyog karenge. **Listing 13-18** mein, hum `map` se return hone wale iterator par iterate karke results ko ek vector mein collect karte hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

\<span class="caption"\>Listing 13-18: Naya iterator banane ke liye `map` method ko call karna aur fir naye iterator ko consume karke vector banane ke liye `collect` method ko call karna\</span\>

-----

### Apne Environment ko Capture karne wale Closures ka Upyog 🖼️

Ab hum **`filter` iterator adaptor** ka upyog karke closures ka ek common use demonstrate kar sakte hain jo apne environment ko capture karte hain. `filter` method ek closure leta hai jo har item se ek Boolean return karta hai. Agar closure `true` return karta hai, to value `filter` dwara produce kiye gaye iterator mein shamil hogi.

**Listing 13-19** mein, hum `filter` ka upyog ek `Shoe` struct instances ke collection par iterate karne ke liye ek closure ke saath karte hain. Ye closure environment se `shoe_size` variable ko capture karta hai aur sirf us size ke shoes return karta hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
        .filter(|s| s.size == shoe_size)
        .collect()
}

#[test]
fn filters_by_size() {
    let shoes = vec![
        Shoe { size: 10, style: String::from("sneaker") },
        Shoe { size: 13, style: String::from("sandal") },
        Shoe { size: 10, style: String::from("boot") },
    ];

    let in_my_size = shoes_in_my_size(shoes, 10);

    assert_eq!(
        in_my_size,
        vec![
            Shoe { size: 10, style: String::from("sneaker") },
            Shoe { size: 10, style: String::from("boot") },
        ]
    );
}
```

\<span class="caption"\>Listing 13-19: `filter` method ka upyog ek closure ke saath jo `shoe_size` ko capture karta hai\</span\>

-----

### `Iterator` Trait se Apne Khud ke Iterators Banana 👷

Hum `Iterator` trait ko apne khud ke types par implement karke bhi iterators bana sakte hain. Jaisa ki humne pehle zikr kiya, aapko sirf `next` method ke liye ek definition provide karne ki zaroorat hai. Uske baad, aap `Iterator` trait ke saare doosre methods ka upyog kar sakte hain\!

Demonstrate karne ke liye, hum ek iterator banayenge jo sirf 1 se 5 tak ginti karega. Sabse pehle, hum `Counter` struct aur uske liye ek `new` function banayenge, jaisa **Listing 13-20** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}
```

\<span class="caption"\>Listing 13-20: `Counter` struct aur ek `new` function define karna jo `count` ke liye 0 ki initial value ke saath `Counter` ke instances banata hai\</span\>

Ab, hum **Listing 13-21** mein dikhaye gaye `next` method ko define karke `Counter` type ke liye `Iterator` trait ko implement karenge.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# struct Counter {
#     count: u32,
# }
#
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

\<span class="caption"\>Listing 13-21: Hamare `Counter` struct par `Iterator` trait ko implement karna\</span\>

Humne apne iterator ke liye associated `Item` type ko `u32` set kiya hai, jiska matlab hai ki iterator `u32` values return karega. `next` method `count` ko 1 se badhata hai aur agar `count` 6 se kam hai, to `Some(count)` return karta hai, warna `None`.

**Listing 13-22** ek test dikhata hai jo demonstrate karta hai ki hum apne `Counter` struct ki iterator functionality ka upyog `next` method ko directly call karke kar sakte hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# struct Counter {
#     count: u32,
# }
#
# impl Iterator for Counter {
#     type Item = u32;
#
#     fn next(&mut self) -> Option<Self::Item> {
#         self.count += 1;
#
#         if self.count < 6 {
#             Some(self.count)
#         } else {
#             None
#         }
#     }
# }
#
#[test]
fn calling_next_directly() {
    let mut counter = Counter::new();

    assert_eq!(counter.next(), Some(1));
    assert_eq!(counter.next(), Some(2));
    assert_eq!(counter.next(), Some(3));
    assert_eq!(counter.next(), Some(4));
    assert_eq!(counter.next(), Some(5));
    assert_eq!(counter.next(), None);
}
```

\<span class="caption"\>Listing 13-22: `next` method implementation ki functionality ko test karna\</span\>

Ek baar jab hum `Iterator` trait implement kar dete hain, to hum standard library mein define kiye gaye `Iterator` trait ke kisi bhi method ki default implementation ka upyog kar sakte hain. **Listing 13-23** ka test dikhata hai ki hum `zip`, `map`, `filter`, aur `sum` jaise kayi `Iterator` trait methods ka upyog hamare `Counter` iterator par kar sakte hain.

\<span class="filename"\>Filename: src/lib.rs\</span\>

```rust
# struct Counter {
#     count: u32,
# }
#
# impl Counter {
#     fn new() -> Counter {
#         Counter { count: 0 }
#     }
# }
#
# impl Iterator for Counter {
#     // Our iterator will produce u32s
#     type Item = u32;
#
#     fn next(&mut self) -> Option<Self::Item> {
#         // increment our count. This is why we started at zero.
#         self.count += 1;
#
#         // check to see if we've finished counting or not.
#         if self.count < 6 {
#             Some(self.count)
#         } else {
#             None
#         }
#     }
# }
#
#[test]
fn using_other_iterator_trait_methods() {
    let sum: u32 = Counter::new().zip(Counter::new().skip(1))
                                 .map(|(a, b)| a * b)
                                 .filter(|x| x % 3 == 0)
                                 .sum();
    assert_eq!(18, sum);
}
```

\<span class="caption"\>Listing 13-23: Hamare `Counter` iterator par kayi tarah ke `Iterator` trait methods ka upyog\</span\>
