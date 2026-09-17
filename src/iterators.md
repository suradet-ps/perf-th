# อิเทอเรเตอร์

## `collect` และ `extend`

[`Iterator::collect`] แปลงอิเทอเรเตอร์ให้เป็นคอลเลกชันอย่าง `Vec` ซึ่งโดยทั่วไปต้องใช้การจัดสรรหน่วยความจำ คุณควรหลีกเลี่ยงการเรียก `collect` หากหลังจากนั้นคอลเลกชันจะถูกวนซ้ำอีกครั้งเท่านั้น

[`Iterator::collect`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect

ด้วยเหตุนี้ การคืนชนิดอิเทอเรเตอร์อย่าง `impl Iterator<Item=T>` จากฟังก์ชันจึงมักดีกว่าการคืน `Vec<T>` โปรดทราบว่าบางครั้งชนิดข้อมูลที่คืนเหล่านี้อาจต้องมีไลฟ์ไทม์เพิ่มเติม ดังที่[บทความบล็อกนี้]อธิบาย
[**ตัวอย่าง**](https://github.com/rust-lang/rust/pull/77990/commits/660d8a6550a126797aa66a417137e39a5639451b).

[บทความบล็อกนี้]: https://blog.katona.me/2019/12/29/Rust-Lifetimes-and-Iterators/

ในทำนองเดียวกัน คุณสามารถใช้ [`extend`] เพื่อขยายคอลเลกชันที่มีอยู่ (เช่น `Vec`) ด้วยอิเทอเรเตอร์ แทนที่จะ collect อิเทอเรเตอร์ลงใน `Vec` แล้วใช้ [`append`]

[`extend`]: https://doc.rust-lang.org/std/iter/trait.Extend.html#tymethod.extend
[`append`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.append

สุดท้าย เมื่อคุณเขียนอิเทอเรเตอร์ มักคุ้มค่าที่จะอิมพลีเมนต์เมธอด [`Iterator::size_hint`] หรือ [`ExactSizeIterator::len`] หากเป็นไปได้ การเรียก `collect` และ `extend` ที่ใช้อิเทอเรเตอร์นั้นอาจจัดสรรหน่วยความจำน้อยลง เพราะมีข้อมูลล่วงหน้าเกี่ยวกับจำนวนองค์ประกอบที่อิเทอเรเตอร์จะให้

[`Iterator::size_hint`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.size_hint
[`ExactSizeIterator::len`]: https://doc.rust-lang.org/std/iter/trait.ExactSizeIterator.html#method.len

## การเชื่อมต่อ

[`chain`] สะดวกมาก แต่ก็อาจช้ากว่าอิเทอเรเตอร์ตัวเดียว หากเป็นไปได้อาจคุ้มค่าที่จะหลีกเลี่ยงสำหรับอิเทอเรเตอร์ที่ฮอต
[**ตัวอย่าง**](https://github.com/rust-lang/rust/pull/64801/commits/5ca99b750e455e9b5e13e83d0d7886486231e48a).

ในทำนองเดียวกัน [`filter_map`] อาจเร็วกว่าการใช้ [`filter`] ตามด้วย [`map`]

[`chain`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.chain
[`filter_map`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter_map
[`filter`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter
[`map`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.map

## ชังก์

เมื่อต้องใช้อิเทอเรเตอร์แบบแบ่งชังก์ และทราบว่าขนาดชังก์หารความยาวสไลซ์ลงตัวพอดี ให้ใช้ [`slice::chunks_exact`] ที่เร็วกว่าแทน [`slice::chunks`]

เมื่อไม่ทราบว่าขนาดชังก์หารความยาวสไลซ์ลงตัวพอดีหรือไม่ การใช้ `slice::chunks_exact` ร่วมกับ [`ChunksExact::remainder`] หรือการจัดการองค์ประกอบส่วนเกินด้วยมือก็ยังอาจเร็วกว่าได้
[**ตัวอย่างที่ 1**](https://github.com/johannesvollmer/exrs/pull/173/files),
[**ตัวอย่างที่ 2**](https://github.com/johannesvollmer/exrs/pull/175/files).

เช่นเดียวกันกับอิเทอเรเตอร์ที่เกี่ยวข้อง:
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

เมื่อวนซ้ำบนคอลเลกชันของชนิดข้อมูลขนาดเล็ก เช่นจำนวนเต็ม การใช้ `iter().copied()` แทน `iter()` อาจดีกว่า สิ่งใดก็ตามที่บริโภคอิเทอเรเตอร์นั้นจะได้รับจำนวนเต็มเป็นค่าแทนที่จะเป็นค่าอ้างอิง และ LLVM อาจสร้างโค้ดได้ดีกว่าในกรณีนั้น
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/issues/106539),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/issues/113789).

นี่เป็นเทคนิคขั้นสูง คุณอาจต้องตรวจสอบโค้ดเครื่องที่ถูกสร้างขึ้นเพื่อให้แน่ใจว่ามันได้ผล ดูรายละเอียดวิธีทำได้ในบท[โค้ดเครื่อง](machine-code.md)
