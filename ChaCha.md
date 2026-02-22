chacha + MAC algorithm (**Poly1305**) > AES-GCM
7-round
chacha20 = 20 rounds ( 80 quarter rounds )
```
constant   constant  constant  constant
key        key       key       key
key        key       key       key
counter    nonce     nonce     nonce
```

```
QUARTERROUND(0,4,8,12)
QUARTERROUND(1,5,9,13)
QUARTERROUND(2,6,10,14)
QUARTERROUND(3,7,11,15)
```
feed forward เอาตารางที่ปั่นเสร็จ + initial state ช่องต่อช่อง
serialization เอาเลขในตารางมาวางเรียง จะได้ **keystream**

**keystream** xor Message =ciphertext

chacha20 input
- key (256-bit)
- nonce (96-bit)
- block count (32-bit)
output = 64 random-looking bytes

attack
- correlation attacks

## Test
- NIST Benchmark Tests

https://www.researchgate.net/journal/SN-Applied-Sciences-2523-3971/publication/349850199_An_improved_chacha_algorithm_for_securing_data_on_IoT_devices/links/6044662a92851c077f2126c6/An-improved-chacha-algorithm-for-securing-data-on-IoT-devices.pdf?_tp=eyJjb250ZXh0Ijp7ImZpcnN0UGFnZSI6Il9kaXJlY3QiLCJwYWdlIjoicHVibGljYXRpb25Eb3dubG9hZCIsInByZXZpb3VzUGFnZSI6InB1YmxpY2F0aW9uIn19
^
|
จาก paper นี้เค้าวัด Time consuming and complexity, Throughput, Power consumption จากอะไร

https://bench.cr.yp.to/
https://bench.cr.yp.to/results-stream/amd64-freshwrap,big.html

## Test chacha12
```
---Benchmarking ChaCha12---
Run 1: 0.049 seconds
Run 2: 0.050 seconds
Run 3: 0.050 seconds
Run 4: 0.050 seconds
Run 5: 0.050 seconds
Run 6: 0.049 seconds
Run 7: 0.050 seconds
Run 8: 0.049 seconds
Run 9: 0.049 seconds
Run 10: 0.050 seconds
16MB Average time: 0.050 seconds
----------------------------------
Run 1: 0.198 seconds
Run 2: 0.198 seconds
Run 3: 0.198 seconds
Run 4: 0.198 seconds
Run 5: 0.198 seconds
Run 6: 0.198 seconds
Run 7: 0.200 seconds
Run 8: 0.200 seconds
Run 9: 0.198 seconds
Run 10: 0.198 seconds
64MB Average time: 0.198 seconds
----------------------------------
Run 1: 1.792 seconds
Run 2: 1.796 seconds
Run 3: 1.789 seconds
Run 4: 1.789 seconds
Run 5: 1.830 seconds
Run 6: 1.789 seconds
Run 7: 1.789 seconds
Run 8: 1.828 seconds
Run 9: 1.789 seconds
Run 10: 1.790 seconds
576MB Average time: 1.798 seconds
----------------------------------
Run 1: 4.820 seconds
Run 2: 4.905 seconds
Run 3: 4.865 seconds
Run 4: 4.811 seconds
Run 5: 4.813 seconds
Run 6: 3.514 seconds
Run 7: 3.375 seconds
Run 8: 8.867 seconds
Run 9: 7.845 seconds
Run 10: 6.774 seconds
1536MB Average time: 5.459 seconds
----------------------------------
Run 1: 35.800 seconds
Run 2: 40.036 seconds
Run 3: 26.127 seconds
Run 4: 21.245 seconds
Run 5: 27.687 seconds
Run 6: 23.216 seconds
Run 7: 23.509 seconds
Run 8: 24.947 seconds
Run 9: 29.210 seconds
Run 10: 35.504 seconds
4096MB Average time: 28.728 seconds
----------------------------------
```

## resource
[The ChaCha family of stream ciphers](https://cr.yp.to/chacha.html)
https://cr.yp.to/chacha/chacha-20080128.pdf
https://phase.dev/blog/chacha-and-aes-simplicity-in-cryptography/


## chat

ไอเดียคุณ “เป็นไปได้เชิงวิจัย” แต่ต้องตั้งความคาดหวังให้ถูกก่อนนะ:

> ❗ ไม่มีของวิเศษที่แทน ARX แล้วได้ **ความปลอดภัยเท่า ChaCha20 + เร็วกว่า** แบบฟรี ๆ  
> ❗ ถ้ามีจริง มันคงถูกใช้แทนไปแล้วในมาตรฐาน

แต่…  
✅ มีช่องให้ทำ **hybrid design ที่น่าสนใจทางงานวิจัย**  
✅ โดยเฉพาะถ้าคุณโฟกัส IoT + fault robustness (ซึ่ง community ยังเปิด)

ผมจะชี้แนวที่ “มีลุ้น publish” ให้ตรง ๆ

---

# 🧠 ก่อนอื่น: ARX มันเก่งตรงไหน

ใน ChaCha20 (และ ChaCha12)

ARX =

- Add
    
- Rotate
    
- XOR
    

ข้อดีใหญ่:

- constant-time
    
- ไม่มี S-box → กัน cache attack ดี
    
- เร็วบน CPU ทั่วไป
    
- diffusion ดีมาก
    

👉 นี่คือเหตุผลที่มันยังครองสนาม

---

# 🎯 เป้าหมายคุณ (ตีความจากที่พูด)

คุณอยากได้:

- ความปลอดภัย ≈ ChaCha20
    
- ความเร็ว ≈ ChaCha12
    
- suitable for IoT
    
- (โบนัส) ทน fault ดีขึ้น
    

🔥 นี่คือ research problem ที่ legit มาก

---

# 🚀 แนวทางที่ “มีโอกาสเวิร์กจริง”

## 🥇 Option 1: Hybrid ARX + Lightweight Nonlinear Layer ⭐⭐⭐⭐⭐ (แนะนำสุด)

แนวคิด:

```
12 rounds = ARX (เหมือน ChaCha12)
+ 8 rounds = lightweight nonlinear strengthening
```

แทนที่จะใช้ ARX ล้วน

---

## 🔥 ตัวเลือก nonlinear ที่น่าสนใจ

### ✅ 1. Tiny S-box layer (lightweight)

แนว:

- ใช้ S-box เล็ก (4-bit หรือ 8-bit)
    
- แทรกบางรอบ
    
- ไม่ใช้ทุก operation
    

---

### 👍 ข้อดี

- เพิ่ม nonlinearity
    
- เพิ่ม resistance ต่อ differential
    
- เพิ่ม fault diffusion
    

---

### 👎 ความเสี่ยง

- timing leak ถ้าไม่ careful
    
- implementation complexity
    
- อาจช้าบนบาง MCU
    

---

📌 แต่สำหรับ research → น่าสนใจมาก

---

## 🥈 Option 2: ARX + Bit Permutation Layer ⭐⭐⭐⭐

อันนี้ underrated มาก

แนวคิด:

หลังบางรอบของ ChaCha12 ใส่:

```
bit shuffle / word permutation
```

เช่น:

- bit interleaving
    
- word shuffle
    
- rotation pattern ใหม่
    

---

### 👍 ข้อดี

- cheap มาก
    
- constant-time
    
- diffusion ดีขึ้น
    
- fault propagation เร็วขึ้น
    

🔥 เหมาะกับ IoT มาก

---

## 🥉 Option 3: ARX + Redundancy-aware design (สาย fault โดยตรง) ⭐⭐⭐⭐

อันนี้เข้าทางโปรเจคคุณสุด

แทนที่จะเพิ่มความแข็งทาง crypto อย่างเดียว

คุณออกแบบ:

> ChaCha12 + fault-detection-friendly structure

เช่น:

- invariant word
    
- checksum word
    
- dual computation
    

---

### 👍 จุดขาย

- เหมาะ IoT
    
- เข้าธีม fault injection
    
- reviewer ชอบ
    
- practical มาก
    

---

# ⚠️ แนวที่ผม “ไม่แนะนำ” สำหรับคุณตอนนี้

## ❌ แทน ARX ด้วย S-box หนัก ๆ

เช่น AES-style

ปัญหา:

- ช้าบน MCU
    
- side-channel risk
    
- complexity สูง
    
- หลุด theme ChaCha
    

---

## ❌ เอา ChaCha12 แล้ว “หวังว่าพอ”

ChaCha12:

- เร็วจริง
    
- แต่ margin ต่ำกว่า ChaCha20
    

👉 ต้องมี strengthening ไม่งั้น reviewer จะถามหนัก

---

# 🧪 Design ที่ผมคิดว่า “sweet spot” สำหรับคุณ

## 🔥 Candidate ที่น่าสนใจมาก

```
ChaCha12 (core speed)
+ Lightweight permutation layer every 2 rounds
+ Optional fault-detection checksum
```

คุณสามารถ claim:

- near ChaCha20 security (empirical)
    
- faster than ChaCha20
    
- better fault detectability
    

👉 publishable narrative

---

# 📊 วิธีพิสูจน์ (สำคัญมาก)

ถ้าคุณทำ hybrid ต้องวัด:

### Security

- differential bias
    
- avalanche
    
- NIST tests
    
- fault propagation
    

### Performance

- cycles
    
- energy
    
- code size
    

### Fault

- detection rate
    
- silent fault
    

🔥 ถ้าวัดครบ = งานแน่นมาก

---

# 🧠 คำแนะนำตรง ๆ สำหรับคุณ

จากที่ผมดู direction คุณ:

👉 อย่าพยายาม “แทน ARX ทั้งหมด”  
👉 ให้ทำ **ARX-based hybrid**

เพราะ:

- ปลอดภัยกว่า
    
- reviewer ยอมรับง่าย
    
- implement ง่าย
    
- ยังเร็ว
    

---

---

ถ้าคุณอยากก้าวต่อแบบจริงจัง ผมช่วยออกแบบได้:

- 🧩 hybrid ChaCha12+X structure
    
- 📐 round schedule ใหม่
    
- 🧪 experiment plan
    
- ⚡ estimation ว่าจะเร็วขึ้นกี่ %
    

พิมพ์มาได้เลยว่า:

> “ช่วย design ChaCha12 hybrid ให้หน่อย”

ผมจะออกแบบระดับ paper ให้คุณ 🚀