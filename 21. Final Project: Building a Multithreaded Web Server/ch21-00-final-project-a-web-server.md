# Final Project: Multithreaded Web Server Banana  

Ye ek lambi journey rahi hai, lekin ab hum finally book ke end pe aa gaye hain.  
Is chapter mein, hum ek aur project saath mein banayenge jo last chapters mein  
seekhe hue concepts ko dikhayega, aur saath hi kuch pehle ke lessons ko bhi  
recap karega.  

Final project ke liye, hum ek **web server** banayenge jo “hello” bolega aur  
browser mein **Figure 21-1** ki tarah dikhega.  

### Web server banane ka plan:  

1. TCP aur HTTP ke baare mein thoda seekhna.  
2. Socket par TCP connections sunna (listen karna).  
3. Thoda HTTP requests parse karna.  
4. Ek proper HTTP response create karna.  
5. Thread pool ka use karke server ki throughput improve karna.  

![hello from rust](img/trpl21-01.png)  

<span class="caption">Figure 21-1: Hamara final shared project</span>  

### Shuru karne se pehle do baatein:  

1. Jo method hum yaha use karenge, vo **best way** nahi hai Rust mein web server  
   banane ka. Community members ne already production-ready crates publish kiye hain  
   [crates.io](https://crates.io/) par jo zyada complete web server aur thread pool  
   implementations dete hain. Lekin yaha hamara goal **seekhna** hai, easy route lena nahi.  
   Rust ek systems programming language hai, isliye hum khud decide kar sakte hain  
   ki kis abstraction level pe kaam karna hai, aur zarurat ho to neeche ke level  
   tak ja sakte hain jo dusre languages mein possible ya practical nahi hota.  

2. Hum yaha `async` aur `await` ka use nahi karenge. Thread pool banana khud mein  
   ek badi challenge hai, aur saath hi async runtime banana aur bhi mushkil ho jaata.  
   Lekin hum note karenge ki `async` aur `await` ka use kuch problems solve karne mein  
   helpful ho sakta hai jo hum is chapter mein dekhenge. Jaise ki Chapter 17 mein  
   bataya tha, bahut saare async runtimes apna kaam manage karne ke liye  
   thread pools ka hi use karte hain.  

### Conclusion  

Isliye, hum manually ek basic HTTP server aur thread pool likhenge taaki aap  
samajh sako ki future mein jo crates aap use karoge unke peeche ke general  
ideas aur techniques kya hote hain.  
