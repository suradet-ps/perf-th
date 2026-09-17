# พจนานุกรมศัพท์ (Glossary) — The Rust Performance Book ฉบับภาษาไทย

ตารางนี้รวบรวมคำศัพท์เชิงเทคนิคและแนวทางการแปลที่ใช้อย่างสม่ำเสมอตลอดทั้งเล่ม เพื่อให้การแปลทุกบทมีความถูกต้อง ลื่นไหล และเป็นธรรมชาติสำหรับนักพัฒนาซอฟต์แวร์ชาวไทย

| ศัพท์ต้นฉบับ | คำแปลไทย | หมายเหตุ |
|---|---|---|
| performance | ประสิทธิภาพ (performance) | กล่าวถึงครั้งแรกด้วย "ประสิทธิภาพ (performance)" |
| optimization | การเพิ่มประสิทธิภาพ / การปรับให้เหมาะสมที่สุด (optimization) | "optimize" ใช้ "เพิ่มประสิทธิภาพ" |
| hot code / hot path | โค้ดฮอต / พาธฮอต (hot code / hot path) | ส่วนของโค้ดที่ถูกเรียกใช้บ่อยจนส่งผลต่อเวลาทำงาน |
| cold call site | จุดเรียกใช้แบบเย็น (cold call site) | จุดเรียกใช้ที่แทบไม่ถูกเรียก |
| benchmarking | การทำเบนช์มาร์ก (benchmarking) | การเปรียบเทียบและวัดประสิทธิภาพของโปรแกรม |
| benchmark | เบนช์มาร์ก (benchmark) | คง `cargo bench` ในโค้ด |
| workload | เวิร์กโหลด (workload) | ภาระงานที่ใช้วัดประสิทธิภาพ |
| metric | เมตริก (metric) | ตัวชี้วัด เช่น เวลา จำนวนคำสั่ง จำนวนไซเคิล |
| wall-time | เวลาตามนาฬิกาจริง (wall-time) | เวลาที่ผ่านไปจริง ซึ่งผู้ใช้รับรู้ได้ |
| profiling | การโปรไฟล์ (profiling) | การเก็บข้อมูลพฤติกรรมการทำงานของโปรแกรม |
| profiler | โปรไฟเลอร์ (profiler) | เครื่องมือโปรไฟล์ |
| sampling profiler | โปรไฟเลอร์แบบสุ่มตัวอย่าง (sampling profiler) | |
| flame graph | เฟลมกราฟ (flame graph) | กราฟแสดงลำดับการเรียกใช้ฟังก์ชัน |
| ad hoc profiling | การโปรไฟล์เฉพาะกิจ (ad hoc profiling) | |
| build configuration | การตั้งค่าการบิลด์ (build configuration) | |
| dev build | บิลด์แบบเดฟ (dev build) | บิลด์เริ่มต้นที่ยังไม่ผ่านการเพิ่มประสิทธิภาพ |
| release build | บิลด์แบบรีลีส (release build) | บิลด์ที่ผ่านการเพิ่มประสิทธิภาพแล้ว |
| codegen unit | หน่วยโค้ดเจน (codegen unit) | |
| link-time optimization (LTO) | การเพิ่มประสิทธิภาพตอนลิงก์ (link-time optimization; LTO) | |
| profile-guided optimization (PGO) | การเพิ่มประสิทธิภาพโดยใช้ข้อมูลโปรไฟล์ (profile-guided optimization; PGO) | |
| linker | ตัวลิงก์ (linker) | |
| allocator | อัลโลเคเตอร์ (allocator) | ตัวจัดสรรหน่วยความจำ |
| heap allocation | การจัดสรรหน่วยความจำบนฮีป (heap allocation) | |
| stack / heap | สแตก / ฮีป (stack / heap) | |
| bounds check | การตรวจสอบขอบเขต (bounds check) | การตรวจสอบดัชนีให้อยู่ในช่วงของคอนเทนเนอร์ |
| slice | สไลซ์ (slice) | |
| vector | เวกเตอร์ / `Vec` (vector) | คง `Vec` เมื่อหมายถึงชนิดข้อมูลในโค้ด |
| hashing / hash | การแฮช / แฮช (hashing / hash) | |
| hasher | ตัวแฮช (hasher) | |
| hash table | ตารางแฮช (hash table) | |
| collision | การชนกัน (collision) | คีย์ต่างกันแต่ได้ค่าแฮชซ้ำกัน |
| newtype | นิวไทป์ (newtype) | struct ที่ห่อค่าหนึ่งค่าไว้ |
| iterator | อิเทอเรเตอร์ (iterator) | |
| closure | โคลเชอร์ (closure) | |
| trait | เทรต (trait) | |
| crate | เครต (crate) | |
| macro | มาโคร (macro) | |
| generic | เจเนอริก (generic) | |
| enum / struct | enum / struct (คงชื่อเดิม) | คำสงวนของภาษา Rust |
| variant | วาเรียนต์ (variant) | |
| field | ฟิลด์ (field) | |
| inlining | การอินไลน์ (inlining) | การฝังเนื้อหาฟังก์ชันเข้าไปในจุดเรียกใช้ |
| outlining | การเอาต์ไลน์ (outlining) | การย้ายโค้ดที่แทบไม่ถูกเรียกออกไปเป็นฟังก์ชันแยก |
| call site | จุดเรียกใช้ (call site) | |
| lint | ลินต์ (lint) | คำเตือนจากเครื่องมือวิเคราะห์โค้ด |
| linting | การใช้ลินต์ (linting) | |
| assertion | แอสเซอร์ชัน (assertion) | |
| panic | การแพนิก (panic) | |
| abort | การยกเลิกทันที (abort) | |
| unwinding | การคลายสแตก (unwinding) | |
| backtrace | แบ็กเทรซ (backtrace) | |
| debug info | ข้อมูลดีบัก (debug info) | |
| frame pointer | เฟรมพอยเตอร์ (frame pointer) | |
| symbol mangling / demangling | การเข้ารหัสชื่อสัญลักษณ์ / การถอดรหัสชื่อสัญลักษณ์ (symbol mangling / demangling) | |
| wrapper type | ชนิดข้อมูลตัวหุ้ม (wrapper type) | |
| reference count | ตัวนับการอ้างอิง (reference count) | |
| clone | โคลน (clone) | |
| lock / locking | ล็อก / การล็อก (lock / locking) | |
| buffering | การบัฟเฟอร์ (buffering) | |
| thread | เธรด (thread) | |
| parallelism | การประมวลผลแบบขนาน (parallelism) | |
| SIMD | SIMD (คงไว้) | Single Instruction Multiple Data |
| vectorization | การทำเวกเตอร์ไรเซชัน (vectorization) | |
| cache miss | แคชพลาด (cache miss) | |
| branch misprediction | การทำนายทิศทางผิดพลาด (branch misprediction) | |
| memory traffic | ปริมาณการเข้าถึงหน่วยความจำ (memory traffic) | |
| padding | แพดดิง (padding) | ไบต์ว่างที่เติมเพื่อจัดแนว |
| alignment | การจัดแนว (alignment) | |
| discriminant | ดิสคริมิแนนต์ (discriminant) | ค่าที่ระบุวาเรียนต์ของ enum |
| layout | เลย์เอาต์ (layout) | การจัดวางข้อมูลในหน่วยความจำ |
| boxed slice | บ็อกซ์สไลซ์ (boxed slice) | `Box<[T]>` |
| overhead | โอเวอร์เฮด (overhead) | ต้นทุนที่แฝงมากับการทำงาน |
| trade-off | ข้อแลกเปลี่ยน (trade-off) | |
| regression | การถดถอยของประสิทธิภาพ (regression) | ค่าที่เคยดีกลับแย่ลงโดยไม่ตั้งใจ |
| binary size | ขนาดไบนารี (binary size) | |
| cross-crate | ข้ามเครต (cross-crate) | |
| output file | ไฟล์ผลลัพธ์ | |

## หลักการทั่วไป

- **ชื่อทางเทคนิค**: ชื่อเครื่องมือ คำสั่ง CLI ตัวเลือก (flag) ชื่อเครต/เทรต/ฟังก์ชัน/ชนิดข้อมูล ชื่อไฟล์ และ URL **ไม่แปล** เช่น `Cargo.toml`, `Vec`, `#[inline(always)]`, `-C target-cpu=native`, `cargo build --release`
- **โค้ดทุกบล็อก (` ```...``` `)**: ต้องคงไว้ตามต้นฉบับภาษาอังกฤษทุกตัวอักษร (byte-identical) รวมถึงคอมเมนต์ การเว้นวรรค และแท็กภาษา (เช่น `rust`, `toml`, `text`, `bash`) ภายในบล็อก
- **ลิงก์ (Links)**: ทั้ง inline links และ reference links ต้องชี้ไปยัง target เดิมเสมอ เพื่อให้ mdbook build ผ่านและไม่เกิด broken links
- **หัวข้อ (Headings)**: แปลเป็นไทยอย่างเป็นธรรมชาติและกระชับ โดยคงจำนวน ระดับ และลำดับของหัวข้อให้ตรงกับต้นฉบับทุกประการ
- **Anchor ของลิงก์ภายในเล่ม**: ตรวจสอบกับ HTML ที่ mdbook ทำการ build แล้วเสมอ โดย mdbook จะตัดสระ/วรรณยุกต์ไทยออกจาก slug อัตโนมัติ จึงต้องตรวจสอบด้วย `scripts/check-links.ps1` ทุกครั้ง
- **รูปประโยค**: ใช้คำว่า "คุณ" แทนผู้อ่าน หลีกเลี่ยงสรรพนามซ้ำซ้อน และคงโทนกึ่งทางการแบบเอกสารเทคนิค
