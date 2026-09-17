# การบันทึกและการดีบัก

บางครั้งโค้ดบันทึก log หรือโค้ดดีบักอาจทำให้โปรแกรมช้าลงอย่างมีนัยสำคัญ ไม่ใช่โค้ดบันทึก/ดีบักเองที่ช้า ก็เป็นโค้ดเก็บข้อมูลที่ป้อนเข้าสู่โค้ดบันทึก/ดีบักนั้นที่ช้า ตรวจสอบให้แน่ใจว่าไม่มีการทำงานที่ไม่จำเป็นเพื่อจุดประสงค์ในการบันทึก/ดีบัก เมื่อไม่ได้เปิดใช้งานการบันทึก/ดีบัก
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/pull/50246/commits/2e4f66a86f7baa5644d18bb2adc07a8cd1c7409d),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/pull/75133/commits/eeb4b83289e09956e0dda174047729ca87c709fe),
[**ตัวอย่างที่ 3**](https://github.com/rust-lang/rust/pull/147293/commits/cb0f969b623a7e12a0d8166c9a498e17a8b5a3c4).

โปรดทราบว่าการเรียก [`assert!`] ทำงานเสมอ แต่การเรียก [`debug_assert!`] ทำงานเฉพาะในบิลด์แบบเดฟ หากคุณมีแอสเซอร์ชันที่ฮอตแต่ไม่จำเป็นต่อความปลอดภัย ลองพิจารณาเปลี่ยนให้เป็น `debug_assert!`
[**ตัวอย่างที่ 1**](https://github.com/rust-lang/rust/pull/58210/commits/f7ed6e18160bc8fccf27a73c05f3935c9e8f672e),
[**ตัวอย่างที่ 2**](https://github.com/rust-lang/rust/pull/90746/commits/580d357b5adef605fc731d295ca53ab8532e26fb).

[`assert!`]: https://doc.rust-lang.org/std/macro.assert.html
[`debug_assert!`]: https://doc.rust-lang.org/std/macro.debug_assert.html
