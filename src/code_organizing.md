# Code Organizing

<div style="text-align: justify">

Pada chapter sebelumnya kita telah membuat sebuah endpoint sederhana yang berfungsi untuk mengambil daftar kategori produk. Kode yang kita buat sebelumnya sudah berjalan sesuai yang kita harapkan, tetapi kita menulis semuanya didalam satu file `main.go`. Saat kode kita semakin besar, tentunya cara ini akan menyulitkan kita untuk memantain seluruh kode yang kita tulis. 

Pada chapter ini kita akan membagi kode yang sudah kita buat kedalam beberapa direktori dan file yang berbeda sesuai dengan tanggung jawab masing-masing baris kode yang kita tulis. Berikut struktur direketori yang akan kita gunakan untuk product-svc dan service-service lainnya yang akan kita buat.

```
└── product-svc
    ├── go.mod
    ├── go.sum
    ├── internal
    │   ├── contract
    │   ├── handler
    │   ├── model
    │   ├── payload
    │   ├── repository
    │   ├── router
    │   └── service
    └── main.go
```

Pada struktur direktori diatas kita akan membagi kode kita kedalam beberapa layer yaitu contract, handler, model, repository, service dan payload. Masing-masing layer akan memiliki tanggung jawabnya masing-masing. Berikut penjelasan dari masing-masing direktori yang ada:  
1. `internal`  
Umumnya pada aplikasi golang kita akan menyimpan sebagian besar kode kita didalam direktori `internal`. Tujuan penggunaan direktori internal adalah untuk memproteksi agar seluruh kode yang ada didalam direktori ini tidak dapat digunakan oleh module/package lain.  

2. `internal/contract`  
Setiap layer pada service yang akan kita buat akan memiliki dependency terhadap satu atau lebih layer lainnya, sebagai contoh layer handle akan memiliki dependency terhadap layer service dan layer service akan memiliki dependency terhadap layer repository. Untuk flexibiltas, kita akan menghindari direct dependencies antara satu layer dengan concrete implementation pada layer lainnya. Layer contract disini berperan sebagai media antar layer, dan berisi interface yang menentukan bagaimana concrete implementation pada layer lainnya harus dibuat, sehingga layer tersebut dinamakan sebagai layer `contract` 

3. `internal/handler`   
Layer handler akan bertanggung jawab menangani dan memvalidasi setiap requests yang datang dari user.  

4. `internal/model`  
Model merupakan representasi dari table yang ada pada database, dan dapat juga berupa representasi dari output dari query yang kita buat.  

5. `internal/payload`
Hampir sama dengan model, payload akan teridiri dari kumpulan struct yang merepresentasikan request dan response body yang datang melalui HTTP Requests.

6. `internal/repository`
Repository bertanggung jawab untuk melakukan persistence logic yang umumnya berupa operasi pada database seperti create, read, update dan delete.

7. `internal/router`
Router berisi kumpulan routing method yang akan kita gunakan pada service kita.

8. `internal/service`
Service layer biasa juga disebut sebaga use case layer, yang berisi bisnis logic utama dari service yang kita buat.

Untuk membagi kode yang sudah kita buat sebelumnya kedalam masing-masing layer, kita dapat memulai dengan memindahkan struct ProductCategory yang ada di line 11 - 13 pada file main.go ke layer payload. Buat file `response.go` didalam direktori payload, pindahkan line 11 - 13 pada file main.go dan ubah nama struct ProductCategory menjadi `GetProductCategoryResponse`

```go
package payload

type GetProductCategoryResponse struct {
	ID   uint   `json:"id"`
	Name string `json:"name"`
}
```

Selanjutnya buat file `service.go` pada directory `internal/contract`, dan tambahkan interface `ProductCategoryService` seperti code dibawah.

```go
package contract

import (
	"context"

	"github.com/raspiantoro/golang-marketplace/product-svc/internal/payload"
)

type ProductCategoryService interface {
	GetAll(ctx context.Context) ([]payload.GetProductCategoryResponse, error)
}
```

Pada direktori handle, buatlah file `product_category.go`. Selanjutnya buat struct dengan nama ProductCategory beserta dengan constructor-nya

```go
package handler

import "github.com/raspiantoro/golang-marketplace/product-svc/internal/contract"

type ProductCategory struct {
	productCategoryServices contract.ProductCategoryService
}

func NewProductCategory(productCategoryServices contract.ProductCategoryService) *ProductCategory {
	return &ProductCategory{
		productCategoryServices: productCategoryServices,
	}
}
```

Jika kamu perhatikan, struct ProductCategory pada package handler memiliki field `productCategoryService` dengan type interface `contract.ProductCategoryService`. Dengan cara ini, memungkinkan bagi kita untuk meng-inject field tersebut melalui function `NewProductCategory` dengan concrete type apapun selama type tersebut mengimplement semua method yang ada pada interface `contract.ProductCategoryService`, teknik ini biasa disebut dengan `Dependency Injection`[^note1]. Dengan begini kita akan memiliki fleksibilitas untuk memilih salah satu dari banyak concrete type yang sesuai dengan kebutuhan kita, kita akan melihat lebih lanjut kegunaan dari teknik ini saat kita mulai membuat unit tests pada kode kita.

Selanjutnya kita perlu membuat method `GetAll` pada handler ProductCategory untuk menghandle requests get daftar kategori melalui HTTP requests.

```go
func (p *ProductCategory) GetAll(w http.ResponseWriter, r *http.Request) {
	ctx := r.Context()

	categories, err := p.productCategoryServices.GetAll(ctx)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		w.Write([]byte("Terjadi kesalahan pada server"))
		return
	}

	var responsePayloads []payload.GetProductCategoryResponse

	for _, category := range categories {
		responsePayloads = append(responsePayloads, payload.GetProductCategoryResponse{
			ID:   category.ID,
			Name: category.Name,
		})
	}

	response, err := json.Marshal(&responsePayloads)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		w.Write([]byte("Terjadi kesalahan pada server"))
		return
	}

	w.WriteHeader(http.StatusOK)
	w.Write(response)
}
```

Pada kode diatas kita memanggil method `GetAll` yang ada pada service ProductCategory. Seperti kita ketahui saat ini kita belum membuat method GetAll pada service ProductCategory tetapi kita sudah dapat memanggil method tersebut melalui handler yang kita buat. Ini adalah merupakan salah satu keuntungan yang kita dapat ketika menggunakan interface. Interface juga bisa digunakan sebagai prototype dari kode yang akan kita bangun, sehingga memungkinkan kita mendistribusikan pengerjaan handler dan service kepada dua programmer yang berbeda dalam waktu yang bersamaan.

Selanjutnya kita dapat membuat concrete implementation dari interface `contract.ProductCategoryService`. Buat file `product_category.go` didalam direktori `internal/service` lalu import package yang kita butuhkan untuk membuat service ProductCategory
```go
package service

import (
	"context"

	"github.com/raspiantoro/golang-marketplace/product-svc/internal/contract"
)
```

Selanjutnya buat struct ProductCategory beserta dengan constructor dari struct tersebut

```go
type ProductCategory struct {
	productCategoryRepo contract.ProductCategoryRepository
}

func NewProductCategory(productCategoryRepo contract.ProductCategoryRepository) *ProductCategory {
	return &ProductCategory{
		productCategoryRepo: productCategoryRepo,
	}
}
```

Struct ProductCategory memiliki dependency terhadap repository ProductCategory, dan lagi-lagi kita akan menggunakan teknik Dependency Injection untuk menentukan value dari field productCategoryRepo. Pada tutorial ini kita akan banyak menggunakan teknik ini kedepannya. Jika kalian lihat, setiap field yang kita buat menggunakan Dependency Injection memiliki type interface. Pada umumnya Dependency Injection memang menggunakan type interface untuk memberikan tingkat flexibilitas yang lebih tinggi, tetapi hal tersebut bukan merupakan hal yang wajib. Interface memang memiliki tingkat flexibilitas yang tinggi tetapi memiliki `trade-off` pada sisi performance[^note2]. Jika aplikasi yang kamu buat merupakan *low latency applications* seperti audio processing, high-frequency trading, dll. sebaiknya kamu memiliki pertimbangan yang matang dalam menggunakan interface. Jika part yang kamu buat tidak memerlukan flexibilitas (dapat dipastikan part tersebut hanya memiliki satu concrete implementation), sebaiknya kamu menggunakan pointer untuk menghindari penggunaan interface yang tidak diperlukan dan menambah latency pada applikasi yang kalian buat.

Service ProductCategory adalah merupakan implementation dari interface `contract.ProductCategoryService` yang memiliki method `GetAll`. Untuk mengimplementasi interface tersebut kita perlu membuat method `GetAll` pada struct `ProductCategory` dengan signature yang sama persis dengan method GetAll pada interface contract.ProductCategoryService. Tambahkan baris tersebut pada file `internal\service\product_category.go`
```go
func (p *ProductCategory) GetAll(ctx context.Context) ([]model.ProductCategory, error) {
	return p.productCategoryRepo.GetAll(ctx)
}
```

Pada kasus `GetAll` ProductCategory kita tidak memiliki bisnis proses yang terlalu kompleks, karena method GetAll hanya mengembalikan daftar kategori produk yang ada pada database. Sehingga penggunaan service pada kasus ini menjadi sangat sederhana, dan kita menyerahkan proses selanjutnya kepada layer repository yang akan menangani proses pengambilan data pada persistent storage (database). Kita tetap memerlukan penggunaan layer service pada kasus ini karena kita ingin seluruh kode yang kita buat tetap konsisten, dimana seluruh handler hanya memiliki dependency terhadap layer service untuk melakukan seluruh operasi yang berhubungan dengan bisnis proses dan persistent logic.

Masih seperti kode pada chapter sebelumnya, repository yang akan kita buat untuk GetAll product category masih hardcoded menggunakan slice untuk menyederhanakan pembahasan pada chapter ini. Untuk memulainya kita buat file `product_category.go` didalam direktori `internal/repository`. Lalu tulis kode berikut pada file product_category.go
```go
package repository

import (
	"context"
	"github.com/raspiantoro/golang-marketplace/product-svc/internal/model"
)

type ProductCategory struct{}

func NewProductCategory() *ProductCategory {
	return &ProductCategory{}
}
```

Saat ini pada struct ProductCategory kita tidak perlu membuat field apapun, karena kita belum memiliki dependency apapun untuk repository ProductCategory. Walaupun struct tersebut tidak terlalu berguna saat ini, kita tetap membutuhkannya untuk memenuhi kebutuhan pada struct ProductCategory service. Struct ProductCategory service yang sudah kita buat memiliki dependency terhadap interface `contract.ProductCategoryRepository` dan kita akan menggunakan struct ProductCategory yang ada dilayer repository sebagai concrete implementation dari interface contract.ProductCategoryRepository. Selanjutnya kita perlu menambahkan method GetAll sesuai dengan requirement yang ada pada interface contract.ProductCategoryRepository.
```go
func (p *ProductCategory) GetAll(ctx context.Context) ([]model.ProductCategory, error) {
	categories := []model.ProductCategory{
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

	return categories, nil
}
```

Pada implementasi method GetAll untuk repository ProductCategory, kita membuat slice `model.ProductCategory` dan menginisialisasi isi dari slice tersebut dan menjadikan slice tersebut sebagai return value. Untuk error return value saat ini kita kembalikan dengan nilai nil karena tidak ada error yang akan terjadi saat ini.

Selanjutnya kita perlu merefactor file main.go yang ada di-root direktori service product-svc menjadi seperti kode dibawah ini
```go
package main

import (
	"fmt"
	"net/http"

	"github.com/go-chi/chi/v5"
	"github.com/raspiantoro/golang-marketplace/product-svc/internal/handler"
	"github.com/raspiantoro/golang-marketplace/product-svc/internal/repository"
	"github.com/raspiantoro/golang-marketplace/product-svc/internal/service"
)

func main() {
	r := chi.NewRouter()

	productCategoryRepo := repository.NewProductCategory()
	productCategorySvc := service.NewProductCategory(productCategoryRepo)
	productCategoryHandler := handler.NewProductCategory(productCategorySvc)

	r.Get("/api/product-category", productCategoryHandler.GetAll)

	fmt.Println("your server is running on port 8080")
	http.ListenAndServe(":8080", r)
}
```

Pada function main, setelah membuat router chi kita melakukan inisialisasi repository, service dan handler yang dibutuhkan. Fungsi service.NewProductCategory pada package service memiliki signature seperti ini `NewProductCategory(productCategoryRepo contract.ProductCategoryRepository) *ProductCategory`. Fungsi tersebut menerima interface `contract.ProductCategoryRepository` sebagai parameter. Pada file main.go, saat kita membuat instance dari service ProductCategory kita menggunakan `productCategoryRepo` yang merupakan pointer dari type `repository.ProductCategory` sebagai inputan. Hal tersebut dianggap valid oleh compiler golang, karena struct repository.ProductCategory mengimplementasi interface contract.ProductCategoryRepository dengan menambahkan method `GetAll`. Hal yang sama juga kita lakukan saat menginisialisasi handler ProductCategory. Kita menggunakan pointer dari type `service.ProductCategory` sebagai inputan, karena struct tersebut mengimplement method `GetAll` yang ada pada interface `contract.ProductCategoryService`.

Kode yang baru kamu buat ini dapat kamu jalankan dengan cara yang sama pada chapter sebelumnya. Ketik perintah berikut untuk menjalankan service melalui root folder worspace (./golang-marketplace) kamu
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

Pada chapter ini kita telah selesai merestrukturisasi kode yang kita miliki dari chapter sebelumnya. Struktur tersebut akan kita gunakan juga pada service-service lain yang akan kita buat nantinya. Kode lengkap pada chapter ini dapat kamu lihat [disini](https://github.com/raspiantoro/golang-microservices-source-code/tree/chapter-02.02) dan untuk perubahan/penambahan file yang kita lakukan, detailnya dapat kamu lihat pada link [Pull Request](https://github.com/raspiantoro/golang-microservices-source-code/pull/3/files) ini.
  
</div>
<br/>   

---

[^note1]: penjelasan detail mengenai dependency injection bisa dilihat pada link berikut [https://en.wikipedia.org/wiki/Dependency_injection](https://en.wikipedia.org/wiki/Dependency_injection)
[^note2]: go interface performance overhead [https://syslog.ravelin.com/go-interfaces-but-at-what-cost-961e0f58a07b](https://syslog.ravelin.com/go-interfaces-but-at-what-cost-961e0f58a07b)