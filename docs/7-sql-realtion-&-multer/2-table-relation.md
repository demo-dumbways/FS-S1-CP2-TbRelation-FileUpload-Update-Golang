---
sidebar_position: 2
---

# 2. Table Relation

import useBaseUrl from '@docusaurus/useBaseUrl';

**Table relation** adalah sebuah konsep database dimana sebuah database terdiri dari beberapa tabel yang saling terkait. Pada case kali ini kita menghubungkan antar tabel user dengan tabel blog. 

<img alt="image1" src={useBaseUrl('img/docs/image-7-1.png')} height="300px"/>

pada gambar diatas menunjukkan relasi yang menghubungkan tabel user dengan tabel blog adalah field dengan nama `user_id`. Field user_id yang terdapat di tabel blog merupakan sebuah foreign key. **Foreign key** merupakan suatu field dalam satu tabel yang digunakan untuk menghubungkan dua tabel. Foreign key merujuk pada primary key tabel lainnya, dalam hal ini adalah field `id` di tabel user.

Menampilkan nama user yang memposting blog maka kita perlu melakukan query multi table dengan memanfaatkan table ralation. Kita akan melakukan sedikit refactor pada query pada saat melakukan rendering tampilan blog di route `/blog`

<a class="btn-example-code" href="https://github.com/demo-dumbways/ebook-code-result-chapter-2/blob/day7-1.table-relation/api/index.js">
Contoh code
</a>

<br />
<br />

```js {9-11} title=index.js
app.get('/home', function (req, res) {
    setHeader(res)
    res.render('index', { isLogin: req.session.isLogin, user: req.session.user })
})

app.get('/blog', function (req, res) {
    setHeader(res)

    let query = `SELECT blog.id, blog.title, blog.content, blog.image, tb_user.name AS author, blog.author_id, blog.post_at
                    FROM blog LEFT JOIN tb_user
                    ON blog.author_id = tb_user.id`

    db.connect((err, client, done) => {
        if (err) throw err

        client.query(query, (err, result) => {
            done()
            if (err) throw

            let data = result.rows

            data = data.map((blog) => {
                return {
                    ...blog,
                    post_at: getFullTime(blog.post_at),
                    post_age: getDistanceTime(blog.post_at),
                    isLogin: req.session.isLogin
                }
            })

            res.render(
                'blog',
                {
                    isLogin: req.session.isLogin,
                    user: req.session.user,
                    blogs: data
                })
        })
    })
})

app.get('/blog/:id', function (req, res) {
    const blogId = req.params.id

    setHeader(res)
    db.connect((err, client, done) => {
        if (err) throw err

        client.query(`SELECT * FROM blog WHERE id = ${id}`, function (err, result) {
            done()
            if (err) throw err

            res.render('blog-detail', { isLogin: req.session.isLogin, blog: result.rows[0] })
        })
    })
})
```


<img alt="image1" src={useBaseUrl('img/docs/image-7-2.png')} height="600px"/>

<br />
<br />

<div>
<a class="btn-demo" href="https://personal-web-chapter-2.herokuapp.com/blog">
Demo
</a>
</div>