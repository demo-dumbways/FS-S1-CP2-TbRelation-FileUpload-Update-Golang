---
sidebar_position: 4
---

# 4. File Upload

import useBaseUrl from '@docusaurus/useBaseUrl';

Menggunakan multer yang telah dikonfigurasi, maka kita perlu mengimportnya kedalam file index.js

```js
const upload = require(path.join(__dirname, './middlewares/uploadFile'))
```

selanjutnya kita akan merefactor route `/blog` dengan method `post` agar bisa menangani file upload menggunakan multer. Kita menambahkan penggunaan middleware upload serta mengambil nama file yang diupload untuk disimpan kedalam database.

<a class="btn-example-code" href="https://github.com/demo-dumbways/ebook-code-result-chapter-2/blob/day7-3.file-upload/api/index.js">
Contoh code
</a>

<br />
<br />

```js {7,16,18-19} title=index.js
// this code below endpoint app.get('/blog/:id', function (req, res)
app.get('/add-blog', function (req, res) {
    setHeader(res)
    res.render("form-blog")
})

app.post('/blog', upload.single('image'), function (req, res) {
    let data = req.body

    if (!req.session.isLogin) {
        req.flash('danger', 'Please login')
        return res.redirect('/add-blog')
    }

    let authorId = req.session.user.id
    let image = req.file ? req.file.filename : null

    let query = `INSERT INTO blog(title, content, image, author_id) VALUES 
                ('${data.title}', '${data.content}', '${image}', '${authorId}')`

    db.connect(function (err, client, done) {
        if (err) throw err

        client.query(query, function (err, result) {
            if (err) throw err
            res.redirect('/blog')
        })
    })
})

app.get('/delete-blog/:id', function (req, res) {
    let id = req.params.id
    let query = `DELETE FROM blog WHERE id = ${id}`

    setHeader(res)
    db.connect(function (err, client, done) {
        done()
        if (err) throw err
        client.query(query, function (err, result) {
            if (err) throw err
            res.redirect('/blog')
        })
    })
})
```

<img alt="image1" src={useBaseUrl('img/docs/image-7-3.png')} height="600px"/>

<br />
<br />

<div>
<a class="btn-demo" href="https://personal-web-chapter-2.herokuapp.com/add-blog">
Demo
</a>
</div>