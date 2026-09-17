# ชนิดข้อมูลในไลบรารีมาตรฐาน

การอ่านเอกสารของชนิดข้อมูลทั่วไปในไลบรารีมาตรฐานนั้นคุ้มค่า—เช่น [`Vec`], [`Option`], [`Result`] และ [`Rc`]/[`Arc`]—เพื่อค้นหาฟังก์ชันที่น่าสนใจซึ่งบางครั้งใช้เพิ่มประสิทธิภาพได้

[`Vec`]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[`Option`]: https://doc.rust-lang.org/std/option/enum.Option.html
[`Result`]: https://doc.rust-lang.org/std/result/enum.Result.html
[`Rc`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

นอกจากนี้ การรู้จักทางเลือกที่มีประสิทธิภาพสูงกว่าชนิดข้อมูลในไลบรารีมาตรฐานก็คุ้มค่า เช่น [`Mutex`], [`RwLock`], [`Condvar`] และ [`Once`]

[`Mutex`]: https://doc.rust-lang.org/std/sync/struct.Mutex.html
[`RwLock`]: https://doc.rust-lang.org/std/sync/struct.RwLock.html
[`Condvar`]: https://doc.rust-lang.org/std/sync/struct.Condvar.html
[`Once`]: https://doc.rust-lang.org/std/sync/struct.Once.html

## `Vec`

วิธีที่ดีที่สุดในการสร้าง `Vec` ที่เต็มไปด้วยศูนย์ความยาว `n` คือ `vec![0; n]` วิธีนี้ง่ายและอาจ[เร็วพอๆ กันหรือเร็วกว่า]ทางเลือกอื่น เช่น การใช้ `resize`, `extend` หรืออะไรก็ตามที่เกี่ยวข้องกับ `unsafe` เพราะมันสามารถใช้ความช่วยเหลือจากระบบปฏิบัติการได้

[เร็วพอๆ กันหรือเร็วกว่า]: https://github.com/rust-lang/rust/issues/54628

[`Vec::remove`] ลบองค์ประกอบที่ดัชนีหนึ่งๆ และเลื่อนองค์ประกอบที่ตามมาทั้งหมดไปทางซ้ายหนึ่งตำแหน่ง ทำให้มีความซับซ้อน O(n) ส่วน [`Vec::swap_remove`] แทนที่องค์ประกอบที่ดัชนีหนึ่งๆ ด้วยองค์ประกอบสุดท้าย ซึ่งไม่รักษาลำดับ แต่มีความซับซ้อน O(1)

[`Vec::retain`] ลบหลายรายการออกจาก `Vec` ได้อย่างมีประสิทธิภาพ และมีเมธอดเทียบเท่าสำหรับชนิดคอลเลกชันอื่น เช่น `String`, `HashSet` และ `HashMap`

[`Vec::remove`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.remove
[`Vec::swap_remove`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.swap_remove
[`Vec::retain`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.retain

## `Option` และ `Result`

[`Option::ok_or`] แปลง `Option` เป็น `Result` และรับพารามิเตอร์ `err` ซึ่งจะถูกใช้หากค่า `Option` เป็น `None` โดย `err` จะถูกคำนวณแบบทันที (eagerly) หากการคำนวณนั้นมีต้นทุนสูง คุณควรใช้ [`Option::ok_or_else`] แทน ซึ่งคำนวณค่า error แบบขี้เกียจ (lazily) ผ่านโคลเชอร์ ตัวอย่างเช่น โค้ดนี้:
```rust
# fn expensive() {}
# let o: Option<u32> = None;
let r = o.ok_or(expensive()); // always evaluates `expensive()`
```
ควรเปลี่ยนเป็นแบบนี้:
```rust
# fn expensive() {}
# let o: Option<u32> = None;
let r = o.ok_or_else(|| expensive()); // evaluates `expensive()` only when needed
```
[**ตัวอย่าง**](https://github.com/rust-lang/rust/pull/50051/commits/5070dea2366104fb0b5c344ce7f2a5cf8af176b0).

[`Option::ok_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or
[`Option::ok_or_else`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or_else

มีทางเลือกที่คล้ายกันสำหรับ [`Option::map_or`], [`Option::unwrap_or`], [`Result::or`], [`Result::map_or`] และ [`Result::unwrap_or`]

[`Option::map_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map_or
[`Option::unwrap_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or
[`Result::or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.or
[`Result::map_or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map_or
[`Result::unwrap_or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.unwrap_or

## `Rc`/`Arc`

[`Rc::make_mut`]/[`Arc::make_mut`] ให้ความหมายแบบ clone-on-write โดยสร้างการอ้างอิงแบบเปลี่ยนได้ไปยัง `Rc`/`Arc` หากตัวนับการอ้างอิงมากกว่าหนึ่ง พวกมันจะ `clone` ค่าภายในเพื่อให้แน่ใจว่าเป็นเจ้าของเพียงรายเดียว มิฉะนั้นจะแก้ไขค่าเดิม พวกมันไม่ได้ถูกใช้บ่อยนัก แต่บางครั้งก็มีประโยชน์อย่างมาก
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/pull/65198/commits/3832a634d3aa6a7c60448906e6656a22f7e35628),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/pull/65198/commits/75e0078a1703448a19e25eac85daaa5a4e6e68ac).

[`Rc::make_mut`]: https://doc.rust-lang.org/std/rc/struct.Rc.html#method.make_mut
[`Arc::make_mut`]: https://doc.rust-lang.org/std/sync/struct.Arc.html#method.make_mut

## `Mutex`, `RwLock`, `Condvar` และ `Once`

เครต [`parking_lot`] มีอิมพลีเมนเทชันทางเลือกของชนิดข้อมูลสำหรับการซิงโครไนซ์เหล่านี้ API และความหมายของชนิดข้อมูลใน `parking_lot` นั้นคล้ายกันแต่ไม่เหมือนกันทุกประการกับชนิดข้อมูลเทียบเท่าในไลบรารีมาตรฐาน

เวอร์ชันของ `parking_lot` เคยเล็กกว่า เร็วกว่า และยืดหยุ่นกว่าตัวในไลบรารีมาตรฐานอย่างสม่ำเสมอ แต่เวอร์ชันในไลบรารีมาตรฐานได้ปรับปรุงไปมากแล้วบนบางแพลตฟอร์ม ดังนั้นคุณควรวัดผลก่อนเปลี่ยนไปใช้ `parking_lot`

[`parking_lot`]: https://crates.io/crates/parking_lot

หากคุณตัดสินใจใช้ชนิดข้อมูลของ `parking_lot` โดยทั่วทั้งโปรเจกต์ ก็เป็นเรื่องง่ายที่จะเผลอใช้ชนิดข้อมูลเทียบเท่าในไลบรารีมาตรฐานในบางจุด คุณสามารถ[ใช้ Clippy]เพื่อหลีกเลี่ยงปัญหานี้ได้

[ใช้ Clippy]: linting.md#การหามใชชนดขอมล
