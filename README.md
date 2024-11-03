
<!---
sekarwulan2609/sekarwulan2609 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Nama : Sekar Wulan Ayu Lisanti <br>
Nim : 21.01.55.0005 </br>

Deskripsi project :
Project ini merupakan project Ujian Tengah Semester web service development. Project ini berisi Web Service REST untuk sistem manajemen sesuai objek yang ditentukan menggunakan PHP dan MySQL. Web Service harus mendukung operasi CRUD. Dan dapat di uji menggunakan postman. Dalam project ini events menjadi objek Rest API, dengan struktur tabel events terdiri dari id, name, date, location, dan price.

Alat yang Dibutuhkan
1. XAMPP (atau server web lain dengan PHP dan MySQL)
2. Text editor (misalnya Visual Studio Code, Notepad++, dll)
3. Postman

1. Langkah-langkah 
    1. Persiapan Lingkungan
    2. Instal XAMPP jika belum ada.
    3. Buat folder baru bernama rest_events di dalam direktori htdocs XAMPP Anda.
2. Membuat Database 
     1. Buka phpMyAdmin
     2. Buat database baru bernama Events 
     3. Pilih database Events , lalu buka tab SQL
     4. Jalankan query SQL berikut untuk membuat tabel dan menambahkan data sampel:
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
3. Membuat File PHP untuk Web Service
   a. Buka text editor .
   b. Buat file baru dan simpan sebagai events_api.php di dalam folder rest_events.
   c. Salin dan tempel kode berikut ke dalam events_api.php:

 ```php 
<?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'],'/'));
}

function getConnection() {
    $host = 'localhost';
    $db   = 'events';
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = NULL) {
    header("HTTP/1.1 " . $status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}

$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            $id = $request[0];
            $stmt = $db->prepare("SELECT * FROM events WHERE id = ?");
            $stmt->execute([$id]);
            $events = $stmt->fetch();
            if ($events) {
                response(200, $events);
            } else {
                response(404, ["message" => "events not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM events");
            $events = $stmt->fetchAll();
            response(200, $events);
        }
        break;
    
    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->name) || !isset($data->date) || !isset($data->location) || !isset($data->price)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "INSERT INTO events (name, date, location, price) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->date, $data->location, $data->price])) {
            response(201, ["message" => "events created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create events"]);
        }
        break;
    
    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "events ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->name) || !isset($data->date) || !isset($data->location) || !isset($data->price)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "UPDATE events SET name = ?, date = ?, location = ?, price = ? WHERE id =?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->date, $data->location, $data->price, $id])) {
            response(200, ["message" => "events updated"]);
        } else {
            response(500, ["message" => "Failed to update events"]);
        }
        break;
    
    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "events ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM events WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "events deleted"]);
        } else {
            response(500, ["message" => "Failed to delete events"]);
        }
        break;
    
    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
?>
````
###

4. Pengujian dengan Postman
    1. Buka Postman
    2. Buat request baru untuk setiap operasi berikut:
    <br>Daftar Endpoint API</br>
    1. Menampilkan semua data
       <br>Method : GET</br>
       URL: http://localhost/rest_events/events_api.php
       Klik "Send"
    2. Menampilkan detail data berdasarkan id (untuk events dengan ID 3)
       <br>Method : GET</br>
       URL: http://localhost/rest_events/events_api.php/3
       Klik "Send"
    3. Menambahkan data baru
       <br>Method : POST</br>
       URL : http://localhost/rest_events/events_api.php
       <br>Headers :</br>
       1. Key: Content-Type
       2. Value: application/json
       3. Body: Pilih "raw" dan "JSON" lalu isi
          ```sql
          {
          "name": "ulang tahun",
          "date": "2024-11-19",
          "location": "Surabaya",
          "price": 5000000
          }
          ```
          ###
         Klik "Send"
    4. Mengupdate data berdasarkan ID (untuk events dengan ID 2)
       <br>Method : PUT</br>
       URL : http://localhost/rest_events/events_api.php/2
       <br>Headers :</br>
       1. Key: Content-Type
       2. Value: application/json
       3. Body:  Pilih "raw" dan "JSON" lalu isi 
        ```sql
        {
        "name": " Bazar ",
        "date": "2024-10-04",
        "location": "Jakarta",
        "price": "400000"
       }
        ```
        ###
       Klik "Send"
    5. Menghapus data berdasarkan ID (untuk events dengan ID 4)
       <br>Method : DELETE</br>
       URL : http://localhost/rest_events/events_api.php/4
       Klik "Send"



