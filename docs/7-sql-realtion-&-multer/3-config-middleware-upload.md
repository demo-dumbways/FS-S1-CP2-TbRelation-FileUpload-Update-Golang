---
sidebar_position: 3
---

# 3. Configuration Middleware Upload

Formulir postingan blog terdapat inputan yang isinya berupa file. Maka untuk menangani file upload pastikan terlebih dahulu file `form-blog.html` pada elemen `form` telah ditambahkan atribut `enctype="multipart/form-data"`.

```html {34} title="form-blog.html"
<!DOCTYPE html>
<html>

<head>
  <title>{{.Title}}</title>
  <link rel="stylesheet" href="/public/style.css">
  <!-- linking boostrap css cdn  -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet"
    integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
</head>

<body>
  <nav class="navbar navbar-expand-lg navbar-light bg-light">
    <div class="container-lg">
      <a class="navbar-brand me-5" href="/home">
        <img src="/public/assets/logo.png" alt="logo" />
      </a>
      <div class="collapse navbar-collapse" id="navbarNav">
        <ul class="navbar-nav">
          <li class="nav-item">
            <a class="nav-link" href="/home">Home</a>
          </li>
          <li class="nav-item">
            <a href="/blog" class="nav-link list-active">Blog</a>
          </li>
        </ul>
      </div>
      <div class="d-flex contact-me">
        <a href="/contact-me"> Contact Me </a>
      </div>
    </div>
  </nav>

  <form class="form-container" action="/blog" method="POST" enctype="multipart/form-data">
    <h1>Create Post Blog</h1>
    <div>
      <label for="input-title" class="form-label">Title</label>
      <input id="input-title" name="title" class="form-control" />
    </div>
    <div>
      <label for="input-content" class="form-label">Content</label>
      <textarea id="input-content" name="content" class="form-control"></textarea>
    </div>
    <div>
      <label for="input-image" class="form-label">Upload Image</label>
      <input class="form-control" type="file" id="input-image" name="input-image">
    </div>
    <div class="d-flex justify-content-end">
      <button type="submit" class="btn">Post Blog</button>
    </div>
  </form>
</body>

</html>
```
<br/>

Selanjutnya, kita akan membuat sebuah file `uploadFile.go` didalam folder middlewares untuk melakukan proses konfigurasi.

Kita akan melakukan konfigurasi tempat penyimpanan file yang diupload, kita akan menyimpan file yang diupload kedalam sebuah folder dengan nama `uploads`

Selain itu, kita jga akan melakukan konfigurasi terkait nama file, kita akan mengubah nama file yang diupload menjadi gabungan dari tanggal saat file di upload dan nama asli filenya.

<a class="btn-example-code" href="">
Contoh code
</a>

<br />
<br />

```go  title=uploadFile.go
package middleware

import (
	"context"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

func UploadFile(next http.HandlerFunc) http.HandlerFunc {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		file, handler, err := r.FormFile("input-image")
		if err != nil {
			fmt.Println(err)
			json.NewEncoder(w).Encode("Error Retrieving the File")
			return
		}
		defer file.Close()
		fmt.Printf("Uploaded File: %+v\n", handler.Filename)

		tempFile, err := ioutil.TempFile("uploads", "image-*"+handler.Filename)
		if err != nil {
			fmt.Println(err)
			fmt.Println("path upload error")
			json.NewEncoder(w).Encode(err)
			return
		}
		defer tempFile.Close()

		fileBytes, err := ioutil.ReadAll(file)
		if err != nil {
			fmt.Println(err)
		}

		tempFile.Write(fileBytes)

		data := tempFile.Name()
		filename := data[8:]

		ctx := context.WithValue(r.Context(), "dataFile", filename)
		next.ServeHTTP(w, r.WithContext(ctx))
	})
}
```