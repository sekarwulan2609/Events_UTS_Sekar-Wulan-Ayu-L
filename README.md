
<!---
sekarwulan2609/sekarwulan2609 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Nama : Sekar Wulan Ayu Lisanti <br>
Nim : 21.01.55.0005 </br>

Deskripsi project :
Project ini merupakan project Ujian Tengah Semester web service development. Project ini berisi Web Service REST untuk sistem manajemen sesuai objek yang ditentukan menggunakan PHP dan MySQL. Web Service harus mendukung operasi CRUD. Dan dapat di uji menggunakan postman. Dalam project ini events menjadi objek Rest API, dengan struktur tabel events terdiri dari id, name, date, location, dan price.
 ```sql
CREATE TABLE events (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    date DATE NOT NULL,
    location VARCHAR (255) NOT NULL,
    price INT NOT NULL
);
INSERT INTO events (name, date, location, price) VALUES
('wedding', '2024-10-15, 'Semarang', '80000000'),
('Workshop', '2024-10-20', 'Semarang', '2000000'),
('Talkshow', '2024-11-28', 'Jakarta', '3000000'),
('Pameran', '2024-11-18', 'Surabaya', '8000000'),
('Seminar & Konferensi', '2024-11-26', 'Surabaya', '5000000'),
(Konser, '2024-12-15', 'Jakarta', '180000000'),
('wedding', '2024-12-28', 'Semarang', '80000000');
```
###

Daftar Endpoint API
1. Menampilkan semua data
   <br>Method : GET</br>
   URL: http://localhost/rest_events/events_api.php
2. Menampilkan detail data berdasarkan id (untuk events dengan ID 3)
   <br>Method : GET</br>
   URL: http://localhost/rest_events/events_api.php/3
4. Menambahkan data baru
   <br>Method : POST</br>
   URL : http://localhost/rest_events/events_api.php
   <br>Headers :</br>
   1. Key: Content-Type
   2. Value: application/json
   Body:
   1. Pilih "raw" dan "JSON"
5. Mengupdate data berdasarkan ID (untuk events dengan ID 2)
   <br>Method : PUT</br>
   URL : http://localhost/rest_events/events_api.php/2
   <br>Headers :</br>
   1. Key: Content-Type
   2. Value: application/json
   Body:
   1. Pilih "raw" dan "JSON"
7. Menghapus data berdasarkan ID (untuk events dengan ID 4)
   <br>Method : DELETE</br>
   URL : http://localhost/rest_events/events_api.php/4

Cara instalasi dan penggunaan 


