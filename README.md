# TugasUTS
1. Persiapan Lingkungan
- Instal XAMPP jika belum ada.
- Buat folder baru bernama rest_sepatu di dalam direktori htdocs XAMPP Anda.
2. Membuat Database
- Buka phpMyAdmin (http://localhost/phpmyadmin)
- Buat database baru bernama bookstore
- Pilih database bookstore, lalu buka tab SQL
- Jalankan query SQL berikut untuk membuat tabel dan menambahkan data sampel:

CREATE TABLE `sepatu` (
  `id` int(11) NOT NULL,
  `name` varchar(255) NOT NULL,
  `type` varchar(255) NOT NULL,
  `price` varchar(100) NOT NULL,
  `stock` int(11) NOT NULL
);
3. Membuat File PHP untuk Web Service
- Buka visual studio code Anda.
- Buat file baru dan simpan sebagai sepatu_api.php di dalam folder rest_sepatu.
- Salin dan tempel kode berikut ke dalam sepatu_api.php:

<?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, typeization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'],'/'));
}

function getConnection() {
    $host = 'localhost';
    $db   = 'sepatustore';
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
            $stmt = $db->prepare("SELECT * FROM sepatu WHERE id = ?");
            $stmt->execute([$id]);
            $sepatu = $stmt->fetch();
            if ($sepatu) {
                response(200, $sepatu);
            } else {
                response(404, ["message" => "Sepatu not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM sepatu");
            $sepatu = $stmt->fetchAll();
            response(200, $sepatu);
        }
        break;
    
    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->name) || !isset($data->type) || !isset($data->price)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "INSERT INTO sepatu (name, type, price) VALUES (?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->type, $data->price])) {
            response(201, ["message" => "Sepatu created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create sepatu"]);
        }
        break;
    
    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "Sepatu ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->name) || !isset($data->type) || !isset($data->price)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "UPDATE sepatu SET name = ?, type = ?, price = ? WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->type, $data->price, $id])) {
            response(200, ["message" => "sepatu updated"]);
        } else {
            response(500, ["message" => "Failed to update sepatu"]);
        }
        break;
    
    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "sepatu ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM sepatu WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "sepatu deleted"]);
        } else {
            response(500, ["message" => "Failed to delete sepatu"]);
        }
        break;
    
    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
?>

4. Pengujian dengan Postman
- Buka Postman
- Buat request baru untuk setiap operasi berikut:

a. GET All Books
- Method: GET
- URL: http://localhost/rest_sepatu/sepatu_api.php
- Klik "Send"

b. GET Specific Book
- Method: GET
- URL: http://localhost/rest_sepatu/sepatu_api.php/1 (untuk buku dengan ID 1)
- Klik "Send"

c. POST New Book
- Method: POST
- URL: http://localhost/rest_sepatu/sepatu_api.php
Headers:
- Key: Content-Type
- Value: application/json
Body:
- Pilih "raw" dan "JSON"
- Masukkan:
{
    "name": "GO RUN Supersonic Max Womens Sneakers",
    "type": "Natural",
    "price": "Rp. 1.199.000"
}

- Klik "Send"

d. PUT (Update) Book
- Method: PUT
- URL: http://localhost/rest_sepatu/sepatu_api.php/7 (asumsikan ID buku baru adalah 7)
Headers:
- Key: Content-Type
- Value: application/json
Body:
- Pilih "raw" dan "JSON"
- Masukkan:
{
    "name": "GO RUN Trail Altitude Women Sneakers",
    "type": "Black",
    "price": "Rp. 1.099.000"
}

- Klik "Send"

e. DELETE Book
- Method: DELETE
- URL: http://localhost/rest_sepatu/sepatu_api.php/9 (untuk menghapus buku dengan ID 9)
- Klik "Send"
