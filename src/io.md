# I/O

## การล็อก

มาโคร [`print!`] และ [`println!`] ของ Rust จะล็อก stdout ทุกครั้งที่เรียก หากคุณเรียกมาโครเหล่านี้ซ้ำๆ การล็อก stdout ด้วยมืออาจดีกว่า

[`print!`]: https://doc.rust-lang.org/std/macro.print.html
[`println!`]: https://doc.rust-lang.org/std/macro.println.html

ตัวอย่างเช่น เปลี่ยนโค้ดนี้:
```rust
# let lines = vec!["one", "two", "three"];
for line in lines {
    println!("{}", line);
}
```
เป็นแบบนี้:
```rust
# fn blah() -> Result<(), std::io::Error> {
# let lines = vec!["one", "two", "three"];
use std::io::Write;
let mut stdout = std::io::stdout();
let mut lock = stdout.lock();
for line in lines {
    writeln!(lock, "{}", line)?;
}
// stdout is unlocked when `lock` is dropped
# Ok(())
# }
```
stdin และ stderr ก็สามารถล็อกได้เช่นเดียวกันเมื่อต้องดำเนินการกับมันซ้ำๆ

## การบัฟเฟอร์

การทำ I/O กับไฟล์ของ Rust ไม่มีการบัฟเฟอร์โดยค่าเริ่มต้น หากคุณมีการเรียกอ่านหรือเขียนไฟล์หรือซ็อกเก็ตเครือข่ายขนาดเล็กซ้ำๆ หลายครั้ง ให้ใช้ [`BufReader`] หรือ [`BufWriter`] ซึ่งดูแลบัฟเฟอร์ในหน่วยความจำสำหรับอินพุตและเอาต์พุต ลดจำนวนการเรียกใช้ระบบ (system call) ที่ต้องใช้ให้น้อยที่สุด

[`BufReader`]: https://doc.rust-lang.org/std/io/struct.BufReader.html
[`BufWriter`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html

ตัวอย่างเช่น เปลี่ยนโค้ดตัวเขียนแบบไม่มีบัฟเฟอร์นี้:
```rust
# fn blah() -> Result<(), std::io::Error> {
# let lines = vec!["one", "two", "three"];
use std::io::Write;
let mut out = std::fs::File::create("test.txt")?;
for line in lines {
    writeln!(out, "{}", line)?;
}
# Ok(())
# }
```
เป็นแบบนี้:
```rust
# fn blah() -> Result<(), std::io::Error> {
# let lines = vec!["one", "two", "three"];
use std::io::{BufWriter, Write};
let mut out = BufWriter::new(std::fs::File::create("test.txt")?);
for line in lines {
    writeln!(out, "{}", line)?;
}
out.flush()?;
# Ok(())
# }
```
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/pull/93954),
[**ตัวอย่างที่ 2**](https://github.com/nnethercote/dhat-rs/pull/22/commits/8c3ae26f1219474ee55c30bc9981e6af2e869be2).

การเรียก [`flush`] อย่างชัดเจนไม่ใช่สิ่งที่จำเป็นเคร่งครัด เพราะการ flush จะเกิดขึ้นอัตโนมัติเมื่อ `out` ถูกดรอป อย่างไรก็ตาม ในกรณีนั้นข้อผิดพลาดใดๆ ที่เกิดขึ้นระหว่าง flush จะถูกละเลย ในขณะที่การ flush อย่างชัดเจนจะทำให้ข้อผิดพลาดนั้นปรากฏชัด

[`flush`]: https://doc.rust-lang.org/std/io/trait.Write.html#tymethod.flush

การลืมบัฟเฟอร์พบได้บ่อยกว่าเมื่อเขียน ทั้งตัวเขียนแบบไม่มีบัฟเฟอร์และแบบมีบัฟเฟอร์ต่างก็อิมพลีเมนต์เทรต [`Write`] ซึ่งหมายความว่าโค้ดสำหรับเขียนไปยังตัวเขียนแบบไม่มีบัฟเฟอร์และแบบมีบัฟเฟอร์นั้นแทบเหมือนกัน ในทางตรงกันข้าม ตัวอ่านแบบไม่มีบัฟเฟอร์อิมพลีเมนต์เทรต [`Read`] แต่ตัวอ่านแบบมีบัฟเฟอร์อิมพลีเมนต์เทรต [`BufRead`] ซึ่งหมายความว่าโค้ดสำหรับอ่านจากตัวอ่านแบบไม่มีบัฟเฟอร์และแบบมีบัฟเฟอร์นั้นแตกต่างกัน ตัวอย่างเช่น การอ่านไฟล์ทีละบรรทัดด้วยตัวอ่านแบบไม่มีบัฟเฟอร์เป็นเรื่องยาก แต่เป็นเรื่องง่ายมากกับตัวอ่านแบบมีบัฟเฟอร์โดยใช้ [`BufRead::read_line`] หรือ [`BufRead::lines`] ด้วยเหตุนี้ การเขียนตัวอย่างสำหรับตัวอ่านแบบเดียวกับตัวอย่างของตัวเขียนข้างต้น ซึ่งเวอร์ชันก่อนและหลังคล้ายกันมาก จึงเป็นเรื่องยาก

[`Write`]: https://doc.rust-lang.org/std/io/trait.Write.html
[`Read`]: https://doc.rust-lang.org/std/io/trait.Read.html
[`BufRead`]: https://doc.rust-lang.org/std/io/trait.BufRead.html
[`BufRead::read_line`]: https://doc.rust-lang.org/std/io/trait.BufRead.html#method.read_line
[`BufRead::lines`]: https://doc.rust-lang.org/std/io/trait.BufRead.html#method.lines

สุดท้าย โปรดทราบว่าการบัฟเฟอร์ใช้ได้กับ stdout ด้วย ดังนั้นคุณอาจต้องการผสมผสานการล็อกด้วยมือ *และ* การบัฟเฟอร์เมื่อเขียนไปยัง stdout หลายครั้ง

## การอ่านไฟล์ทีละบรรทัด

[ส่วนนี้]อธิบายวิธีหลีกเลี่ยงการจัดสรรหน่วยความจำมากเกินไปเมื่อใช้ [`BufRead`] อ่านไฟล์ทีละบรรทัด

[ส่วนนี้]: heap-allocations.md#การอานไฟลทละบรรทด
[`BufRead`]: https://doc.rust-lang.org/std/io/trait.BufRead.html

## การอ่านอินพุตเป็นไบต์ดิบ

ชนิดข้อมูล [String] ที่มีมาให้ในตัวใช้ UTF-8 ภายใน ซึ่งเพิ่มโอเวอร์เฮดเล็กน้อยแต่ไม่เป็นศูนย์จากการตรวจสอบความถูกต้องของ UTF-8 เมื่อคุณอ่านอินพุตเข้าไปในมัน หากคุณเพียงต้องการประมวลผลไบต์ของอินพุตโดยไม่ต้องกังวลเรื่อง UTF-8 (ตัวอย่างเช่น หากคุณจัดการกับข้อความ ASCII) คุณสามารถใช้ [`BufRead::read_until`] ได้

[String]: https://doc.rust-lang.org/std/string/struct.String.html
[`BufRead::read_until`]: https://doc.rust-lang.org/std/io/trait.BufRead.html#method.read_until

ยังมีเครตเฉพาะทางสำหรับอ่าน[ข้อมูลทีละบรรทัดแบบไบต์][byte-oriented lines of data]และทำงานกับ[ไบต์สตริง][byte strings]

[byte-oriented lines of data]: https://github.com/Freaky/rust-linereader
[byte strings]: https://github.com/BurntSushi/bstr
