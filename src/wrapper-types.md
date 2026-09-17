# ชนิดข้อมูลตัวหุ้ม

Rust มีชนิดข้อมูล "ตัวหุ้ม" (wrapper type) หลากหลายแบบ เช่น [`RefCell`] และ [`Mutex`] ซึ่งให้พฤติกรรมพิเศษกับค่า การเข้าถึงค่าเหล่านี้อาจใช้เวลาพอสมควร หากโดยปกติแล้วมีการเข้าถึงค่าลักษณะนี้หลายตัวพร้อมกัน การใส่ค่าเหล่านั้นไว้ในตัวหุ้มตัวเดียวอาจดีกว่า

[`RefCell`]: https://doc.rust-lang.org/std/cell/struct.RefCell.html
[`Mutex`]: https://doc.rust-lang.org/std/sync/struct.Mutex.html

ตัวอย่างเช่น struct แบบนี้:
```rust
# use std::sync::{Arc, Mutex};
struct S {
    x: Arc<Mutex<u32>>,
    y: Arc<Mutex<u32>>,
}
```
อาจแทนค่าได้ดีกว่าแบบนี้:
```rust
# use std::sync::{Arc, Mutex};
struct S {
    xy: Arc<Mutex<(u32, u32)>>,
}
```
การทำเช่นนี้จะช่วยเรื่องประสิทธิภาพหรือไม่ ขึ้นอยู่กับรูปแบบการเข้าถึงค่าที่แน่นอน
[**ตัวอย่าง**](https://github.com/rust-lang/rust/pull/68694/commits/7426853ba255940b880f2e7f8026d60b94b42404).
