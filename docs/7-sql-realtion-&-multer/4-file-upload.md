---
sidebar_position: 4
---

# 4. File Upload

import useBaseUrl from '@docusaurus/useBaseUrl';

Kita akan menjadikan static folder uploads agar bisa diakses.

Kemudian kita menggunakan `middleware uploadFile` yang telah dikonfigurasi, maka kita perlu menambahkan nya pada saat endpoint blog diakses untuk melakukan penambahan data blog.

```go {12,20} title="main.go"
// .............
// continuation this code same like before
// .............

func main() {
	route := mux.NewRouter()

	connection.DatabaseConnect()

	// static folder
	route.PathPrefix("/public/").Handler(http.StripPrefix("/public/", http.FileServer(http.Dir("./public/"))))
	route.PathPrefix("/uploads/").Handler(http.StripPrefix("/uploads/", http.FileServer(http.Dir("./uploads/"))))

	// routing
	route.HandleFunc("/", helloWorld).Methods("GET")
	route.HandleFunc("/home", home).Methods("GET").Name("home")
	route.HandleFunc("/blog", blogs).Methods("GET")
	route.HandleFunc("/blog/{id}", blogDetail).Methods("GET")
	route.HandleFunc("/add-blog", formBlog).Methods("GET")
	route.HandleFunc("/blog", middleware.UploadFile(addBlog)).Methods("POST")
	route.HandleFunc("/delete-blog/{id}", deleteBlog).Methods("GET")

	route.HandleFunc("/contact-me", contactMe).Methods("GET")

	route.HandleFunc("/register", formRegister).Methods("GET")
	route.HandleFunc("/register", register).Methods("POST")

	route.HandleFunc("/login", formLogin).Methods("GET")
	route.HandleFunc("/login", login).Methods("POST")

	route.HandleFunc("/logout", logout).Methods("GET")

	fmt.Println("Server running on port 5000")
	http.ListenAndServe("localhost:5000", route)
}

// .............
// continuation this code same like before
// .............
```

selanjutnya kita akan merefactor function `addBlog` agar mengambil data filename yang berasal dari file upload untuk disimpan kedalam database, selain itu kita juga akan mengambil data `id user` yang sedang login untuk disimpan kedalam database sebagai data `author_id`.

<a class="btn-example-code" href="https://github.com/demo-dumbways/ebook-code-result-chapter-2-golang/blob/day7-3-file-upload/main.go">
Contoh code
</a>

<br />
<br />

```go {14-24} title="main.go"
// .............
// continuation this code same like before
// .............

func addBlog(w http.ResponseWriter, r *http.Request) {
	err := r.ParseForm()
	if err != nil {
		log.Fatal(err)
	}

	title := r.PostForm.Get("title")
	content := r.PostForm.Get("content")

	dataContex := r.Context().Value("dataFile")
	image := dataContex.(string)

	var store = sessions.NewCookieStore([]byte("SESSION_ID"))
	session, _ := store.Get(r, "SESSION_ID")

	author := session.Values["Id"].(int)

	_, err = connection.Conn.Exec(context.Background(),
            "INSERT INTO tb_blog(title, content,image,author_id)
                VALUES ($1,$2,$3,$4)", title, content, image, author)

	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		w.Write([]byte("message : " + err.Error()))
		return
	}

	http.Redirect(w, r, "/blog", http.StatusMovedPermanently)
}

// .............
// continuation this code same like before
// .............
```

Pastikan untuk menambahkan Id pada login agar kita bisa secara otomatis meng-set id saat kita post blog berdasarkan id user yang kita login, sebagai berikut :

```go {36} title="main.go"
// .............
// continuation this code same like before
// .............
func login(w http.ResponseWriter, r *http.Request) {
	err := r.ParseForm()
	if err != nil {
		log.Fatal(err)
	}

	email := r.PostForm.Get("email")
	password := r.PostForm.Get("password")

	user := User{}

	err = connection.Conn.QueryRow(context.Background(), "SELECT * FROM tb_user WHERE email=$1", email).Scan(
		&user.Id, &user.Name, &user.Email, &user.Password,
	)
	if err != nil {
		w.WriteHeader(http.StatusBadRequest)
		w.Write([]byte("message : " + err.Error()))
		return
	}

	err = bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(password))
	if err != nil {
		w.WriteHeader(http.StatusBadRequest)
		w.Write([]byte("message : " + err.Error()))
		return
	}

	var store = sessions.NewCookieStore([]byte("SESSION_ID"))
	session, _ := store.Get(r, "SESSION_ID")

	session.Values["IsLogin"] = true
	session.Values["Name"] = user.Name
	session.Values["Id"] = user.Id
	session.Options.MaxAge = 10800

	session.AddFlash("Login success", "message")
	session.Save(r, w)

	http.Redirect(w, r, "/home", http.StatusMovedPermanently)
}
// .............
// continuation this code same like before
// .............
```

Kemudian kita coba ubah pemanggilan image sebagai berikut :

```go {7-10} title="blog.html"
// .............
// continuation this code same like before
// .............
{{range $index, $data := .Blogs}}
	<div class="blog-list-item">
		<div class="blog-image">
			<img
				src="http://localhost:5000/uploads/{{$data.Image}}"
				alt="Pasar Coding di Indonesia Dinilai Masih Menjanjikan"
			/>
		</div>
		<div class="blog-content">
			{{if $.Data.IsLogin}}
			<div class="button-group">
				<a class="btn-edit">Edit Post</a>
				<a class="btn-post" href="/delete-blog/{{$data.Id}}">Delete Blog</a>
			</div>
			{{end}}
			<h1>
				<a href="/blog/{{$data.Id}}" target="_blank"> {{$data.Title}} </a>
			</h1>
			<div class="detail-blog-content">
				{{$data.Post_date}} | {{$data.Author}}
			</div>
			<p>{{$data.Content}}</p>
		</div>
	</div>
{{end}}
// .............
// continuation this code same like before
// .............
```

<img alt="image1" src={useBaseUrl('img/docs/image-7-3.png')} height="600px"/>

<br />
<br />

<div>
<a class="btn-demo" href="">
Demo
</a>
</div>
