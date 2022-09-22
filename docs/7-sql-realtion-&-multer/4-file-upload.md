---
sidebar_position: 4
---

# 4. File Upload

import useBaseUrl from '@docusaurus/useBaseUrl';

Menggunakan `middleare uploadFile` yang telah dikonfigurasi, maka kita perlu menambahkan nya pada saat endpoint blog diakses untuk melakukan penambahan data blog

```go {20} title="main.go"
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

<a class="btn-example-code" href="">
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
            "INSERT INTO blog(title, content,image,author_id) 
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

<img alt="image1" src={useBaseUrl('img/docs/image-7-3.png')} height="600px"/>

<br />
<br />

<div>
<a class="btn-demo" href="">
Demo
</a>
</div>