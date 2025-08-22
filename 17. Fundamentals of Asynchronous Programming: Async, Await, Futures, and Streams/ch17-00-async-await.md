## Asynchronous Programming ke Fundamentals: Async, Await, Futures, aur Streams

Computer ko diye gaye kai operations ko poora hone mein waqt lag sakta hai. Yeh accha hota agar hum un lambe chalne wale processes ke poora hone ka intezaar karte hue kuch aur kar paate. Modern computers ek samay mein ek se zyada operation par kaam karne ke liye do techniques pradan karte hain: **parallelism** aur **concurrency**. Lekin jaise hi hum parallel ya concurrent operations wale program likhna shuru karte hain, humein jald hi *asynchronous programming* ki nayi chunautiyon ka saamna karna padta hai, jahan operations shuru hone ke kram (order) mein sequentially poora nahi ho sakte. Yeh chapter Chapter 16 ke threads ke istemal par aadharit hai aur asynchronous programming ke liye ek alag approach pesh karta hai: Rust ke Futures, Streams, unhe support karne wala `async` aur `await` syntax, aur asynchronous operations ko manage aur coordinate karne ke tools.

Chaliye ek udaharan dekhte hain. Maan lijiye aap ek family celebration ka video export kar rahe hain, ek aisa operation jismein minute se lekar ghanton tak lag sakte hain. Video export jitna ho sake utna CPU aur GPU power ka istemal karega. Agar aapke paas sirf ek CPU core hota aur aapka operating system us export ko poora hone tak pause nahi karta—yaani, agar vah export ko *synchronously* execute karta—to aap us task ke chalte hue apne computer par kuch aur nahi kar paate. Yeh kaafi niraashaajanak anubhav hota. Khushkismati se, aapka computer ka operating system, export ko itni baar अदृश्य roop se interrupt kar sakta hai, aur karta hai, ki aap ek saath doosre kaam kar paate hain.

Ab maan lijiye aap kisi aur dwara share kiya gaya video download kar rahe hain, jismein bhi waqt lag sakta hai lekin utna CPU time nahi lagta. Is case mein, CPU ko network se data aane ka intezaar karna padta hai. Bhale hi aap data aane par use padhna shuru kar sakte hain, lekin poora data aane mein kuch samay lag sakta hai. Yahan tak ki jab poora data aa jaata hai, agar video kaafi bada hai, to use poora load karne mein kam se kam ek ya do second lag sakte hain. Shayad yeh zyada na lage, lekin ek modern processor ke liye yeh bahut lamba samay hai, jo har second mein arabon operations kar sakta hai. Fir se, aapka operating system aapke program ko अदृश्य roop se interrupt karega taaki CPU network call ke poora hone ka intezaar karte hue doosre kaam kar sake.

Video export ek **CPU-bound** ya **compute-bound** operation ka ek udaharan hai. Yeh computer ki CPU ya GPU ke andar data processing ki potential speed se seemit hai. Video download ek **IO-bound** operation ka udaharan hai, kyunki yeh computer ke **input aur output** ki speed se seemit hai; yah utni hi tezi se chal sakta hai jitni tezi se data network par bheja ja sakta hai.

In dono udaharanon mein, operating system ke अदृश्य interrupts concurrency ka ek roop pradan karte hain. Lekin vah concurrency sirf poore program ke level par hoti hai. Kai maamlon mein, kyunki hum apne programs ko operating system se kahin zyada gehre level par samajhte hain, hum concurrency ke aise mauke dekh sakte hain jo operating system nahi dekh sakta.

Udaharan ke liye, agar hum file downloads manage karne ke liye ek tool bana rahe hain, to humein apne program ko is tarah likhna chahiye ki ek download shuru karne se UI lock na ho, aur users ek hi samay mein kai downloads shuru kar sakein. Lekin, network se interact karne ke liye kai operating system APIs **blocking** hote hain; yaani, ve program ki progress ko tab tak rok dete hain jab tak ki ve jo data process kar rahe hain, woh poori tarah se taiyaar na ho jaaye.

> **Note:** Agar aap iske baare mein sochen to zyada tar function calls aise hi kaam karte hain. Halaanki, *blocking* shabd aam taur par un function calls ke liye istemal hota hai jo files, network, ya computer par anya resources ke saath interact karte hain, kyunki ye aise maamle hain jahan ek individual program ko operation ke *non*-blocking hone se fayda hoga.

Hum har file ko download karne ke liye ek dedicated thread spawn karke apne main thread ko block hone se bacha sakte hain. Lekin, un threads ka overhead (atirikt bojh) aakhirkaar ek samasya ban jayega. Behtar hota agar call pehle se hi block na hota. Yah bhi behtar hota agar hum usi seedhe style mein likh paate jaisa hum blocking code mein likhte hain, kuch is tarah:

```rust,ignore,does_not_compile
let data = fetch_data_from(url).await;
println!("{data}");
```

Rust ka **async** (asynchronous ka short form) abstraction humein bilkul yahi deta hai. Is chapter mein, aap async ke baare mein sab kuch seekhenge jab hum nimnalikhit vishayon ko cover karenge:

  * Rust ke `async` aur `await` syntax ka upyog kaise karein
  * Async model ka upyog unhi kuch chunautiyon ko hal karne ke liye kaise karein jinhe humne Chapter 16 mein dekha tha
  * Multithreading aur async kaise ek doosre ke poorak samadhan pradan karte hain, jinhe aap kai maamlon mein jod sakte hain

Lekin isse pehle ki hum dekhen ki async vyavhaar mein kaise kaam karta hai, humein parallelism aur concurrency ke beech ke antron par charcha karne ke liye ek chhota sa detour lena hoga.

-----

### Parallelism aur Concurrency

Ab tak humne parallelism aur concurrency ko lagbhag ek jaisa maana hai. Ab humein unke beech adhik sateek roop se antar karne ki zaroorat hai.

Ek software project par ek team kaam ko kaise baant sakti hai, iske alag-alag tarikon par vichaar karein.

Jab ek vyakti unmein se kisi ke bhi poora hone se pehle kai alag-alag tasks par kaam karta hai, to yah **concurrency** hai. Maan lijiye aapke computer par do alag-alag projects hain, aur jab aap ek project se bor ho jaate hain ya atak jaate hain, to aap doosre par switch kar lete hain. Aap sirf ek vyakti hain, isliye aap dono tasks par theek ek hi samay mein progress nahi kar sakte, lekin aap multi-task kar sakte hain, unke beech switch karke ek samay mein ek par progress kar sakte hain (Figure 17-1 dekhen).

\<figure\>

<img src="../img/trpl17-01.svg" class="center" alt="A diagram with boxes labeled Task A and Task B, with diamonds in them representing subtasks. There are arrows pointing from A1 to B1, B1 to A2, A2 to B2, B2 to A3, A3 to A4, and A4 to B3. The arrows between the subtasks cross the boxes between Task A and Task B." />


\<figcaption\>Figure 17-1: Ek concurrent workflow, Task A aur Task B ke beech switch karte hue\</figcaption\>

\</figure\>

Jab team har sadasya ko ek task dekar aur us par akele kaam karvakar tasks ke ek samuh ko baantti hai, to yah **parallelism** hai. Team ka har vyakti theek ek hi samay mein progress kar sakta hai (Figure 17-2 dekhen).

\<figure\>

\<figcaption\>Figure 17-2: Ek parallel workflow, jahan Task A aur Task B par svatantra roop se kaam hota hai\</figcaption\>

\</figure\>

In dono workflows mein, aapko alag-alag tasks ke beech coordinate karna pad sakta hai. Shayad aapne socha tha ki ek vyakti ko saunpa gaya task baaki sabke kaam se bilkul svatantra tha, lekin vastav mein use pehle team ke kisi aur vyakti ko apna task poora karne ki zaroorat hai. Kuch kaam parallel mein kiya ja sakta tha, lekin kuch vastav mein **serial** tha: yah keval ek shrinkhala (series) mein ho sakta tha, ek ke baad ek task, jaisa ki Figure 17-3 mein hai.

\<figure\>

\<figcaption\>Figure 17-3: Ek anshik roop se parallel workflow, jahan Task A3, Task B3 ke parinaamon par block ho jaata hai.\</figcaption\>

\</figure\>

Isi tarah, aapko shayad ehsaas ho ki aapka apna ek task aapke hi doosre task par nirbhar hai. Ab aapka concurrent kaam bhi serial ho gaya hai.

Ek single CPU core wali machine par, CPU ek samay mein keval ek hi operation kar sakta hai, lekin vah abhi bhi concurrently kaam kar sakta hai. Computer ek gatividhi ko pause kar sakta hai aur doosri par switch kar sakta hai. Ek multiple CPU cores wali machine par, vah parallel mein bhi kaam kar sakta hai. Ek core ek task kar raha ho sakta hai jabki doosra core ek bilkul alag task kar raha ho, aur ye operations vastav mein ek hi samay par hote hain.

Jab Rust mein async ke saath kaam karte hain, to hum hamesha **concurrency** ke saath kaam kar rahe hote hain. Hardware, operating system, aur hum jo async runtime istemal kar rahe hain, uske aadhar par, vah concurrency parde ke peeche **parallelism** ka bhi upyog kar sakti hai.

Ab, aaiye gehraai se dekhen ki Rust mein async programming vastav mein kaise kaam karti hai.