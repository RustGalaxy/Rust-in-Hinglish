## Closures: Anonymous Functions jo Apne Environment ko Capture kar sakte hain 🎯

Rust ke **closures** anonymous functions hote hain jinhein aap ek variable mein save kar sakte hain ya doosre functions mein arguments ke roop mein pass kar sakte hain. Aap ek jagah closure bana sakte hain aur fir use alag context mein evaluate karne ke liye call kar sakte hain. Functions ke ulat, closures us **scope** se values ko capture kar sakte hain jismein unhein call kiya jaata hai. Hum dikhayenge ki kaise ye features code reuse aur behavior customization mein madad karte hain.

-----

### Closures se Behavior ka Abstraction Banana 📝

Aaiye ek example par kaam karte hain jismein ek closure ko baad mein execute karne ke liye store karna useful hai. Iske dauran, hum closures ke syntax, type inference, aur traits ke baare mein baat karenge.

Maan lijiye hum ek startup mein kaam karte hain jo custom exercise workout plans generate karne wala ek app bana rahi hai. Algorithm, jo workout plan generate karta hai, kayi factors par depend karta hai, aur is calculation mein kuch seconds lagte hain. Hum is algorithm ko tabhi call karna chahte hain jab humein iski zaroorat ho aur sirf ek hi baar call karna chahte hain taaki user ko zaroorat se zyaada wait na karna pade.

Hum is **hypothetical algorithm** ko **`simulated_expensive_calculation`** function se simulate karenge, jaisa **Listing 13-1** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
use std::thread;
use std::time::Duration;

fn simulated_expensive_calculation(intensity: u32) -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    intensity
}
```

\<span class="caption"\>Listing 13-1: Ek function jo ek aise calculation ko simulate karta hai jismein 2 seconds lagte hain\</span\>

Ab **`main` function** ko dekhte hain jismein workout app ke important parts hain. Ye function us code ko represent karta hai jise app call karegi jab user workout plan maangega. Hum inputs ko hardcode kar denge.

**Listing 13-2** mein `main` function hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let simulated_user_specified_value = 10;
    let simulated_random_number = 7;

    generate_workout(
        simulated_user_specified_value,
        simulated_random_number
    );
}
# fn generate_workout(intensity: u32, random_number: u32) {}
```

\<span class="caption"\>Listing 13-2: User input aur random number generation ko simulate karne ke liye hardcoded values ke saath ek `main` function\</span\>

`main` function **`generate_workout`** function ko simulated input values ke saath call karta hai.

`generate_workout` function mein app ka business logic hai. **Listing 13-3** mein ye function dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# use std::thread;
# use std::time::Duration;
#
# fn simulated_expensive_calculation(num: u32) -> u32 {
#     println!("calculating slowly...");
#     thread::sleep(Duration::from_secs(2));
#     num
# }
#
fn generate_workout(intensity: u32, random_number: u32) {
    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            simulated_expensive_calculation(intensity)
        );
        println!(
            "Next, do {} situps!",
            simulated_expensive_calculation(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                simulated_expensive_calculation(intensity)
            );
        }
    }
}
```

\<span class="caption"\>Listing 13-3: Business logic jo inputs aur `simulated_expensive_calculation` function calls ke aadhar par workout plans print karta hai\</span\>

`Listing 13-3` ke code mein slow calculation function ko multiple times call kiya gaya hai. Hum `simulated_expensive_calculation` function ko sirf tab call karna chahte hain jab humein result ki zaroorat ho aur sirf ek hi baar call karna chahte hain.

#### Functions ka Upyog karke Refactoring 🔄

Hum `simulated_expensive_calculation` function ke duplicated call ko ek **variable** mein extract kar sakte hain, jaisa **Listing 13-4** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# use std::thread;
# use std::time::Duration;
#
# fn simulated_expensive_calculation(num: u32) -> u32 {
#     println!("calculating slowly...");
#     thread::sleep(Duration::from_secs(2));
#     num
# }
#
fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_result =
        simulated_expensive_calculation(intensity);

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_result
        );
        println!(
            "Next, do {} situps!",
            expensive_result
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_result
            );
        }
    }
}
```

\<span class="caption"\>Listing 13-4: `simulated_expensive_calculation` ke calls ko ek jagah extract karna aur result ko `expensive_result` variable mein store karna\</span\>

Is change se pehle `if` block mein function ko do baar call karne ki problem solve ho gayi hai. Lekin, ab hum is function ko har case mein call kar rahe hain, jismein wo inner `if` block bhi shamil hai jise result ki zaroorat hi nahi hai.

#### Code Store karne ke liye Closures se Refactoring 💾

Humein code ko ek jagah define karna hai, lekin use tabhi **execute** karna hai jab humein actually result ki zaroorat ho. Iske liye **closures** ka use hota hai. Hum `simulated_expensive_calculation` function ko hamesha call karne ke bajaye, ek closure define kar sakte hain aur us closure ko ek variable mein store kar sakte hain, jaisa **Listing 13-5** mein dikhaya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# use std::thread;
# use std::time::Duration;
#
let expensive_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
# expensive_closure(5);
```

\<span class="caption"\>Listing 13-5: Ek closure ko define karna aur use `expensive_closure` variable mein store karna\</span\>

Closure definition `=` ke baad aati hai. Ek closure ko define karne ke liye, hum **vertical pipes (`|`)** se shuru karte hain, jiske andar hum closure ke parameters specify karte hain. Is closure mein ek parameter hai jiska naam `num` hai. Parameters ke baad, hum curly brackets rakhte hain jismein closure ki body hoti hai. Closure body ki last line (`num`) se jo value return hoti hai, wahi closure ko call karne par return hoti hai.

`expensive_closure` mein ab ek anonymous function ki **definition** hai, na ki uske call ki **result value**. Ab hum `if` blocks mein code ko change karke closure ko call kar sakte hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# use std::thread;
# use std::time::Duration;
#
fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_closure = |num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_closure(intensity)
        );
        println!(
            "Next, do {} situps!",
            expensive_closure(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_closure(intensity)
            );
        }
    }
}
```

\<span class="caption"\>Listing 13-6: Define kiye gaye `expensive_closure` ko call karna\</span\>

Ab expensive calculation sirf tab call hota hai jab humein result ki zaroorat hoti hai.

-----

### Closure Type Inference aur Annotation 🖊️

Closures ko `fn` functions ki tarah parameters ya return value ke types ko annotate karne ki zaroorat nahi hoti. Compiler reliably **types ko infer** kar sakta hai.

Jaise variables ke saath, hum closure mein bhi type annotations add kar sakte hain. **Listing 13-7** mein, humne `expensive_closure` mein type annotations add ki hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# use std::thread;
# use std::time::Duration;
#
let expensive_closure = |num: u32| -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

\<span class="caption"\>Listing 13-7: Closure mein parameter aur return value types ke optional type annotations add karna\</span\>

Closure definitions mein har parameter aur return value ke liye ek concrete type infer kiya jaata hai. Ek baar type infer ho jaane ke baad, wo **lock ho jaata hai**. Agar aap `String` aur `u32` ko ek hi closure mein pass karne ki koshish karte hain, to aapko error milegi.

-----

### Generic Parameters aur `Fn` Traits se Closures Store karna 💾

Hum ek **`struct`** bana sakte hain jo closure ko hold karega aur us closure ko execute karke result ko **cache** karega. Is pattern ko **memoization** ya **lazy evaluation** kehte hain.

Closure ko hold karne ke liye, hum **generics** aur **trait bounds** ka use karte hain. **`Fn` traits** standard library dwara provide kiye gaye hain. Saare closures kam se kam ek trait ko implement karte hain: `Fn`, `FnMut`, ya `FnOnce`.

**Listing 13-9** `Cacher` struct ki definition dikhati hai jo `calculation` mein ek closure aur `value` mein ek optional result hold karta hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}
```

\<span class="caption"\>Listing 13-9: Ek `Cacher` struct define karna jo `calculation` mein ek closure aur `value` mein ek optional result hold karta hai\</span\>

`Cacher` struct ke pass ek `calculation` field hai jo generic type `T` ka hai. `T` par trait bound specify karta hai ki ye ek closure hai jismein ek `u32` parameter aur `u32` return type hona chahiye. `value` field `Option<u32>` type ka hai.

**Listing 13-10** mein caching logic define kiya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# struct Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     calculation: T,
#     value: Option<u32>,
# }
#
impl<T> Cacher<T>
    where T: Fn(u32) -> u32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}
```

\<span class="caption"\>Listing 13-10: `Cacher` ka caching logic\</span\>

`Cacher::new` function ek `Cacher` instance return karta hai jismein `calculation` field mein closure aur `value` field mein `None` hota hai. Jab calling code ko result ki zaroorat hoti hai, to wo `value` method ko call karta hai. Ye method pehle check karta hai ki `self.value` mein koi value hai ya nahi; agar hai, to wo use return kar deta hai. Agar nahi hai, to ye closure ko execute karta hai aur result ko `self.value` mein store kar deta hai.

**Listing 13-11** dikhata hai ki hum `Cacher` struct ko `generate_workout` function mein kaise use kar sakte hain.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
# use std::thread;
# use std::time::Duration;
#
# struct Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     calculation: T,
#     value: Option<u32>,
# }
#
# impl<T> Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     fn new(calculation: T) -> Cacher<T> {
#         Cacher {
#             calculation,
#             value: None,
#         }
#     }
#
#     fn value(&mut self, arg: u32) -> u32 {
#         match self.value {
#             Some(v) => v,
#             None => {
#                 let v = (self.calculation)(arg);
#                 self.value = Some(v);
#                 v
#             },
#         }
#     }
# }
#
fn generate_workout(intensity: u32, random_number: u32) {
    let mut expensive_result = Cacher::new(|num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    });

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_result.value(intensity)
        );
        println!(
            "Next, do {} situps!",
            expensive_result.value(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_result.value(intensity)
            );
        }
    }
}
```

\<span class="caption"\>Listing 13-11: `Cacher` ka upyog `generate_workout` function mein caching logic ko abstract karne ke liye\</span\>

Ab hum `Cacher` ke instance ko call karte hain, aur expensive calculation maximum ek baar hi run hoga.

-----

### Closures se Environment Capture karna 🧑‍💻

Closures mein ek additional capability hai jo functions mein nahi hoti: ye apne environment ko capture kar sakte hain aur us scope se variables ko access kar sakte hain jismein unhein define kiya gaya hai. **Listing 13-12** mein, `equal_to_x` closure `x` variable ka upyog karta hai jo closure ke surrounding environment mein define kiya gaya hai.

\<span class="filename"\>Filename: src/main.rs\</span\>

```rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x;

    let y = 4;

    assert!(equal_to_x(y));
}
```

\<span class="caption"\>Listing 13-12: Ek closure ka example jo apne enclosing scope mein ek variable ko refer karta hai\</span\>

Hum functions ke saath aisa nahi kar sakte hain. Compiler humein batata hai ki ye sirf closures ke saath kaam karta hai.

Closures apne environment se values ko teen tareekon se capture kar sakte hain: **ownership lena**, **mutably borrow karna**, aur **immutably borrow karna**. Ye teen tareeke teen `Fn` traits mein encoded hain:

  * `FnOnce` captured variables ki **ownership leta hai** aur closure ko sirf **ek baar** call kiya ja sakta hai.
  * `FnMut` captured variables ko **mutably borrow** kar sakta hai.
  * `Fn` captured variables ko **immutably borrow** kar sakta hai.

Agar aap closure ko environment mein use hone wali values ki **ownership** lene ke liye force karna chahte hain, to aap parameter list se pehle **`move` keyword** ka upyog kar sakte hain.

```rust,ignore
fn main() {
    let x = vec![1, 2, 3];

    let equal_to_x = move |z| z == x;

    println!("can't use x here: {:?}", x);

    let y = vec![1, 2, 3];

    assert!(equal_to_x(y));
}
```

`move` keyword add karne par `x` ki value closure mein move ho jaati hai, aur `main` mein `x` ko use karne ki anumati nahi hai.
