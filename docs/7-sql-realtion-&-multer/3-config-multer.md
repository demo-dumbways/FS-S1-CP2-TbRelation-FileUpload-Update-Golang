---
sidebar_position: 3
---

# 3. Configuration Multer

Formulir postingan blog terdapat inputan yang isinya berupa file. Maka untuk menangani file upload didalam node js memerlukan bantuan package `Multer`. **Multer** adalah middleware node.js untuk menangani multipart/form-data, yang digunakan untuk mengunggah file.

menginstall package multer dapat menggunakan npm dengan command berikut

```shell
npm install multer
```

setelah menginstall multer, maka kita perlu melakukan konfigurasi terkait tempat penyimpanan file serta penamaan file yang akan diupload nantinya. Kita akan membuat sebuah file `uploadFile.js` didalam folder middlewares untuk melakukan proses konfigurasi.

pertama kita akan mengimport package multer kedalam file konfigurasi kita

```js title=uploadFile.js
const multer = require('multer');
```

selanjutnya kita akan melakukan konfigurasi tempat penyimpanan file yang diupload, kita akan menyimpan file yang diupload kedalam sebuah folder dengan nama `uploads`

```js {3-6} title=uploadFile.js
const multer = require('multer');

const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, 'uploads');
  },
});
```

selanjutnya kita akan melakukan konfigurasi terkait nama file, kita akan mengubah nama file yang diupload menjadi gabungan dari tanggal saat file di upload dan nama asli filenya. Hanya saja untuk nama asli file akan dihapus penggunaan whitespace menggunakan `regular expression`. Setelah melakukan konfigurasi maka kita export module.

<a class="btn-example-code" href="https://github.com/demo-dumbways/ebook-code-result-chapter-2/blob/day7-2.multer-config/middlewares/uploadFile.js">
Contoh code
</a>

<br />
<br />

```js {7-9,12-14} title=uploadFile.js
const multer = require('multer');

const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, 'uploads');
  },
  filename: function (req, file, cb) {
    cb(null, Date.now() + '-' + file.originalname.replace(/\s/g, ''));
  },
});

const upload = multer({ storage: storage });

module.exports = upload;
```
