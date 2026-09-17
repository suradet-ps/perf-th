# การบันทึกและการดีบัก

บางครั้งโค้ดสำหรับการบันทึก log หรือโค้ดสำหรับดีบัก (debug) อาจทำให้โปรแกรมทำงานช้าลงอย่างมีนัยสำคัญ ไม่ว่าจะเป็นตัวโค้ดบันทึก log/ดีบักเองที่ทำงานช้า หรือโค้ดส่วนที่รวบรวมข้อมูลเพื่อส่งต่อไปยังการบันทึก log/ดีบักนั้นทำงานช้า ดังนั้น จึงควรตรวจสอบให้แน่ใจว่าโปรแกรมไม่มีการประมวลผลใดๆ ที่ไม่จำเป็นสำหรับการบันทึก log หรือการดีบัก ในช่วงเวลาที่ฟังก์ชันบันทึก log หรือดีบักนั้นไม่ได้ถูกเปิดใช้งาน
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/pull/50246/commits/2e4f66a86f7baa5644d18bb2adc07a8cd1c7409d),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/pull/75133/commits/eeb4b83289e09956e0dda174047729ca87c709fe),
[**ตัวอย่างที่ 3**](https://github.com/rust-lang/rust/pull/147293/commits/cb0f969b623a7e12a0d8166c9a498e17a8b5a3c4)

โปรดทราบว่าคำสั่ง [`assert!`] จะถูกเอ็กซีคิวต์เสมอในทุกบิลด์ แต่คำสั่ง [`debug_assert!`] จะทำงานเฉพาะในบิลด์แบบ dev เท่านั้น หากคุณมี assertion ที่อยู่ใน hot path แต่ไม่ได้จำเป็นต่อความปลอดภัย (safety) ของระบบ ลองพิจารณาเปลี่ยนไปใช้ `debug_assert!` แทน
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/pull/58210/commits/f7ed6e18160bc8fccf27a73c05f3935c9e8f672e),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/pull/90746/commits/580d357b5adef605fc731d295ca53ab8532e26fb)

[`assert!`]: https://doc.rust-lang.org/std/macro.assert.html
[`debug_assert!`]: https://doc.rust-lang.org/std/macro.debug_assert.html
