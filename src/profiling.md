# การโปรไฟล์

เมื่อต้องการเพิ่มประสิทธิภาพโปรแกรม สิ่งสำคัญที่คุณต้องรู้คือส่วนใดของโปรแกรมเป็นจุดที่ทำงานหนักเป็นพิเศษหรือ "hot" (ถูกเรียกใช้บ่อยจนส่งผลต่อเวลาทำงานโดยรวมอย่างมีนัยสำคัญ) และคุ้มค่าแก่การปรับแก้ ซึ่งวิธีที่ดีที่สุดในการค้นหาส่วนนี้คือการทำ profiling

## โปรไฟเลอร์

ปัจจุบันมีเครื่องมือโปรไฟเลอร์ให้เลือกใช้งานมากมาย โดยแต่ละตัวต่างมีจุดเด่นและข้อจำกัดเฉพาะตัว รายการต่อไปนี้เป็นเพียงส่วนหนึ่งของโปรไฟเลอร์ยอดนิยมที่นำมาใช้วิเคราะห์โปรแกรม Rust ได้อย่างมีประสิทธิภาพ
- [perf] เป็นโปรไฟเลอร์เอนกประสงค์ที่อาศัยตัวนับประสิทธิภาพฮาร์ดแวร์ (hardware performance counters) โดยสามารถใช้ [Hotspot] และ [Firefox Profiler] ร่วมด้วยเพื่อเปิดดูข้อมูลกราฟิกที่บันทึกไว้ได้อย่างสะดวก รองรับบน Linux
- [Instruments] เป็นโปรไฟเลอร์เอนกประสงค์ที่ติดตั้งมาพร้อมกับ Xcode บน macOS
- [Intel VTune Profiler] เป็นโปรไฟเลอร์เอนกประสงค์ประสิทธิภาพสูง ใช้งานได้ทั้งบน Windows, Linux และ macOS
- [AMD μProf] เป็นโปรไฟเลอร์เอนกประสงค์ ใช้งานได้บน Windows และ Linux
- [samply] เป็นโปรไฟเลอร์แบบสุ่มตรวจตัวอย่าง (sampling profiler) ที่สร้างไฟล์โปรไฟล์สำหรับเปิดดูใน Firefox Profiler ได้ รองรับทั้งบน macOS, Linux และ Windows
- [flamegraph] เป็นคำสั่งของ Cargo ที่เรียกใช้ perf หรือ DTrace เพื่อโปรไฟล์โค้ด แล้วนำผลลัพธ์มาวาดเป็นเฟลมกราฟ (flame graph) อย่างสวยงาม รองรับบน Linux และทุกแพลตฟอร์มที่รองรับ DTrace (macOS, FreeBSD, NetBSD และอาจรวมถึง Windows)
- [Cachegrind] และ [Callgrind] ช่วยนับจำนวนคำสั่งประมวลผลทั้งในระดับภาพรวม รายฟังก์ชัน และรายบรรทัดของซอร์สโค้ด พร้อมทั้งจำลองพฤติกรรมของแคชและการทำนายทิศทางสาขาคำสั่ง (branch prediction) รองรับการทำงานบน Linux และระบบปฏิบัติการตระกูล Unix อื่นๆ บางระบบ
- [DHAT] เหมาะอย่างยิ่งสำหรับการค้นหาว่าโค้ดส่วนใดสร้างภาระการจัดสรรหน่วยความจำ (allocations) สูง รวมถึงช่วยวิเคราะห์ปริมาณการใช้หน่วยความจำสูงสุด (peak memory) นอกจากนี้ยังช่วยระบุจุดที่มีการเรียกใช้ `memcpy` ถี่เกินไปได้ด้วย ทำงานบน Linux และ Unix อื่นๆ บางระบบ ส่วน [dhat-rs] เป็นอีกหนึ่งทางเลือกเชิงทดลองที่ความสามารถน้อยกว่าเล็กน้อย และต้องปรับโค้ดในโปรแกรม Rust เล็กน้อย แต่สามารถใช้งานได้บนทุกแพลตฟอร์ม
- [heaptrack] และ [bytehound] เป็นเครื่องมือสำหรับโปรไฟล์ฮีปโดยเฉพาะ ทำงานบน Linux
- [`counts`] รองรับการโปรไฟล์เฉพาะจุด (ad hoc profiling) ซึ่งทำงานโดยผสานการแทรกคำสั่ง `eprintln!` เข้ากับการวิเคราะห์ความถี่ของเหตุการณ์ที่เกิดขึ้น ช่วยให้ได้ข้อมูลเชิงลึกเฉพาะทางของโค้ดแต่ละส่วน ใช้งานได้บนทุกแพลตฟอร์ม
- [Coz] ใช้เทคนิค *การโปรไฟล์เชิงสาเหตุ (causal profiling)* เพื่อวัดศักยภาพที่แท้จริงว่าการเพิ่มประสิทธิภาพในแต่ละจุดจะช่วยให้โปรแกรมโดยรวมเร็วขึ้นได้แค่ไหน โดยรองรับ Rust ผ่านทาง [coz-rs] ทำงานบน Linux

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

เพื่อโปรไฟล์บิลด์แบบ release ได้อย่างมีประสิทธิภาพ คุณอาจต้องเปิดใช้งานข้อมูลดีบักในระดับบรรทัดของซอร์สโค้ด (source line debug info) โดยเพิ่มบรรทัดต่อไปนี้ลงในไฟล์ `Cargo.toml`:
```toml
[profile.release]
debug = "line-tables-only"
```
ดูรายละเอียดเพิ่มเติมเกี่ยวกับการตั้งค่า `debug` ได้ที่ [เอกสาร Cargo]

[เอกสาร Cargo]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug

อย่างไรก็ดี แม้จะเปิดคอนฟิกข้างต้นแล้ว คุณก็ยังไม่สามารถดูข้อมูลการโปรไฟล์โดยละเอียดของโค้ดใน Standard Library ได้อยู่ดี เนื่องจาก Rust เวอร์ชันเผยแพร่ทั่วไป (official toolchain) ไม่ได้บิลด์ไลบรารีมาตรฐานพร้อม debug info มาด้วยตั้งแต่แรก

วิธีแก้ที่แน่นอนที่สุดคือการบิลด์ตัวคอมไพเลอร์และไลบรารีมาตรฐานด้วยตัวคุณเอง โดยทำตาม [คำแนะนำเหล่านี้] แล้วเพิ่มบรรทัดต่อไปนี้ลงในไฟล์ `bootstrap.toml` ที่โฟลเดอร์รากของคลังโค้ด:
```toml
[rust]
debuginfo-level = 1
```
วิธีนี้ค่อนข้างยุ่งยากและใช้เวลา แต่ก็คุ้มค่าในบางกรณีที่คุณจำเป็นต้องวิเคราะห์โค้ดอย่างละเอียดจริงๆ

[คำแนะนำเหล่านี้]: https://github.com/rust-lang/rust

หรืออีกทางเลือกหนึ่งคือการใช้ฟีเจอร์ทดลอง [build-std] เพื่อสั่งคอมไพล์ไลบรารีมาตรฐานไปพร้อมกับการคอมไพล์โปรแกรมตามปกติภายใต้การตั้งค่าบิลด์เดียวกัน อย่างไรก็ตาม ชื่อไฟล์ที่ปรากฏในข้อมูลดีบักจะไม่ชี้ไปยังไฟล์ซอร์สโค้ดจริง เนื่องจากฟีเจอร์นี้ไม่ได้ดาวน์โหลดซอร์สโค้ดของไลบรารีมาตรฐานลงมาด้วย วิธีนี้จึงไม่ช่วยสำหรับโปรไฟเลอร์อย่าง Cachegrind และ samply ที่จำเป็นต้องอ่านไฟล์ซอร์สโค้ดเพื่อแสดงผลเต็มรูปแบบ

[build-std]: https://doc.rust-lang.org/cargo/reference/unstable.html#build-std

## เฟรมพอยเตอร์

คอมไพเลอร์ Rust อาจตัดเฟรมพอยเตอร์ (frame pointer) ทิ้งไปในระหว่างการ optimize ซึ่งอาจส่งผลเสียต่อความสมบูรณ์ของข้อมูลการโปรไฟล์ เช่น ลำดับสแตกของการเรียกใช้ฟังก์ชัน (stack traces / backtraces) หากต้องการบังคับให้คอมไพเลอร์คงเฟรมพอยเตอร์ไว้ ให้ระบุแฟล็ก `-C force-frame-pointers=yes` ตัวอย่างเช่น:
```bash
RUSTFLAGS="-C force-frame-pointers=yes" cargo build --release
```

หรือหากต้องการกำหนดให้ใช้เฟรมพอยเตอร์ถาวรผ่านไฟล์ [`config.toml`] (สำหรับโปรเจกต์เดียวหรือทั้งเวิร์กสเปซ) ให้เพิ่มบรรทัดต่อไปนี้:
```toml
[build]
rustflags = ["-C", "force-frame-pointers=yes"]
```
[`config.toml`]: https://doc.rust-lang.org/cargo/reference/config.html

## การถอดรหัสชื่อสัญลักษณ์

Rust มีกลไกแปลงชื่อฟังก์ชัน (name mangling) ให้อยู่ในรูปแบบเฉพาะเมื่อคอมไพล์เป็นโค้ดไบนารี หากโปรไฟเลอร์ไม่รู้จักกลไกนี้ ข้อมูลผลลัพธ์อาจแสดงชื่อฟังก์ชันเป็นสัญลักษณ์ที่อ่านยาก โดยขึ้นต้นด้วย `_ZN` หรือ `_R` เช่น `_ZN3foo3barE`, `_ZN28_$u7b$$u7b$closure$u7d$$u7d$E` หรือ `_RMCsno73SFvQKx_1cINtB0_3StrKRe616263_E`

ชื่อในลักษณะนี้สามารถนำมาถอดรหัสกลับ (demangle) ด้วยตนเองได้โดยใช้เครื่องมือ [`rustfilt`]

[`rustfilt`]: https://crates.io/crates/rustfilt

หากคุณพบปัญหากับการถอดรหัสชื่อสัญลักษณ์ระหว่างทำ profiling ลองเปลี่ยน [รูปแบบการเข้ารหัสชื่อ][mangling format] จากรูปแบบเดิม (legacy) ไปเป็นรูปแบบ v0 ที่ใหม่กว่า

[mangling format]: https://doc.rust-lang.org/rustc/codegen-options/index.html#symbol-mangling-version

หากต้องการใช้รูปแบบ v0 ผ่านบรรทัดคำสั่ง ให้ส่งแฟล็ก `-C symbol-mangling-version=v0` ตัวอย่างเช่น:
```bash
RUSTFLAGS="-C symbol-mangling-version=v0" cargo build --release
```

หรือหากต้องการกำหนดไว้ในไฟล์ [`config.toml`] ให้เพิ่มบรรทัดต่อไปนี้:
```toml
[build]
rustflags = ["-C", "symbol-mangling-version=v0"]
```
[`config.toml`]: https://doc.rust-lang.org/cargo/reference/config.html

