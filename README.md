
<!---
sekarwulan2609/sekarwulan2609 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Nama : Sekar Wulan Ayu Lisanti <br>
Nim : 21.01.55.0005 </br>

Deskripsi project :
Project ini merupakan project Ujian Tengah Semester web service development. Project ini berisi Web Service REST untuk sistem manajemen sesuai objek yang ditentukan menggunakan PHP dan MySQL. Web Service harus mendukung operasi CRUD. Dan dapat di uji menggunakan postman. Dalam project ini events menjadi objek Rest API, dengan struktur tabel events terdiri dari id, name, date, location, dan price.

### Tujuan
Membuat dan menguji web service REST untuk manajemen events menggunakan PHP dan MySQL.

### Alat yang Dibutuhkan
1. XAMPP (atau server web lain dengan PHP dan MySQL)
2. Text editor (misalnya Visual Studio Code, Notepad++, dll)
3. Postman

### 1. Langkah-langkah :
   1. Persiapan Lingkungan
   2. Instal XAMPP jika belum ada.
   3. Buat folder baru bernama `rest_events` di dalam direktori `htdocs` XAMPP Anda.
### 2. Membuat Database :
   1. Buka phpMyAdmin (http://localhost/phpmyadmin) 
   2. Buat database baru bernama `events`
   3. Pilih database `events` , lalu buka tab SQL
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
### 3. Membuat File PHP untuk Web Service
   <br>a. Buka text editor .</br>
   <br>b. Buat file baru dan simpan sebagai `events_api.php` di dalam folder `rest_events`.</br>
   <br>c. Salin dan tempel kode berikut ke dalam `events_api.php`:</br>

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
function validateevents($name, $date, $location, $price) {
    $errors = [];
    if (empty($name)) {
        $errors[] = "name is required";
    }
    if (empty($date)) {
        $errors[] = "date is required";
    }
    if (empty($location)) {
        $errors[] = "location is required";
    }
    if (empty($price)) {
        $errors[] = "price is required";
    }
    return $errors;
}
$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            if ($request[0] === 'search') {
                // 5.1 Search functionality
                $searchTerm = $_GET['term'] ?? '';
                $stmt = $db->prepare("SELECT * FROM events WHERE name LIKE ? OR location LIKE ?");
                $searchTerm = "%$searchTerm%";
                $stmt->execute([$searchTerm, $searchTerm]);
                $events = $stmt->fetchAll();
                response(200, $events);
            } else {
                // Get specific events
                $id = $request[0];
                $stmt = $db->prepare("SELECT * FROM events WHERE id = ?");
                $stmt->execute([$id]);
                $events = $stmt->fetch();
                if ($events) {
                    response(200, $events);
                } else {
                    response(404, ["message" => "events not found"]);
                }
            }
        } else {
            // 5.2 Pagination
            $page = isset($_GET['page']) ? (int)$_GET['page'] : 1;
            $limit = isset($_GET['limit']) ? (int)$_GET['limit'] : 10;
            $offset = ($page - 1) * $limit;

            $stmt = $db->prepare("SELECT * FROM events LIMIT ? OFFSET ?");
            $stmt->bindValue(1, $limit, PDO::PARAM_INT);
            $stmt->bindValue(2, $offset, PDO::PARAM_INT);
            $stmt->execute();
            $events = $stmt->fetchAll();

            $totalStmt = $db->query("SELECT COUNT(*) FROM events");
            $total = $totalStmt->fetchColumn();

            response(200, [
                'events' => $events,
                'total' => $total,
                'page' => $page,
                'limit' => $limit
            ]);
        }
        break;
    
    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        $errors = validateevents($data->name ?? '', $data->date ?? '', $data->location ?? '', $data->price ?? '' );
        if (!empty($errors)) {
            response(400, ["errors" => $errors]);
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
        $errors = validateevents($data->name ?? '', $data->date ?? '', $data->location ?? '', $data->price ?? '');
        if (!empty($errors)) {
            response(400, ["errors" => $errors]);
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

### 4. Pengujian dengan Postman
1. Buka Postman
2. Buat request baru untuk setiap operasi berikut:
<br>Daftar Endpoint API
#### 1. Menampilkan semua data
   Method : GET
   <br>URL: `http://localhost/rest_events/events_api.php`
   <br>Klik "Send"
#### 2. Menampilkan detail data berdasarkan id (untuk events dengan ID 3)
   Method : GET
   <br>URL: `http://localhost/rest_events/events_api.php/3`
   <br>Klik "Send"
#### 3. Menambahkan data baru
   Method : POST
   <br>URL : `http://localhost/rest_events/events_api.php`
  <br> Headers :
   1. Key: Content-Type
   2. Value: application/json
   3. Body: Pilih "raw" dan "JSON" lalu isi
       
          
          {
          "name": "ulang tahun",
          "date": "2024-11-19",
          "location": "Surabaya",
          "price": 5000000
          }
          
4. Klik "Send"
### 4. Mengupdate data berdasarkan ID (untuk events dengan ID 2)
   Method : PUT
   <br>URL : `http://localhost/rest_events/events_api.php/2`
   <br>Headers :
   1. Key: Content-Type
   2. Value: application/json</br>
   3. Body:  Pilih "raw" dan "JSON" lalu isi
      
            {
            "name": " Bazar ",
            "date": "2024-10-04",
            "location": "Jakarta",
            "price": "400000"
           }
            ````
   4. Klik "Send"

### 5. Menghapus data berdasarkan ID (untuk events dengan ID 4)
   Method : DELETE
   <br>URL : `http://localhost/rest_events/events_api.php/4`
   <br>Klik "Send"
### Kesimpulan

Dalam praktikum ini, telah berhasil membuat web service REST untuk events menggunakan PHP dan MySQL. Dan dapat menguji API menggunakan Postman. Praktik ini memberikan dasar yang kuat untuk pengembangan API RESTful lebih lanjut.


