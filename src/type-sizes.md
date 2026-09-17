# ขนาดชนิดข้อมูล

การลดขนาดชนิดข้อมูลที่ถูกอินสแตนซ์บ่อยๆ ช่วยเพิ่มประสิทธิภาพได้

ตัวอย่างเช่น หากการใช้หน่วยความจำสูง โปรไฟเลอร์ฮีปอย่าง [DHAT] สามารถระบุจุดจัดสรรที่ฮอตและชนิดข้อมูลที่เกี่ยวข้องได้ การลดขนาดชนิดข้อมูลเหล่านี้ช่วยลดการใช้หน่วยความจำสูงสุด และอาจเพิ่มประสิทธิภาพโดยลดปริมาณการเข้าถึงหน่วยความจำและแรงกดดันต่อแคช

[DHAT]: https://www.valgrind.org/docs/manual/dh-manual.html

นอกจากนี้ ชนิดข้อมูล Rust ที่มีขนาดใหญ่กว่า 128 ไบต์จะถูกคัดลอกด้วย `memcpy` แทนที่จะเป็นโค้ดแบบอินไลน์ หาก `memcpy` ปรากฏในโปรไฟล์ในปริมาณที่ไม่น้อย โหมด "copy profiling" ของ DHAT จะบอกคุณได้อย่างแม่นยำว่าการเรียก `memcpy` ที่ฮอตอยู่ตรงไหนและชนิดข้อมูลใดเกี่ยวข้อง การลดขนาดชนิดข้อมูลเหล่านี้ให้เหลือ 128 ไบต์หรือน้อยกว่าสามารถทำให้โค้ดเร็วขึ้นได้ โดยหลีกเลี่ยงการเรียก `memcpy` และลดปริมาณการเข้าถึงหน่วยความจำ

## การวัดขนาดชนิดข้อมูล

[`std::mem::size_of`] ให้ขนาดของชนิดข้อมูลเป็นไบต์ แต่บ่อยครั้งคุณต้องการทราบเลย์เอาต์ที่แน่นอนด้วย ตัวอย่างเช่น enum อาจมีขนาดใหญ่จนน่าประหลาดใจเพราะวาเรียนต์เดียวที่มีขนาดใหญ่เกินไป

[`std::mem::size_of`]: https://doc.rust-lang.org/std/mem/fn.size_of.html

ตัวเลือก `-Zprint-type-sizes` ทำหน้าที่นี้พอดี มันไม่เปิดใช้งานบน rustc เวอร์ชัน release ดังนั้นคุณต้องใช้ rustc เวอร์ชัน nightly นี่คือตัวอย่างการเรียกผ่าน Cargo วิธีหนึ่ง:
```text
RUSTFLAGS=-Zprint-type-sizes cargo +nightly build --release
```
และนี่คือตัวอย่างการเรียก rustc:
```text
rustc +nightly -Zprint-type-sizes input.rs
```
มันจะพิมพ์รายละเอียดขนาด เลย์เอาต์ และการจัดแนวของทุกชนิดข้อมูลที่ใช้งาน ตัวอย่างเช่น สำหรับชนิดข้อมูลนี้:
```rust
enum E {
    A,
    B(i32),
    C(u64, u8, u64, u8),
    D(Vec<u32>),
}
```
มันจะพิมพ์ข้อมูลต่อไปนี้ พร้อมข้อมูลของชนิดข้อมูลในตัวบางตัว
```text
print-type-size type: `E`: 32 bytes, alignment: 8 bytes
print-type-size     discriminant: 1 bytes
print-type-size     variant `D`: 31 bytes
print-type-size         padding: 7 bytes
print-type-size         field `.0`: 24 bytes, alignment: 8 bytes
print-type-size     variant `C`: 23 bytes
print-type-size         field `.1`: 1 bytes
print-type-size         field `.3`: 1 bytes
print-type-size         padding: 5 bytes
print-type-size         field `.0`: 8 bytes, alignment: 8 bytes
print-type-size         field `.2`: 8 bytes
print-type-size     variant `B`: 7 bytes
print-type-size         padding: 3 bytes
print-type-size         field `.0`: 4 bytes, alignment: 4 bytes
print-type-size     variant `A`: 0 bytes
```
เอาต์พุตแสดงสิ่งต่อไปนี้
- ขนาดและการจัดแนวของชนิดข้อมูล
- สำหรับ enum ขนาดของดิสคริมิแนนต์
- สำหรับ enum ขนาดของแต่ละวาเรียนต์ (เรียงจากมากไปน้อย)
- ขนาด การจัดแนว และลำดับของทุกฟิลด์ (โปรดทราบว่าคอมไพเลอร์ได้จัดเรียงฟิลด์ของวาเรียนต์ `C` ใหม่เพื่อลดขนาดของ `E` ให้เหลือน้อยที่สุด)
- ขนาดและตำแหน่งของแพดดิงทั้งหมด

หรืออีกทางหนึ่ง เครต [top-type-sizes] สามารถใช้แสดงเอาต์พุตในรูปแบบที่กะทัดรัดขึ้นได้

[top-type-sizes]: https://crates.io/crates/top-type-sizes

เมื่อคุณทราบเลย์เอาต์ของชนิดข้อมูลฮอตแล้ว มีหลายวิธีในการลดขนาดมัน

## ลำดับฟิลด์

คอมไพเลอร์ Rust จัดเรียงฟิลด์ใน struct และ enum โดยอัตโนมัติเพื่อลดขนาดให้เหลือน้อยที่สุด (เว้นแต่จะระบุแอตทริบิวต์ `#[repr(C)]`) ดังนั้นคุณไม่ต้องกังวลเรื่องลำดับฟิลด์ แต่ยังมีวิธีอื่นในการลดขนาดชนิดข้อมูลฮอต

## enum ขนาดเล็กลง

หาก enum มีวาเรียนต์ที่ใหญ่เกินไป ลองพิจารณาใส่ `Box` ให้ฟิลด์หนึ่งหรือหลายฟิลด์ ตัวอย่างเช่น คุณอาจเปลี่ยนชนิดข้อมูลนี้:
```rust
type LargeType = [u8; 100];
enum A {
    X,
    Y(i32),
    Z(i32, LargeType),
}
```
เป็นแบบนี้:
```rust
# type LargeType = [u8; 100];
enum A {
    X,
    Y(i32),
    Z(Box<(i32, LargeType)>),
}
```
วิธีนี้ลดขนาดชนิดข้อมูลลง โดยแลกกับการต้องจัดสรรบนฮีปเพิ่มขึ้นหนึ่งครั้งสำหรับวาเรียนต์ `A::Z` ซึ่งมีแนวโน้มจะเป็นผลดีต่อประสิทธิภาพโดยรวมมากกว่าหากวาเรียนต์ `A::Z` เกิดขึ้นค่อนข้างน้อย `Box` ยังทำให้การใช้วาเรียนต์ `A::Z` สะดวกน้อยลงเล็กน้อย โดยเฉพาะในแพตเทิร์น `match`
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/pull/37445/commits/a920e355ea837a950b484b5791051337cd371f5d),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/pull/55346/commits/38d9277a77e982e49df07725b62b21c423b6428e),
[**ตัวอย่างที่ 3**](https://github.com/rust-lang/rust/pull/64302/commits/b972ac818c98373b6d045956b049dc34932c41be),
[**ตัวอย่างที่ 4**](https://github.com/rust-lang/rust/pull/64374/commits/2fcd870711ce267c79408ec631f7eba8e0afcdf6),
[**ตัวอย่างที่ 5**](https://github.com/rust-lang/rust/pull/64394/commits/7f0637da5144c7435e88ea3805021882f077d50c),
[**ตัวอย่างที่ 6**](https://github.com/rust-lang/rust/pull/71942/commits/27ae2f0d60d9201133e1f9ec7a04c05c8e55e665).

## จำนวนเต็มขนาดเล็กลง

บ่อยครั้งสามารถลดขนาดชนิดข้อมูลได้โดยใช้ชนิดจำนวนเต็มที่เล็กลง ตัวอย่างเช่น แม้ว่าการใช้ `usize` สำหรับดัชนีจะเป็นเรื่องธรรมชาติที่สุด แต่ก็มักสมเหตุสมผลที่จะเก็บดัชนีเป็น `u32`, `u16` หรือแม้แต่ `u8` แล้วแปลงเป็น `usize` ณ จุดใช้งาน
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/pull/49993/commits/4d34bfd00a57f8a8bdb60ec3f908c5d4256f8a9a),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/pull/50981/commits/8d0fad5d3832c6c1f14542ea0be038274e454524).

## บ็อกซ์สไลซ์

เวกเตอร์ของ Rust ประกอบด้วยสามคำ ได้แก่ ความยาว ความจุ และพอยเตอร์ หากคุณมีเวกเตอร์ที่ไม่น่าจะถูกแก้ไขในอนาคต คุณสามารถแปลงมันเป็น *บ็อกซ์สไลซ์ (boxed slice)* ด้วย [`Vec::into_boxed_slice`] บ็อกซ์สไลซ์มีเพียงสองคำ คือความยาวและพอยเตอร์ ความจุส่วนเกินขององค์ประกอบจะถูกทิ้ง ซึ่งอาจทำให้เกิดการจัดสรรใหม่
```rust
# use std::mem::{size_of, size_of_val};
let v: Vec<u32> = vec![1, 2, 3];
assert_eq!(size_of_val(&v), 3 * size_of::<usize>());

let bs: Box<[u32]> = v.into_boxed_slice();
assert_eq!(size_of_val(&bs), 2 * size_of::<usize>());
```
หรืออีกทางหนึ่ง บ็อกซ์สไลซ์สามารถสร้างโดยตรงจากอิเทอเรเตอร์ด้วย [`Iterator::collect`] หากทราบความยาวของอิเทอเรเตอร์ล่วงหน้า วิธีนี้จะหลีกเลี่ยงการจัดสรรใหม่ทั้งหมด
```rust
let bs: Box<[u32]> = (1..3).collect();
```
บ็อกซ์สไลซ์สามารถแปลงกลับเป็นเวกเตอร์ได้ด้วย [`slice::into_vec`] โดยไม่ต้องโคลนหรือจัดสรรใหม่

[`Vec::into_boxed_slice`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_boxed_slice
[`Iterator::collect`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect
[`slice::into_vec`]: https://doc.rust-lang.org/std/primitive.slice.html#method.into_vec

## `ThinVec`

ทางเลือกแทนบ็อกซ์สไลซ์คือ `ThinVec` จากเครต [`thin_vec`] ซึ่งทำงานเทียบเท่า `Vec` แต่เก็บความยาวและความจุไว้ในการจัดสรรเดียวกับองค์ประกอบ (หากมี) ซึ่งหมายความว่า `size_of::<ThinVec<T>>` มีเพียงหนึ่งคำ

`ThinVec` เป็นตัวเลือกที่ดีภายในชนิดข้อมูลที่ถูกอินสแตนซ์บ่อย สำหรับเวกเตอร์ที่มักว่างเปล่า นอกจากนี้ยังใช้ลดขนาดวาเรียนต์ที่ใหญ่ที่สุดของ enum ได้ หากวาเรียนต์นั้นมี `Vec` อยู่

[`thin_vec`]: https://crates.io/crates/thin-vec

## การป้องกันการถดถอยของประสิทธิภาพ

หากชนิดข้อมูลฮอตพอที่ขนาดของมันจะส่งผลต่อประสิทธิภาพ การใช้ static assertion เพื่อให้แน่ใจว่ามันจะไม่ถดถอยโดยไม่ตั้งใจนั้นเป็นความคิดที่ดี ตัวอย่างต่อไปนี้ใช้มาโครจากเครต [`static_assertions`]
```rust,ignore
  // This type is used a lot. Make sure it doesn't unintentionally get bigger.
  #[cfg(target_arch = "x86_64")]
  static_assertions::assert_eq_size!(HotType, [u8; 64]);
```
แอตทริบิวต์ `cfg` สำคัญมาก เพราะขนาดชนิดข้อมูลอาจแตกต่างกันบนแต่ละแพลตฟอร์ม การจำกัด assertion ให้เฉพาะ `x86_64` (ซึ่งโดยทั่วไปเป็นแพลตฟอร์มที่ใช้กันแพร่หลายที่สุด) น่าจะเพียงพอที่จะป้องกันการถดถอยในทางปฏิบัติได้

[`static_assertions`]: https://crates.io/crates/static_assertions
