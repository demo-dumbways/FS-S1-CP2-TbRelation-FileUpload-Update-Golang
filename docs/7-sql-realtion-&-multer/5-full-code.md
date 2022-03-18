---
sidebar_position: 5
---

# 5. Full Code Index.JS

Berikut adalah Full Code pada file `index.js` :

<a class="btn-example-code" href="https://github.com/demo-dumbways/ebook-code-result-chapter-2/blob/day7-4.full-code/api/index.js">
Contoh code
</a>

<br />
<br />

```js title="index.js"
const express = require('express')
const path = require("path");
const bcrypt = require('bcrypt');
const flash = require('express-flash')
const session = require('express-session')

const db = require(path.join(__dirname, '../connection/db'));

const app = express()

app.use(flash())

app.use(
    session({
        cookie: {
            maxAge: 2 * 60 * 60 * 1000,
            secure: false,
            httpOnly: true,
        },
        store: new session.MemoryStore(),
        saveUninitialized: true,
        resave: false,
        secret: "secretValue",
    })
)

app.set('view engine', 'hbs');
app.set("views", path.join(__dirname, "../views"));

app.use("/public", express.static(path.join(__dirname, "../public")));

app.use(express.urlencoded({ extended: false }))

app.get('/', function (req, res) {
    res.send("Hello World")
})

app.get('/home', function (req, res) {
    setHeader(res)
    res.render('index', { isLogin: req.session.isLogin, user: req.session.user })
})

app.get('/blog', function (req, res) {
    setHeader(res)

    db.connect((err, client, done) => {
        if (err) throw err

        client.query('SELECT * FROM tb_blog', (err, result) => {
            done()
            if (err) throw

            let data = result.rows

            data = data.map((blog) => {
                return {
                    ...blog,
                    post_at: getFullTime(blog.post_at),
                    post_age: getDistanceTime(blog.post_at),
                    isLogin: req.session.isLogin,
                }
            })

            res.render(
                'blog',
                {
                    isLogin: req.session.isLogin,
                    blogs: data,
                    user: req.session.user
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

app.get('/add-blog', function (req, res) {
    setHeader(res)
    res.render("form-blog")
})

app.post('/blog', function (req, res) {
    let data = req.body

    if (!req.session.isLogin) {
        req.flash('danger', 'Please login')
        return res.redirect('/add-blog')
    }

    let query = `INSERT INTO blog(title, content, image) VALUES ('${data.title}', '${data.content}', 'image.png')`

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

app.get('/update-blog/:id', function (req, res) {
    let id = req.params.id
    let query = `UPDATE FROM blog WHERE id = ${id}`

    setHeader(res)
    db.connect(function (err, client, done) {
        done()
        if (err) throw err
        client.query(query, function (err, result) {
            if (err) throw err
            res.render('update-blog', { isLogin: req.session.isLogin, blog: result.rows[0] })
        })
    })
})

app.post('/update-blog/:id', function (req, res) {
    let id = req.params.id
    let data = req.body

    let query = `UPDATE blog SET title='${data.title}', content='${data.content}' WHERE id='${id}'`

    db.connect(function (err, client, done) {
        if (err) throw err

        client.query(query, function (err, result) {
            done()
            if (err) throw err
            res.redirect('/blog')
        })
    })
})

app.get('/contact-me', function (req, res) {
    setHeader(res)
    res.render('contact')
})

app.get('/register', function (req, res) {
    setHeader(res)
    res.render('register')
})

app.post('/register', function (req, res) {
    const { email, name, password } = req.body

    const hashedPassword = bcrypt.hashSync(password, 10)

    let query = `INSERT INTO tb_user(name, email, password) VALUES('${name}', '${email}', '${hashedPassword}')`

    db.connect(function (err, client, done) {
        if (err) throw err

        client.query(query, function (err, result) {
            if (err) throw err
            res.redirect('/login')
        })
    })
})

app.get('/login', function (req, res) {
    setHeader(res)
    res.render('login')
})

app.post('/login', function (req, res) {
    const { email, password } = req.body

    let query = `SELECT * FROM tb_user WHERE email = '${email}'`

    db.connect(function (err, client, done) {
        if (err) throw err

        client.query(query, function (err, result) {
            if (err) throw err

            if (result.rows.length == 0) {
                req.flash('danger', "Email & Password don't match!")
                return res.redirect('/login')
            }

            let isMatch = bcrypt.compareSync(password, result.rows[0].password)

            if (isMatch) {
                req.session.isLogin = true
                req.session.user = {
                    id: result.rows[0].id,
                    name: result.rows[0].name,
                    email: result.rows[0].email
                }

                req.flash('success', "Login success")
                res.redirect('/blog')
            } else {
                req.flash('danger', "Email & Password don't match!")
                res.redirect('/login')
            }
        })
    })
})

const port = 5000
app.listen(port, function () {
    console.debug(`Server running on port ${port}`)
})

function setHeader(res) {
    res.setHeader("Content-Type", "text/html");
    res.setHeader("Cache-Control", "s-max-age=1, stale-while-revalidate");
}
```
