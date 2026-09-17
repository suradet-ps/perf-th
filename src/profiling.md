# การโปรไฟล์

เมื่อเพิ่มประสิทธิภาพโปรแกรม คุณยังต้องมีวิธีระบุว่าส่วนใดของโปรแกรมเป็น "ฮอต" (ถูกเรียกใช้บ่อยพอที่จะส่งผลต่อเวลาทำงาน) และคุ้มค่าที่จะแก้ไข ซึ่งทำได้ดีที่สุดด้วยการโปรไฟล์

## โปรไฟเลอร์

มีโปรไฟเลอร์ให้เลือกใช้หลายชนิด แต่ละชนิดมีจุดแข็งและจุดอ่อนของตัวเอง ต่อไปนี้เป็นรายการโปรไฟเลอร์ที่ไม่ครบถ้วนสมบูรณ์ ซึ่งเคยถูกนำไปใช้กับโปรแกรม Rust ได้สำเร็จ
- [perf] เป็นโปรไฟเลอร์เอนกประสงค์ที่ใช้ตัวนับประสิทธิภาพฮาร์ดแวร์ (hardware performance counter) โดย [Hotspot] และ [Firefox Profiler] เหมาะสำหรับดูข้อมูลที่บันทึกโดย perf ทำงานบน Linux
- [Instruments] เป็นโปรไฟเลอร์เอนกประสงค์ที่มาพร้อมกับ Xcode บน macOS
- [Intel VTune Profiler] เป็นโปรไฟเลอร์เอนกประสงค์ ทำงานบน Windows, Linux และ macOS
- [AMD μProf] เป็นโปรไฟเลอร์เอนกประสงค์ ทำงานบน Windows และ Linux
- [samply] เป็นโปรไฟเลอร์แบบสุ่มตัวอย่าง (sampling profiler) ที่สร้างโปรไฟล์ซึ่งดูได้ใน Firefox Profiler ทำงานบน Mac, Linux และ Windows
- [flamegraph] เป็นคำสั่ง Cargo ที่ใช้ perf/DTrace ในการโปรไฟล์โค้ดของคุณ แล้วแสดงผลลัพธ์เป็นเฟลมกราฟ (flame graph) ทำงานบน Linux และทุกแพลตฟอร์มที่รองรับ DTrace (macOS, FreeBSD, NetBSD และอาจรวมถึง Windows)
- [Cachegrind] และ [Callgrind] ให้จำนวนคำสั่งแบบภาพรวม รายฟังก์ชัน และรายบรรทัดซอร์สโค้ด พร้อมข้อมูลจำลองของแคชและการทำนายทิศทางสาขา ทำงานบน Linux และยูนิกซ์อื่นๆ บางระบบ
- [DHAT] เหมาะสำหรับการค้นหาว่าส่วนใดของโค้ดทำให้เกิดการจัดสรรหน่วยความจำจำนวนมาก และให้ข้อมูลเชิงลึกเกี่ยวกับการใช้หน่วยความจำสูงสุด นอกจากนี้ยังใช้ระบุการเรียก `memcpy` ที่ฮอตได้ ทำงานบน Linux และยูนิกซ์อื่นๆ บางระบบ [dhat-rs] เป็นทางเลือกเชิงทดลองที่ทรงพลังน้อยกว่าเล็กน้อยและต้องแก้ไขโปรแกรม Rust ของคุณเล็กน้อย แต่ทำงานได้บนทุกแพลตฟอร์ม
- [heaptrack] และ [bytehound] เป็นเครื่องมือโปรไฟล์ฮีป ทำงานบน Linux
- [`counts`] รองรับการโปรไฟล์เฉพาะกิจ (ad hoc profiling) ซึ่งผสมผสานการใช้คำสั่ง `eprintln!` เข้ากับการประมวลผลภายหลังแบบอิงความถี่ เหมาะสำหรับการได้ข้อมูลเชิงลึกเฉพาะโดเมนเกี่ยวกับบางส่วนของโค้ด ทำงานได้บนทุกแพลตฟอร์ม
- [Coz] ทำ *การโปรไฟล์เชิงสาเหตุ (causal profiling)* เพื่อวัดศักยภาพในการเพิ่มประสิทธิภาพ และรองรับ Rust ผ่าน [coz-rs] ทำงานบน Linux

[perf]: https://perf.wiki.kernel.org/index.php/Main_Page
[Hotspot]: https://github.com/KDAB/hotspot
[Firefox Profiler]: https://profiler.firefox.com/
[Instruments]: https://developer.apple.com/forums/tags/instruments
[Intel VTune Profiler]: https://www.intel.com/content/www/us/en/developer/tools/oneapi/vtune-profiler.html
[AMD μProf]: https://developer.amd.com/amd-uprof/
[samply]: https://github.com/mstange/samply/
[flamegraph]: https://github.com/flamegraph-rs/flamegraph
[Cachegrind]: https://www.valgrind.org/docs/manual/cg-manual.html
[Callgrind]: https://www.valgrind.org/docs/manual/cl-manual.html
[DHAT]: https://www.valgrind.org/docs/manual/dh-manual.html
[dhat-rs]: https://github.com/nnethercote/dhat-rs/
[Valgrind]: https://valgrind.org/
[heaptrack]: https://github.com/KDE/heaptrack
[bytehound]: https://github.com/koute/bytehound
[`counts`]: https://github.com/nnethercote/counts/
[Coz]: https://github.com/plasma-umass/coz
[coz-rs]: https://github.com/plasma-umass/coz/tree/master/rust

## ข้อมูลดีบัก

การโปรไฟล์บิลด์แบบรีลีสอย่างมีประสิทธิผล คุณอาจต้องเปิดข้อมูลดีบักระดับบรรทัดซอร์สโค้ด หากต้องการทำเช่นนี้ ให้เพิ่มบรรทัดต่อไปนี้ลงในไฟล์ `Cargo.toml` ของคุณ:
```toml
[profile.release]
debug = "line-tables-only"
```
ดู[เอกสาร Cargo] สำหรับรายละเอียดเพิ่มเติมเกี่ยวกับการตั้งค่า `debug`

[เอกสาร Cargo]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug

น่าเสียดายที่แม้ทำตามขั้นตอนข้างต้นแล้ว คุณก็ยังไม่ได้ข้อมูลการโปรไฟล์โดยละเอียดสำหรับโค้ดในไลบรารีมาตรฐาน เพราะเวอร์ชันที่เผยแพร่ของไลบรารีมาตรฐาน Rust ไม่ได้บิลด์มาพร้อมข้อมูลดีบัก

วิธีที่เชื่อถือได้มากที่สุดคือบิลด์คอมไพเลอร์และไลบรารีมาตรฐานเวอร์ชันของคุณเอง ตาม[คำแนะนำเหล่านี้] แล้วเพิ่มบรรทัดต่อไปนี้ลงในไฟล์ `bootstrap.toml` ที่รากของที่เก็บ:
```toml
[rust]
debuginfo-level = 1
```
เรื่องนี้น่าปวดหัว แต่ก็อาจคุ้มค่ากับความพยายามในบางกรณี

[คำแนะนำเหล่านี้]: https://github.com/rust-lang/rust

หรืออีกทางหนึ่ง ฟีเจอร์ [build-std] ที่ยังไม่เสถียรช่วยให้คุณคอมไพล์ไลบรารีมาตรฐานเป็นส่วนหนึ่งของการคอมไพล์โปรแกรมตามปกติ ด้วยการตั้งค่าการบิลด์เดียวกัน อย่างไรก็ตาม ชื่อไฟล์ที่ปรากฏในข้อมูลดีบักของไลบรารีมาตรฐานจะไม่ชี้ไปยังไฟล์ซอร์สโค้ด เพราะฟีเจอร์นี้ไม่ได้ดาวน์โหลดซอร์สโค้ดของไลบรารีมาตรฐานด้วย ดังนั้นวิธีนี้จะไม่ช่วยกับโปรไฟเลอร์อย่าง Cachegrind และ samply ที่ต้องใช้ซอร์สโค้ดจึงจะทำงานได้เต็มที่

[build-std]: https://doc.rust-lang.org/cargo/reference/unstable.html#build-std

## เฟรมพอยเตอร์

คอมไพเลอร์ Rust อาจตัดเฟรมพอยเตอร์ออกด้วยการเพิ่มประสิทธิภาพ ซึ่งอาจลดคุณภาพของข้อมูลการโปรไฟล์ เช่น แบ็กเทรซ หากต้องการบังคับให้คอมไพเลอร์ใช้เฟรมพอยเตอร์ ให้ใช้แฟล็ก `-C force-frame-pointers=yes` ตัวอย่างเช่น:
```bash
RUSTFLAGS="-C force-frame-pointers=yes" cargo build --release
```

หรืออีกทางหนึ่ง หากต้องการบังคับให้ใช้เฟรมพอยเตอร์จากไฟล์ [`config.toml`] (สำหรับหนึ่งโปรเจกต์หรือมากกว่า) ให้เพิ่มบรรทัดต่อไปนี้:
```toml
[build]
rustflags = ["-C", "force-frame-pointers=yes"]
```
[`config.toml`]: https://doc.rust-lang.org/cargo/reference/config.html

## การถอดรหัสชื่อสัญลักษณ์

Rust ใช้การเข้ารหัสชื่อ (name mangling) รูปแบบหนึ่งเพื่อเข้ารหัสชื่อฟังก์ชันในโค้ดที่คอมไพล์แล้ว หากโปรไฟเลอร์ไม่รู้จักเรื่องนี้ เอาต์พุตของมันอาจมีชื่อสัญลักษณ์ที่ขึ้นต้นด้วย `_ZN` หรือ `_R` เช่น `_ZN3foo3barE` หรือ
`_ZN28_$u7b$$u7b$closure$u7d$$u7d$E` หรือ
`_RMCsno73SFvQKx_1cINtB0_3StrKRe616263_E`

ชื่อลักษณะนี้สามารถถอดรหัสด้วยมือได้โดยใช้ [`rustfilt`]

[`rustfilt`]: https://crates.io/crates/rustfilt

หากคุณพบปัญหากับการถอดรหัสชื่อสัญลักษณ์ระหว่างการโปรไฟล์ การเปลี่ยน[รูปแบบการเข้ารหัสชื่อ][mangling format] จากรูปแบบ legacy เริ่มต้นเป็นรูปแบบ v0 ที่ใหม่กว่าอาจช่วยได้

[mangling format]: https://doc.rust-lang.org/rustc/codegen-options/index.html#symbol-mangling-version

หากต้องการใช้รูปแบบ v0 จากบรรทัดคำสั่ง ให้ใช้แฟล็ก `-C
symbol-mangling-version=v0` ตัวอย่างเช่น:
```bash
RUSTFLAGS="-C symbol-mangling-version=v0" cargo build --release
```

หรืออีกทางหนึ่ง หากต้องการเรียกใช้คำสั่งเหล่านี้จากไฟล์ [`config.toml`] (สำหรับหนึ่งโปรเจกต์หรือมากกว่า) ให้เพิ่มบรรทัดต่อไปนี้:
```toml
[build]
rustflags = ["-C", "symbol-mangling-version=v0"]
```
[`config.toml`]: https://doc.rust-lang.org/cargo/reference/config.html
