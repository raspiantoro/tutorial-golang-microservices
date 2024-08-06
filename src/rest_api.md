
# REST API

<div style="text-align: justify">   

REST API atau biasa disebut juga dengan RESTful API adalah *web application programming interface (Web API)* yang biasa digunakan sebagai *communication interface* antar backend services atau antara FE/Mobile applications dengan backend services. Dari beberapa macam Web API yang ada saat ini, REST API adalah Web API yang paling umum digunakan.

REST (representational state transfer) adalah merupakan sebuah arsitektur yang memanfaatkan HTTP Requests untuk dapat mengakses dan menggunakan data yang disediakannya. Karena REST API menggunakan HTTP Requests, maka untuk mengaksesnya kita dapat menggunakan HTTP Method dan yang umum digunakan adalah GET, PUT, POST dan DELETE dimana masing-masing akan mewakili operasi read, update, create dan delete.

Pada tutorial ini kita akan menggunakan package [chi](https://github.com/go-chi/chi) untuk melakukan pengembangan REST API yang akan kita buat. Chi dipilih karena sifatnya yang berupa library (bukan framework), sehingga dapat memberikan flexibilitas lebih kepada kita dalam menyusun code dalam pengembangan service yang akan kita buat.

</div>
