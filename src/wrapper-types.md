# ชนิดข้อมูลตัวหุ้ม

ภาษา Rust มีชนิดข้อมูล "ตัวห่อหุ้ม" (wrapper type) อยู่หลากหลายแบบ เช่น [`RefCell`] และ [`Mutex`] ซึ่งช่วยมอบพฤติกรรมพิเศษบางประการให้กับข้อมูลที่อยู่ภายใน ทว่าการเข้าถึงข้อมูลผ่าน wrapper type เหล่านี้มักมี overhead และกินเวลาไม่น้อย หากโปรแกรมของคุณมักจะเข้าถึงข้อมูลที่ถูกห่อหุ้มลักษณะนี้หลายๆ ค่าพร้อมกันอยู่เสมอ การนำข้อมูลเหล่านั้นมารวมไว้ภายใน wrapper ตัวเดียวกันเพียงตัวเดียวอาจเป็นทางเลือกที่ดีกว่า

[`RefCell`]: https://doc.rust-lang.org/std/cell/struct.RefCell.html
[`Mutex`]: https://doc.rust-lang.org/std/sync/struct.Mutex.html

ตัวอย่างเช่น struct ในลักษณะนี้:
```rust
# use std::sync::{Arc, Mutex};
struct S {
    x: Arc<Mutex<u32>>,
    y: Arc<Mutex<u32>>,
}
```
อาจปรับโครงสร้างให้ดีขึ้นได้เป็นแบบนี้:
```rust
# use std::sync::{Arc, Mutex};
struct S {
    xy: Arc<Mutex<(u32, u32)>>,
}
```
การปรับเปลี่ยนเช่นนี้จะช่วยเพิ่มประสิทธิภาพได้จริงหรือไม่นั้น ขึ้นอยู่กับรูปแบบการเข้าถึงข้อมูล (access patterns) ที่แท้จริงของโปรแกรม
[**ตัวอย่าง**](https://github.com/rust-lang/rust/pull/68694/commits/7426853ba255940b880f2e7f8026d60b94b42404)
