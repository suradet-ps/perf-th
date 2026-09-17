# I/O

## การล็อก

มาโคร [`print!`] และ [`println!`] ของภาษา Rust จะทำการล็อก stdout ทุกครั้งที่มีการเรียกใช้ หากโปรแกรมของคุณจำเป็นต้องเรียกมาโครเหล่านี้ซ้ำ ๆ หลายครั้ง การล็อก stdout ด้วยตนเอง (manual locking) อาจช่วยให้ทำงานได้เร็วกว่า

[`print!`]: https://doc.rust-lang.org/std/macro.print.html
[`println!`]: https://doc.rust-lang.org/std/macro.println.html

ตัวอย่างเช่น เปลี่ยนจากโค้ดต่อไปนี้:
```rust
# let lines = vec!["one", "two", "three"];
for line in lines {
    println!("{}", line);
}
```
เป็นโค้ดนี้:
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
ในทำนองเดียวกัน stdin และ stderr ก็สามารถล็อกด้วยวิธีนี้ได้เช่นกันเมื่อต้องทำงานกับสตรีมดังกล่าวซ้ำ ๆ ต่อเนื่อง

## การบัฟเฟอร์

การจัดการ File I/O ในภาษา Rust โดยค่าเริ่มต้นจะไม่มีการบัฟเฟอร์ (unbuffered) หากโปรแกรมของคุณมีการเรียกอ่านหรือเขียนไฟล์หรือเน็ตเวิร์กซ็อกเก็ต (network socket) ขนาดเล็กซ้ำ ๆ บ่อยครั้ง ควรเลือกใช้ [`BufReader`] หรือ [`BufWriter`] ซึ่งจะคอยบริหารจัดการบัฟเฟอร์ในหน่วยความจำสำหรับอินพุตและเอาต์พุต ช่วยลดจำนวนครั้งในการเรียกใช้งาน system call ให้เหลือน้อยที่สุด

[`BufReader`]: https://doc.rust-lang.org/std/io/struct.BufReader.html
[`BufWriter`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html

ตัวอย่างเช่น เปลี่ยนจากโค้ดตัวเขียนแบบไม่มีบัฟเฟอร์นี้:
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
มาเป็นแบบนี้:
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

การเรียกเมธอด [`flush`] อย่างชัดเจนไม่ได้เป็นข้อบังคับตายตัว เพราะการ flush จะเกิดขึ้นโดยอัตโนมัติเมื่อ `out` หลุดจากขอบเขตการทำงาน (drop) อย่างไรก็ตาม หากเกิดข้อผิดพลาดใด ๆ ขึ้นระหว่างการ flush อัตโนมัติ ข้อผิดพลาดนั้นจะถูกเพิกเฉยทิ้งไป ในขณะที่การเรียก flush แบบชัดเจนจะทำให้คุณสามารถจัดการกับข้อผิดพลาดดังกล่าวได้อย่างถูกต้อง

[`flush`]: https://doc.rust-lang.org/std/io/trait.Write.html#tymethod.flush

การลืมใส่บัฟเฟอร์มักเกิดขึ้นบ่อยกว่าเมื่อเป็นการเขียน เนื่องจากทั้ง writer แบบไม่มีบัฟเฟอร์และแบบมีบัฟเฟอร์ต่างก็อิมพลีเมนต์เทรต [`Write`] เหมือนกัน ทำให้โครงสร้างโค้ดในการเขียนข้อมูลของทั้งสองแบบแทบจะเหมือนกันทุกประการ ในทางตรงกันข้าม reader แบบไม่มีบัฟเฟอร์จะอิมพลีเมนต์เทรต [`Read`] แต่ reader แบบมีบัฟเฟอร์จะอิมพลีเมนต์เทรต [`BufRead`] ทำให้โค้ดสำหรับอ่านข้อมูลของสองแบบนี้แตกต่างกัน เช่น การอ่านไฟล์ทีละบรรทัดด้วย reader ที่ไม่มีบัฟเฟอร์ทำได้ยากมาก แต่กลับทำได้อย่างง่ายดายเมื่อใช้ reader แบบมีบัฟเฟอร์ผ่าน [`BufRead::read_line`] หรือ [`BufRead::lines`] ด้วยเหตุนี้ จึงเป็นการยากที่จะยกตัวอย่างเปรียบเทียบก่อนและหลังของ reader ให้เห็นชัดเหมือนฝั่ง writer ข้างต้น ซึ่งโค้ดแทบไม่ต่างกันเลย

[`Write`]: https://doc.rust-lang.org/std/io/trait.Write.html
[`Read`]: https://doc.rust-lang.org/std/io/trait.Read.html
[`BufRead`]: https://doc.rust-lang.org/std/io/trait.BufRead.html
[`BufRead::read_line`]: https://doc.rust-lang.org/std/io/trait.BufRead.html#method.read_line
[`BufRead::lines`]: https://doc.rust-lang.org/std/io/trait.BufRead.html#method.lines

สุดท้ายนี้ โปรดทราบว่าการบัฟเฟอร์สามารถใช้ร่วมกับ stdout ได้เช่นกัน ดังนั้นหากโปรแกรมของคุณต้องเขียนข้อความจำนวนมากลง stdout การรวมการล็อกด้วยตนเอง (manual locking) *และ* การบัฟเฟอร์ (buffering) เข้าด้วยกัน จะช่วยเพิ่มประสิทธิภาพได้อย่างมาก

## การอ่านไฟล์ทีละบรรทัด

[ส่วนนี้] อธิบายวิธีหลีกเลี่ยงการจัดสรรหน่วยความจำมากเกินความจำเป็น เมื่อใช้ [`BufRead`] สำหรับอ่านไฟล์ทีละบรรทัด

[ส่วนนี้]: heap-allocations.md#การอานไฟลทีละบรรทัด
[`BufRead`]: https://doc.rust-lang.org/std/io/trait.BufRead.html

## การอ่านอินพุตเป็นไบต์ดิบ

ชนิดข้อมูล [String] ที่มีมาให้ในตัวของ Rust จะจัดเก็บข้อมูลภายในเป็น UTF-8 เสมอ ซึ่งทำให้มีโอเวอร์เฮดเล็กน้อย (แต่ไม่เป็นศูนย์) จากการตรวจสอบความถูกต้องของรูปแบบ UTF-8 (UTF-8 validation) ทุกครั้งที่อ่านอินพุตเข้ามา หากคุณต้องการเพียงประมวลผลข้อมูลไบต์ดิบโดยไม่ต้องสนใจเรื่อง UTF-8 (เช่น จัดการกับข้อความ ASCII เท่านั้น) คุณสามารถเลือกใช้ [`BufRead::read_until`] แทนได้

[String]: https://doc.rust-lang.org/std/string/struct.String.html
[`BufRead::read_until`]: https://doc.rust-lang.org/std/io/trait.BufRead.html#method.read_until

นอกจากนี้ ยังมีเครตเฉพาะทางสำหรับอ่าน[ข้อมูลทีละบรรทัดในรูปแบบไบต์][byte-oriented lines of data] และสำหรับทำงานกับ[สตริงระดับไบต์ (byte strings)][byte strings]

[byte-oriented lines of data]: https://github.com/Freaky/rust-linereader
[byte strings]: https://github.com/BurntSushi/bstr
