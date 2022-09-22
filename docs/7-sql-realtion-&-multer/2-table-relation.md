---
sidebar_position: 2
---

# 2. Table Relation

import useBaseUrl from '@docusaurus/useBaseUrl';

**Table relation** adalah sebuah konsep database dimana sebuah database terdiri dari beberapa tabel yang saling terkait. Pada case kali ini kita menghubungkan antar tabel user dengan tabel blog. 

<img alt="image1" src={useBaseUrl('img/docs/image-7-1.png')} height="300px"/>

pada gambar diatas menunjukkan relasi yang menghubungkan tabel user dengan tabel blog adalah field dengan nama `user_id`. Field user_id yang terdapat di tabel blog merupakan sebuah foreign key. **Foreign key** merupakan suatu field dalam satu tabel yang digunakan untuk menghubungkan dua tabel. Foreign key merujuk pada primary key tabel lainnya, dalam hal ini adalah field `id` di tabel user.

Menampilkan nama user yang memposting blog maka kita perlu melakukan query multi table dengan memanfaatkan table ralation. Kita akan melakukan sedikit refactor pada query pada saat melakukan rendering tampilan blog di route `/blog`

<a class="btn-example-code" href="">
Contoh code
</a>

<br />
<br />

```go {25-30} title=main.go
// .............
// continuation this code same like before 
// .............

func blogs(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "text/html; charset=utf-8")

	var tmpl, err = template.ParseFiles("views/blog.html")
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		w.Write([]byte("message : " + err.Error()))
		return
	}

	var store = sessions.NewCookieStore([]byte("SESSION_ID"))
	session, _ := store.Get(r, "SESSION_ID")

	if session.Values["IsLogin"] != true {
		Data.IsLogin = false
	} else {
		Data.IsLogin = session.Values["IsLogin"].(bool)
		Data.UserName = session.Values["Name"].(string)
	}

	rows, _ := connection.Conn.Query(context.Background(), 
                "SELECT blog.id, title, image, content, post_at, users.name as author 
                    FROM blog 
                    LEFT JOIN users 
                    ON blog.author_id = users.id  
                    ORDER BY id DESC")

	var result []Blog
	for rows.Next() {
		var each = Blog{}

		var err = rows.Scan(&each.Id, &each.Title, &each.Image, &each.Content, &each.Post_date, &each.Author)
		if err != nil {
			fmt.Println(err.Error())
			return
		}

		each.Format_date = each.Post_date.Format("2 January 2006")

		if session.Values["IsLogin"] != true {
			each.IsLogin = false
		} else {
			each.IsLogin = session.Values["IsLogin"].(bool)
		}

		result = append(result, each)
	}

	fmt.Println(result)
	respData := map[string]interface{}{
		"Data":  Data,
		"Blogs": result,
	}

	w.WriteHeader(http.StatusOK)
	tmpl.Execute(w, respData)
}

// .............
// continuation this code same like before 
// .............
```


<img alt="image1" src={useBaseUrl('img/docs/image-7-2.png')} height="600px"/>

<br />
<br />

<div>
<a class="btn-demo" href="https://personal-web-chapter-2.herokuapp.com/blog">
Demo
</a>
</div>