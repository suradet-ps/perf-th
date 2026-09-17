# ชนิดข้อมูลในไลบรารีมาตรฐาน

การสละเวลาอ่านเอกสารของชนิดข้อมูลทั่วไปใน Standard Library อย่างละเอียด—เช่น [`Vec`], [`Option`], [`Result`] และ [`Rc`]/[`Arc`]—นับเป็นเรื่องที่คุ้มค่ามาก เพื่อมองหาฟังก์ชันหรือเมธอดที่มีประโยชน์ซึ่งในบางครั้งสามารถนำมาช่วยเพิ่มประสิทธิภาพได้อย่างคาดไม่ถึง

[`Vec`]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[`Option`]: https://doc.rust-lang.org/std/option/enum.Option.html
[`Result`]: https://doc.rust-lang.org/std/result/enum.Result.html
[`Rc`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

นอกจากนี้ การทำความรู้จักกับทางเลือกอื่นที่มีประสิทธิภาพสูงกว่าชนิดข้อมูลใน Standard Library สำหรับงานบางประเภทก็คุ้มค่าเช่นกัน เช่น ทางเลือกสำหรับ [`Mutex`], [`RwLock`], [`Condvar`] และ [`Once`]

[`Mutex`]: https://doc.rust-lang.org/std/sync/struct.Mutex.html
[`RwLock`]: https://doc.rust-lang.org/std/sync/struct.RwLock.html
[`Condvar`]: https://doc.rust-lang.org/std/sync/struct.Condvar.html
[`Once`]: https://doc.rust-lang.org/std/sync/struct.Once.html

## `Vec`

วิธีที่ดีที่สุดในการสร้าง `Vec` ที่เติมค่าศูนย์เต็มพื้นที่ความยาว `n` คือการใช้คำสั่ง `vec![0; n]` วิธีนี้ทั้งเรียบง่ายและน่าจะ [เร็วพอๆ กันหรือเร็วกว่า] ทางเลือกอื่นอย่างเช่น การใช้ `resize`, `extend` หรือเทคนิคใดๆ ก็ตามที่ใช้ `unsafe` เนื่องจากมันสามารถขอรับความช่วยเหลือจากระบบปฏิบัติการในการจัดสรรหน้าหน่วยความจำที่เต็มไปด้วยศูนย์ได้โดยตรง

[เร็วพอๆ กันหรือเร็วกว่า]: https://github.com/rust-lang/rust/issues/54628

เมธอด [`Vec::remove`] จะลบสมาชิก ณ index ที่ระบุออก แล้วเลื่อนสมาชิกที่อยู่ถัดไปทั้งหมดไปทางซ้ายหนึ่งตำแหน่ง ซึ่งทำให้มีความซับซ้อนเชิงเวลาเป็น O(n) ในขณะที่ [`Vec::swap_remove`] จะสลับเอาสมาชิกตัวสุดท้ายมาใส่แทนที่ index นั้น ซึ่งแม้จะไม่รักษาลำดับเดิมของข้อมูล แต่ก็มีความซับซ้อนเพียงแค่ O(1)

เมธอด [`Vec::retain`] ช่วยให้เราสามารถลบสมาชิกหลายตัวออกจาก `Vec` ได้อย่างมีประสิทธิภาพ และมีเมธอดเทียบเคียงในคอลเลกชันชนิดอื่นๆ ด้วย เช่น `String`, `HashSet` และ `HashMap`

[`Vec::remove`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.remove
[`Vec::swap_remove`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.swap_remove
[`Vec::retain`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.retain

## `Option` และ `Result`

เมธอด [`Option::ok_or`] ทำหน้าที่แปลง `Option` ให้เป็น `Result` โดยรับพารามิเตอร์ `err` เพื่อนำมาใช้เป็นค่าความผิดพลาดหาก `Option` มีค่าเป็น `None` ซึ่งค่า `err` นี้จะถูกประมวลผลทันที (evaluated eagerly) หากขั้นตอนการคำนวณค่า error ดังกล่าวมีค่าใช้จ่ายสูง คุณควรเปลี่ยนไปใช้ [`Option::ok_or_else`] แทน ซึ่งจะคำนวณค่า error แบบตามความจำเป็น (lazily) ผ่าน closure ตัวอย่างเช่น:
```rust
# fn expensive() {}
# let o: Option<u32> = None;
let r = o.ok_or(expensive()); // always evaluates `expensive()`
```
ควรเปลี่ยนมาเขียนเป็นแบบนี้:
```rust
# fn expensive() {}
# let o: Option<u32> = None;
let r = o.ok_or_else(|| expensive()); // evaluates `expensive()` only when needed
```
[**ตัวอย่าง**](https://github.com/rust-lang/rust/pull/50051/commits/5070dea2366104fb0b5c344ce7f2a5cf8af176b0)

[`Option::ok_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or
[`Option::ok_or_else`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or_else

นอกจากนี้ ยังมีเมธอดทางเลือกรูปแบบ lazy ในลักษณะเดียวกันสำหรับ [`Option::map_or`], [`Option::unwrap_or`], [`Result::or`], [`Result::map_or`] และ [`Result::unwrap_or`]

[`Option::map_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map_or
[`Option::unwrap_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or
[`Result::or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.or
[`Result::map_or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map_or
[`Result::unwrap_or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.unwrap_or

## `Rc`/`Arc`

เมธอด [`Rc::make_mut`]/[`Arc::make_mut`] ให้พฤติกรรมการทำงานแบบ clone-on-write โดยจะสร้าง mutable reference ไปยังข้อมูลภายใน `Rc`/`Arc` หากตัวนับการอ้างอิง (reference count) มีค่ามากกว่าหนึ่ง เมธอดจะทำการ `clone` ข้อมูลภายในออกมาเป็นสำเนาใหม่เพื่อให้แน่ใจว่ามีความเป็นเจ้าของแต่เพียงผู้เดียว (unique ownership) แต่หากไม่มีการแชร์กับใครอยู่แล้ว ก็จะเข้าไปแก้ไขค่าในข้อมูลเดิมได้ทันที แม้เราอาจไม่ได้เรียกใช้เมธอดเหล่านี้บ่อยนัก แต่ในบางสถานการณ์ก็ถือว่ามีประโยชน์อย่างยิ่งยวด
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/pull/65198/commits/3832a634d3aa6a7c60448906e6656a22f7e35628),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/pull/65198/commits/75e0078a1703448a19e25eac85daaa5a4e6e68ac)

[`Rc::make_mut`]: https://doc.rust-lang.org/std/rc/struct.Rc.html#method.make_mut
[`Arc::make_mut`]: https://doc.rust-lang.org/std/sync/struct.Arc.html#method.make_mut

## `Mutex`, `RwLock`, `Condvar` และ `Once`

crate [`parking_lot`] นำเสนอ implementation ทางเลือกสำหรับ synchronization primitives เหล่านี้ โดย API และพฤติกรรมของชนิดข้อมูลใน `parking_lot` จะมีความคล้ายคลึงกัน แต่ไม่ได้เหมือนกับชนิดข้อมูลใน Standard Library ทุกประการ

ในอดีต เวอร์ชันของ `parking_lot` เคยมีขนาดกะทัดรัดกว่า ทำงานเร็วกว่า และยืดหยุ่นกว่าตัวใน Standard Library อย่างเห็นได้ชัด แต่ในปัจจุบัน ชนิดข้อมูลใน Standard Library ได้รับการปรับปรุงประสิทธิภาพไปมากแล้วบนหลายๆ แพลตฟอร์ม ดังนั้น คุณจึงควรทำเบนช์มาร์กวัดผลก่อนตัดสินใจเปลี่ยนมาใช้ `parking_lot`

[`parking_lot`]: https://crates.io/crates/parking_lot

หากคุณตัดสินใจนำชนิดข้อมูลจาก `parking_lot` มาใช้ทั่วทั้งโปรเจกต์ ก็อาจเผลอเรียกใช้ตัวเทียบเท่าใน Standard Library ในบางจุดโดยไม่ได้ตั้งใจได้ง่าย ซึ่งคุณสามารถ [ใช้ Clippy] เพื่อช่วยป้องกันปัญหานี้ได้

[ใช้ Clippy]: linting.md#การหามใชชนิดขอมูล
