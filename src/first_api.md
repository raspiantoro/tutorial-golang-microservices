
# First API

<div style="text-align: justify">

Untuk memulai pengembangan REST API menggunakan Chi, kita perlu *meng-install* package chi pada service yang akan kita bangun. Pada chapter sebelumnya, kita sudah membuat satu service bernama product-svc. Sebelum *meng-install* chi, pastikan didalam terminal yang kamu gunakan sudah berada difolder product-svc, jalankan perintah berikut untuk berpindah dari root direktori workspace kamu

```bash
cd ./product-svc
```

Jika kamu sudah berada didalam folder product-svc, ketikan perintah ini untuk meng-install chi kedalam service kamu.

```bash
go get -u github.com/go-chi/chi/v5
```

Buka file main.go yang sudah kamu buat sebelumnya pada folder product-svc untuk *meng-import* package chi, json dan http

```go
import (
    "encoding/json"
    "net/http"
	"github.com/go-chi/chi/v5"
)
```

Masih pada file main.go tambahkan baris code berikut untuk membuat type baru dengan nama ProductCategory.

```go
type ProductCategory struct {
	ID   uint   `json:"id"`
	Name string `json:"name"`
}
```

Selanjutnya pada function main, kita perlu membuat chi router untuk *meng-handle* setiap requests yang datang dari users.

```go
function main(){
    r := chi.NewRouter()
}
```

Baris diatas kita membuat satu instance router yang kita simpan didalam variable `r`. Selanjutnya kita perlu membuat endpoint `/api/product-category` dengan method GET untuk mengembalikan daftar kategori product kepada users. Tambahkan baris berikut setelah baris instansiasi chi router.

```go
    r.Get("/api/product-category", func(w http.ResponseWriter, r *http.Request) {
		// langkah pertama
        categories := []ProductCategory{
			{
				ID:   1,
				Name: "Alat Tulis",
			},
			{
				ID:   2,
				Name: "Perlengkapan Rumah Tangga",
			},
			{
				ID:   3,
				Name: "Fashion",
			},
		}

        // langkah kedua
		result, err := json.Marshal(&categories)
		if err != nil {
			w.WriteHeader(http.StatusInternalServerError)
			w.Write([]byte("Terjadi kesalahan pada server"))
			return
		}

        // langkah ketiga
		w.WriteHeader(http.StatusOK)
		w.Write(result)
	})
```

Pada baris kode diatas, kita membuat sebuah endpoint GET menggunakan method (function) Get dari chi router. Method Get pada chi router menerima dua parameter sebagai inputan, dimana parameter pertama berupa url path dari endpoint yang kita buat (`/api/product-category`), dan parameter kedua berupa closure (function) yang akan dieksekusi oleh chi untuk menghandle setiap request yang datang kepada path `/api/product-category`.

Terdapat tiga langkah yang kita tulis dalam closure yang kita buat untuk menangani requests dari users. Berikut penjelasan dari masing-masing langkah yang kita buat:  
1. Pada langkah pertama kita menyiapakan daftar kategori produk dengan menggunakan slice (saat ini kita membuat daftar kategori secara hardcoded, pada chapter integrasi dengan database kita akan belajar bagaimana menampilkan daftar produk yang ada di database)
2. Sebelum dapat mengembalikan slice daftar kategori kepada users, kita harus melakukan serialisasi (mengubah object menjadi rangkaian bytes) terlebih dahulu terhadap slice yang sudah kita buat. Untuk melakukannya kita menggunakan function **Marshal** dari package **encoding/json**. Function Marshal menerima parameter berupa pointer dari data yang akan kita serialisasi, dan function tersebut mengembalikan dua return value berupa byte slice dan error. Jika terdapat error pada saat melakukan serialisasi, kita akan mengembalikan *response* kepada user dengan status code **500** dan pesan kesalahan **Terjadi kesalahan pada server**. Selain itu kita juga melakukan *early return* saat terjadi error dengan tujuan mencegah agar kode pada langkah ketiga tidak ikut dieksekusi.
3. Jika proses serialisasi berhasil, kita akan mengembalikan *response* sukses kepada user dengan status code **200** dan daftar kategori produk dalam bentuk json.

Selanjutnya kita perlu melakukan listen dan serve terhadap setiap request http yang datang. Tambahkan baris berikut pada function main

```go
    fmt.Println("your server is running on port 8080")
	http.ListenAndServe(":8080", r)
```

Berikut adalah kode lengkap dari seluruh yang kita buat diatas

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"

	"github.com/go-chi/chi/v5"
)

type ProductCategory struct {
	ID   uint   `json:"id"`
	Name string `json:"name"`
}

func main() {
	r := chi.NewRouter()
	r.Get("/api/product-category", func(w http.ResponseWriter, r *http.Request) {
		categories := []ProductCategory{
			{
				ID:   1,
				Name: "Alat Tulis",
			},
			{
				ID:   2,
				Name: "Perlengkapan Rumah Tangga",
			},
			{
				ID:   3,
				Name: "Fashion",
			},
		}

		result, err := json.Marshal(&categories)
		if err != nil {
			w.WriteHeader(http.StatusInternalServerError)
			w.Write([]byte("Terjadi kesalahan pada server"))
			return
		}

		w.WriteHeader(http.StatusOK)
		w.Write(result)
	})

	fmt.Println("your server is running on port 8080")
	http.ListenAndServe(":8080", r)
}
```

Ketik perintah berikut untuk menjalankan service melalui root folder worspace (./golang-marketplace) kamu
```bash
go run ./product-svc
```

Kamu bisa menggunakan curl untuk melakukan testing terhadap endpoint yang sudah kamu buat
```bash
curl http://localhost:8080/api/product-category
```

Berikut adalah response yang didapat dari endpoint yang sudah kita buat

```
[{"id":1,"name":"Alat Tulis"},{"id":2,"name":"Perlengkapan Rumah Tangga"},{"id":3,"name":"Fashion"}]
```

Kode lengkap pada chapter ini dapat kamu lihat [disini](https://github.com/raspiantoro/golang-microservices-source-code/tree/chapter-02.01) dan untuk perubahan/penambahan file yang kita lakukan, detailnya dapat kamu lihat pada link [Pull Request](https://github.com/raspiantoro/golang-microservices-source-code/pull/2/files) ini 

</div>
