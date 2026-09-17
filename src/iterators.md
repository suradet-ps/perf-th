# อิเทอเรเตอร์

## `collect` และ `extend`

[`Iterator::collect`] ทำหน้าที่แปลงอิเทอเรเตอร์ให้กลายเป็นคอลเลกชัน เช่น `Vec` ซึ่งโดยทั่วไปจะต้องมีการจัดสรรหน่วยความจำ (allocation) คุณควรหลีกเลี่ยงการเรียก `collect` หากคอลเลกชันนั้นถูกนำไปแค่วนลูปซ้ำต่ออีกเพียงรอบเดียวเท่านั้น

[`Iterator::collect`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect

ด้วยเหตุนี้ การคืนค่าเป็นชนิดข้อมูลอิเทอเรเตอร์ เช่น `impl Iterator<Item=T>` ออกจากฟังก์ชันจึงมักจะดีกว่าการคืนค่าเป็น `Vec<T>` ทั้งนี้โปรดทราบว่าในบางกรณีอาจจำเป็นต้องระบุ lifetime เพิ่มเติมให้กับชนิดข้อมูลที่คืนค่ากลับมา ดังที่[บทความบล็อกนี้]ได้อธิบายไว้
[**ตัวอย่าง**](https://github.com/rust-lang/rust/pull/77990/commits/660d8a6550a126797aa66a417137e39a5639451b).

[บทความบล็อกนี้]: https://blog.katona.me/2019/12/29/Rust-Lifetimes-and-Iterators/

ในทำนองเดียวกัน คุณสามารถใช้ [`extend`] เพื่อขยายคอลเลกชันเดิมที่มีอยู่แล้ว (เช่น `Vec`) ด้วยอิเทอเรเตอร์ แทนที่จะนำอิเทอเรเตอร์ไป collect ใส่ `Vec` ใหม่แล้วค่อยนำมาต่อด้วย [`append`]

[`extend`]: https://doc.rust-lang.org/std/iter/trait.Extend.html#tymethod.extend
[`append`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.append

สุดท้าย เมื่อคุณสร้างอิเทอเรเตอร์ขึ้นมาเอง หากเป็นไปได้ก็มักจะคุ้มค่าที่จะอิมพลีเมนต์เมธอด [`Iterator::size_hint`] หรือ [`ExactSizeIterator::len`] ด้วย เพราะการเรียกใช้ `collect` และ `extend` กับอิเทอเรเตอร์ดังกล่าวจะสามารถลดจำนวนครั้งในการจัดสรรหน่วยความจำลงได้ เนื่องจากทราบข้อมูลจำนวนสมาชิกที่อิเทอเรเตอร์จะสร้างออกมาล่วงหน้า

[`Iterator::size_hint`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.size_hint
[`ExactSizeIterator::len`]: https://doc.rust-lang.org/std/iter/trait.ExactSizeIterator.html#method.len

## การเชื่อมต่อ

[`chain`] สะดวกมาก แต่ก็อาจช้ากว่าการใช้อิเทอเรเตอร์ตัวเดียว หากเป็นไปได้จึงอาจคุ้มค่าที่จะหลีกเลี่ยงการใช้ `chain` กับอิเทอเรเตอร์ที่ทำงานบ่อย (hot iterator)
[**ตัวอย่าง**](https://github.com/rust-lang/rust/pull/64801/commits/5ca99b750e455e9b5e13e83d0d7886486231e48a).

ในทำนองเดียวกัน [`filter_map`] ก็อาจเร็วกว่าการใช้ [`filter`] แล้วตามด้วย [`map`]

[`chain`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.chain
[`filter_map`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter_map
[`filter`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter
[`map`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.map

## ชังก์

เมื่อจำเป็นต้องใช้อิเทอเรเตอร์เพื่อแบ่งข้อมูลเป็นกลุ่ม (chunks) และทราบแน่ชัดว่าขนาดของชังก์หารความยาวของสไลซ์ (slice) ลงตัวพอดี ให้เลือกใช้ [`slice::chunks_exact`] ซึ่งทำงานได้เร็วกว่าแทนที่จะใช้ [`slice::chunks`]

แม้จะไม่ทราบแน่ชัดว่าขนาดของชังก์หารความยาวของสไลซ์ลงตัวพอดีหรือไม่ การใช้ `slice::chunks_exact` ร่วมกับ [`ChunksExact::remainder`] หรือจัดการสมาชิกส่วนที่เหลือด้วยตนเอง ก็ยังอาจทำงานได้เร็วกว่าอยู่ดี
[**ตัวอย่างที่ 1**](https://github.com/johannesvollmer/exrs/pull/173/files),
[**ตัวอย่างที่ 2**](https://github.com/johannesvollmer/exrs/pull/175/files).

หลักการเดียวกันนี้ยังใช้ได้กับอิเทอเรเตอร์ที่เกี่ยวข้องกลุ่มอื่น ๆ ด้วย:
- [`slice::rchunks`], [`slice::rchunks_exact`] และ [`RChunksExact::remainder`]
- [`slice::chunks_mut`], [`slice::chunks_exact_mut`] และ [`ChunksExactMut::into_remainder`]
- [`slice::rchunks_mut`], [`slice::rchunks_exact_mut`] และ [`RChunksExactMut::into_remainder`]

[`slice::chunks`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.chunks
[`slice::chunks_exact`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.chunks_exact
[`ChunksExact::remainder`]: https://doc.rust-lang.org/stable/std/slice/struct.ChunksExact.html#method.remainder

[`slice::rchunks`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.rchunks
[`slice::rchunks_exact`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.rchunks_exact
[`RChunksExact::remainder`]: https://doc.rust-lang.org/stable/std/slice/struct.RChunksExact.html#method.remainder

[`slice::chunks_mut`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.chunks_mut
[`slice::chunks_exact_mut`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.chunks_exact_mut
[`ChunksExactMut::into_remainder`]: https://doc.rust-lang.org/stable/std/slice/struct.ChunksExactMut.html#method.into_remainder

[`slice::rchunks_mut`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.rchunks_mut
[`slice::rchunks_exact_mut`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.rchunks_exact_mut
[`RChunksExactMut::into_remainder`]: https://doc.rust-lang.org/stable/std/slice/struct.RChunksExactMut.html#method.into_remainder

## `copied`

เมื่อต้องวนลูปบนคอลเลกชันของชนิดข้อมูลขนาดเล็ก เช่น จำนวนเต็ม (integers) การใช้ `iter().copied()` อาจให้ประสิทธิภาพดีกว่าการใช้ `iter()` ธรรมดา โค้ดส่วนที่นำอิเทอเรเตอร์ดังกล่าวไปใช้งานจะได้รับค่าจำนวนเต็มโดยตรง (by value) แทนที่จะเป็นค่าอ้างอิง (by reference) ซึ่งอาจช่วยให้ LLVM สามารถออปติไมซ์และสร้างโค้ดเครื่องออกมาได้มีประสิทธิภาพยิ่งขึ้น
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/issues/106539),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/issues/113789).

นี่เป็นเทคนิคขั้นสูง คุณอาจต้องตรวจสอบโค้ดเครื่องที่คอมไพเลอร์สร้างขึ้นเพื่อให้แน่ใจว่าส่งผลดีจริง สามารถดูรายละเอียดวิธีตรวจสอบได้ในบท[โค้ดเครื่อง](machine-code.md)
