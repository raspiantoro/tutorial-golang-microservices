
# Memulai Project Golang

Sebelum membuat project golang, Anda harus *men-download* dan *meng-install* Golang terlebih dahulu pada PC/laptop Anda. Bagaimana caara *meng-install* Golang sendiri dapat Anda lihat [disini](https://go.dev/doc/install). Pada saat menulis tutorial ini, Saya menggunakan Golang versi 1.22.5. Untuk memastikan seluruh code yang ada pada tutorial ini dapat berjalan dengan baik di PC/laptop Anda, pastikan Anda memiliki Golang versi 1.22.5 atau yang lebih baru pada PC/laptop Anda.

Seluruh service/komponen yang akan dibuat pada tutorial akan menggunakan pendekatan monorepo, sehingga semua source code akan berada didalam satu folder utama yang sama. Untuk memulai nya kita akan membuat satu folder utama yang bernama "golang-marketplace", untuk membuatnya Anda dapat menulis perintah tersebut pada terminal yang anda gunakan:

```bash
mkdir golang-marketplace && cd ./golang-marketplace
```

Karena kita akan membuat seluruh service dan komponen pada folder yang sama, maka kita perlu menggunakan multi-module workspaces untuk memudahkan kita dalam menjalankan code kita:

```bash
go work init .
```

Selanjutnya buat folder dengan nama **product-svc** yang akan kita gunakan untuk membuat product services yang bertanggung jawab untuk product management.

```bash
mkdir product-svc
```

Selanjutnya didalam folder product-svc kita perlu menginisialisasi golang module untuk product services. Berikut command untuk membuatnya:
```bash
cd ./product-svc && go mod init github.com/raspiantoro/golang-marketplace/product-svc
```

Buat file main.go didalam folder product-svc, dan ketikan baris berikut:
```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, Product Services!")
}
```

Selanjutnya kita perlu menambahkan product-svc kedalam workspace yang sudah kita buat

```bash
cd .. && go work use ./product-svc
```

Untuk menjalankan product-svc dari dalam folder workspace (./golang-marketplace), gunakan perintah berikut:
```bash
go run ./product-svc
```

Pada chapter ini Anda telah berhasil membuat golang workspace dan menambahkan module pada golang workspace yg telah dibuat. Code lengkap pada chapter ini dapat Anda lihat [disini](https://github.com/raspiantoro/golang-marketplace-source-code/tree/main/chapter-01)

Pada chapter berikutnya, kita akan fokus pada pembahasan pengembangan service menggunakan REST API.