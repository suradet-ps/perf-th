# เวลาในการคอมไพล์

แม้ว่าจุดมุ่งหมายหลักของหนังสือเล่มนี้จะเน้นไปที่การปรับปรุงประสิทธิภาพการทำงานของโปรแกรม Rust แต่หัวข้อนี้จะกล่าวถึงการลดระยะเวลาในการคอมไพล์โปรแกรม Rust เนื่องจากเป็นหัวข้อเกี่ยวเนื่องที่หลายคนให้ความสนใจอย่างมาก

ในหัวข้อ [การลดเวลาในการคอมไพล์] เราได้อธิบายถึงวิธีลดเวลาคอมไพล์ผ่านตัวเลือกการตั้งค่าบิลด์ (build configuration) ไปแล้ว ส่วนเนื้อหาที่เหลือของบทนี้จะพูดถึงเทคนิคการลดเวลาคอมไพล์ที่จำเป็นต้องปรับแก้โค้ดของโปรแกรมโดยตรง

[การลดเวลาในการคอมไพล์]: build-configuration.md#การลดเวลาในการคอมไพล

สำหรับเทคนิคการลดเวลาคอมไพล์เพิ่มเติมอย่างครอบคลุม สามารถอ่านได้จาก [เคล็ดลับการลดเวลาในการคอมไพล์ Rust][Tips] โดย Corrode

[Tips]: https://corrode.dev/blog/tips-for-faster-rust-compile-times/

## การแสดงภาพการคอมไพล์

Cargo มีฟีเจอร์ที่ช่วยให้คุณเห็นภาพกระบวนการคอมไพล์โปรแกรมของคุณได้ เพียงบิลด์ด้วยคำสั่งนี้:
```text
cargo build --timings
```
เมื่อบิลด์เสร็จสิ้น Cargo จะพิมพ์พาธของไฟล์ HTML ออกมา เมื่อเปิดไฟล์ดังกล่าวในเว็บเบราว์เซอร์ คุณจะเห็น [Gantt chart][Gantt chart] ที่แสดงลำดับและ dependency ระหว่าง crate ต่างๆ ภายในโปรแกรม แผนภูมินี้จะช่วยให้เห็นว่า crate graph มีความสามารถในการคอมไพล์แบบขนาน (parallelism) ได้มากน้อยเพียงใด และช่วยระบุว่ามี crate ขนาดใหญ่ตัวใดที่กลายเป็นคอขวดทำให้การคอมไพล์ต้องรอเรียงตามลำดับ (serialize) จนควรแยกย่อยออกมาหรือไม่ ศึกษารายละเอียดเพิ่มเติมเกี่ยวกับการอ่านกราฟได้ที่ [เอกสารประกอบ][timings]

[Gantt chart]: https://en.wikipedia.org/wiki/Gantt_chart
[timings]: https://doc.rust-lang.org/nightly/cargo/reference/timings.html

## มาโคร

มาโครบางตัวสร้างโค้ดออกมาจำนวนมหาศาล ซึ่งโค้ดเหล่านั้นต้องใช้เวลาในการคอมไพล์ตามไปด้วย แฟล็ก `-Zmacro-stats` ของคอมไพเลอร์ Rust สามารถช่วยระบุกรณีเหล่านี้ได้

ตัวอย่างเช่น หากคุณต้องการวัดผลเฉพาะ crate ปลายน้ำ (leaf crate) ของโปรเจกต์:
```text
cargo +nightly rustc -- -Zmacro-stats
```
คอมไพเลอร์จะแสดงข้อมูลเกี่ยวกับปริมาณโค้ดที่ถูกสร้างขึ้นโดย procedural macro และ declarative macro ซึ่งโดยทั่วไปแล้วแบบแรกมักจะสร้างโค้ดออกมามากกว่าอย่างเห็นได้ชัด

หรือหากคุณต้องการวัดผลทุก crate ในโปรเจกต์:
```text
RUSTFLAGS="-Zmacro-stats" cargo +nightly build
```
หากต้องการดูโค้ดจริงที่มาโครขยาย (expand) ออกมา คุณสามารถใช้เครื่องมือ [cargo-expand]

[cargo-expand]: https://github.com/dtolnay/cargo-expand

เราไม่จำเป็นต้องกังวลกับมาโครที่สร้างโค้ดออกมาเพียงเล็กน้อย แต่ถ้ามาโครตัวใดตัวหนึ่งสร้างโค้ดออกมาในปริมาณที่พอๆ กับโค้ดที่เขียนด้วยมือทั้งหมด คุณอาจลองพิจารณาเลิกใช้มาโครตัวนั้น หรือเปลี่ยนไปใช้วิธีอื่นที่มี overhead ต่ำกว่าแทน
[**ตัวอย่าง**](https://nnethercote.github.io/2025/06/26/how-much-code-does-that-proc-macro-generate.html)

หรืออีกทางเลือกหนึ่งคือ ปรับแก้ตัวมาโครเองเพื่อให้สร้างโค้ดออกมาน้อยลง
[**ตัวอย่างที่ 1**](https://github.com/bevyengine/bevy/issues/19873),
[**ตัวอย่างที่ 2**](https://nnethercote.github.io/2025/08/16/speed-wins-when-fuzzing-rust-code-with-derive-arbitrary.html)

## LLVM IR

คอมไพเลอร์ Rust ใช้ [LLVM] เป็นแบ็กเอนด์ ซึ่งขั้นตอนการทำงานของ LLVM อาจกินเวลาส่วนใหญ่ของการคอมไพล์ โดยเฉพาะเมื่อฟรอนต์เอนด์ของคอมไพเลอร์ Rust ผลิต [IR] ออกมาเป็นจำนวนมาก จนทำให้ LLVM ต้องใช้เวลา optimize นานตามไปด้วย

[LLVM]: https://llvm.org/
[IR]: https://en.wikipedia.org/wiki/Intermediate_representation

เราสามารถตรวจวินิจฉัยปัญหานี้ได้ด้วยเครื่องมือ [`cargo llvm-lines`] ซึ่งจะแสดงให้เห็นว่าฟังก์ชัน Rust ใดที่ทำให้เกิด LLVM IR มากที่สุด โดยฟังก์ชันแบบ generic มักจะเป็นตัวการหลักเสมอ เนื่องจากมันสามารถถูก instantiate (สร้างโค้ดเฉพาะสำหรับแต่ละ type) ออกมาได้หลายสิบหรือแม้กระทั่งหลายร้อยครั้งในโปรแกรมขนาดใหญ่

[`cargo llvm-lines`]: https://github.com/dtolnay/cargo-llvm-lines/

หากฟังก์ชัน generic ก่อให้เกิด IR บวม (IR bloat) ก็มีวิธีแก้ไขอยู่หลายวิธี วิธีที่ตรงไปตรงมาที่สุดคือลดขนาดของตัวฟังก์ชันลง
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/pull/72166/commits/5a0ac0552e05c079f252482cfcdaab3c4b39d614),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/pull/91246/commits/f3bda74d363a060ade5e5caeb654ba59bfed51a4)

อีกวิธีหนึ่งคือแยกส่วนที่ไม่จำเป็นต้องเป็น generic ออกไปเป็นฟังก์ชันต่างหาก ซึ่งฟังก์ชันที่ไม่ใช่ generic นี้จะถูก instantiate เพียงครั้งเดียวเท่านั้น การจะใช้วิธีนี้ได้หรือไม่นั้นขึ้นอยู่กับรายละเอียดของฟังก์ชัน generic แต่ละตัว หากทำได้ เรามักจะเขียนฟังก์ชันที่ไม่ใช่ generic เป็น inner function ซ้อนไว้ข้างในฟังก์ชัน generic ได้อย่างเป็นระเบียบ ดังตัวอย่างโค้ดของ
[`std::fs::read`]:
```rust,ignore
pub fn read<P: AsRef<Path>>(path: P) -> io::Result<Vec<u8>> {
    fn inner(path: &Path) -> io::Result<Vec<u8>> {
        let mut file = File::open(path)?;
        let size = file.metadata().map(|m| m.len()).unwrap_or(0);
        let mut bytes = Vec::with_capacity(size as usize);
        io::default_read_to_end(&mut file, &mut bytes)?;
        Ok(bytes)
    }
    inner(path.as_ref())
}
```
[`std::fs::read`]: https://doc.rust-lang.org/std/fs/fn.read.html

[**ตัวอย่าง**](https://github.com/rust-lang/rust/pull/72013/commits/68b75033ad78d88872450a81745cacfc11e58178)

ในบางกรณี ฟังก์ชัน utility ทั่วไปอย่าง [`Option::map`] และ [`Result::map_err`] ก็ถูก instantiate บ่อยครั้ง การเปลี่ยนไปใช้นิพจน์ `match` ที่ให้ผลลัพธ์เทียบเท่ากันแทน อาจช่วยลดเวลาในการคอมไพล์ลงได้

[`Option::map`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map
[`Result::map_err`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map_err

ผลลัพธ์ของการปรับเปลี่ยนลักษณะนี้ต่อเวลาคอมไพล์โดยทั่วไปอาจมีไม่มากนัก แต่ในบางครั้งก็อาจส่งผลดีอย่างมีนัยสำคัญ
[**ตัวอย่าง**](https://github.com/servo/servo/issues/26585)

นอกจากนี้ การปรับเปลี่ยนดังกล่าวยังอาจช่วยลดขนาดของไฟล์ไบนารีลงได้อีกด้วย
