# Writing Automated Tests

Edsger W. Dijkstra ne apne 1972 ke essay “The Humble Programmer” mein kaha tha, “Program testing bugs ki presence ko dikhane ka ek bahut effective tareeka ho sakta hai, lekin unki absence ko dikhane ke liye bilkul hi inadequate hai.” Iska ye matlab nahi ki humein jitna ho sake utna test karne ki koshish nahi karni chahiye!

Hamare programs mein **correctness** ka matlab ye hai ki hamara code wahi karta hai jo hum usse karwana chahte hain. Rust ko programs ki correctness ke baare mein bahut zyaada dhyaan rakhkar design kiya gaya hai, lekin correctness complex hai aur ise prove karna aasan nahi hai. Rust ka **type system** is bojh ka ek bahut bada hissa uthaata hai, lekin type system har tarah ki incorrectness ko nahi pakad sakta. Isliye, Rust mein language ke andar hi **automated software tests** likhne ke liye support shamil hai.

Ek example ke roop mein, maan lijiye hum ek `add_two` naam ka function likhte hain jo usko diye gaye number mein **2** jodta hai. Jab hum is function ko implement aur compile karte hain, to Rust ab tak aapne jo type checking aur borrow checking seekhi hai, wo sab karta hai taaki ye ensure ho sake ki, jaise, hum is function ko `String` value ya koi invalid reference nahi de rahe hain. Lekin Rust ye check **nahi** kar sakta ki ye function wahi karega jo hum isse karwana chahte hain, jo ki parameter mein 2 jodna hai, na ki parameter mein 10 jodna ya 50 ghataana! Yahin par **tests** kaam aate hain.

Hum tests likh sakte hain jo ye **assert** karein ki jab hum `add_two` function ko `3` dete hain, to return ki gayi value `5` hai. Jab bhi hum apne code mein changes karte hain, to hum in tests ko run kar sakte hain taaki ye ensure kar saken ki koi bhi existing correct behavior change nahi hua hai.

Testing ek complex skill hai: bhale hi hum ek chapter mein acche tests likhne ke baare mein har detail ko cover nahi kar sakte, hum Rust ki testing facilities ke **mechanics** discuss karenge. Hum aapke liye available **annotations aur macros**, tests ko run karne ke liye default behavior aur options, aur **unit tests aur integration tests** mein tests ko kaise organize kiya jaata hai, iske baare mein baat karenge.
