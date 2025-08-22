# Using Modules to Reuse and Organize Code

Ek bar jab aap Rust mein programs likhna shuru kar dete hain, toh aapka code poori tarah se **main** function mein hi rehta hai. Jaise-jaise aapka code badhta hai, aap aakhir mein code ko reuse aur behtar organization ke liye dusre functions mein move kar dete hain. Apne code ko chhote-chhote parts mein divide karke, aap har part ko akele hi samajhne mein aasan banate hain. Lekin agar aapke paas bahut saare functions hon toh kya hoga? Rust mein ek **module system** hai jo code ko organized tarike se reuse karne mein madad karta hai.

---

### **Rust Modules**

Jaise aap code ki lines ko ek function mein extract karte hain, usi tarah aap functions (aur dusre code jaise **structs** aur **enums**) ko alag-alag modules mein extract kar sakte hain. Ek **module** ek **namespace** hai jismein functions ya types ki definitions hoti hain. Aap yeh choose kar sakte hain ki woh definitions module ke bahar dikhengi (public) ya nahi (private). Yahan ek overview hai ki modules kaise kaam karte hain:

* **`mod`** keyword ek naye module ko declare karta hai. Module ke andar ka code ya toh is declaration ke baad curly brackets mein aata hai ya kisi aur file mein.
* By default, functions, types, constants, aur modules **private** hote hain. **`pub`** keyword ek item ko public banata hai aur isliye uske namespace ke bahar visible karta hai.
* **`use`** keyword modules, ya modules ke andar ki definitions ko, scope mein lata hai taaki unhe refer karna aasan ho.

Hum inmein se har part ko dekh kar samjhenge ki woh poore system mein kaise fit hote hain.
