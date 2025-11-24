# lab9_php_modular

# Nama : Muhammad Ridho Hafiedz
# Nim  : 312410195
# Kelas: TI.24.A2**


**1. index.php (Front Controller)**
File ini akan menangani routing dan memuat halaman yang sesuai.
``` php
<?php
// index.php

// Start session (jika diperlukan)
session_start();

// Konfigurasi dasar
define('BASE_URL', 'http://localhost/project'); // Ganti dengan base URL project Anda

// Ambil parameter page dari URL, default ke 'dashboard'
$page = isset($_GET['page']) ? $_GET['page'] : 'dashboard';

// Daftar halaman yang valid
$allowed_pages = [
    'dashboard',
    'user/list',
    'user/add',
    'auth/login',
    'auth/logout'
];

// Cek jika halaman yang diminta valid
if (!in_array($page, $allowed_pages)) {
    $page = 'dashboard'; // Default ke dashboard jika halaman tidak valid
}

// Include header
require_once 'views/header.php';

// Load halaman yang diminta
switch ($page) {
    case 'dashboard':
        require_once 'views/dashboard.php';
        break;
    case 'user/list':
        require_once 'modules/user/list.php';
        break;
    case 'user/add':
        require_once 'modules/user/add.php';
        break;
    case 'auth/login':
        require_once 'modules/auth/login.php';
        break;
    case 'auth/logout':
        require_once 'modules/auth/logout.php';
        break;
    default:
        require_once 'views/dashboard.php';
        break;
}

// Include footer
require_once 'views/footer.php';
?>
```

**2. .htaccess (URL Rewriting)**
File ini akan mengubah URL dari `index.php?page=user/list` menjadi `user/list`.
```apache
RewriteEngine On

# Jika file atau direktori tidak ada, arahkan ke index.php
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php?page=$1 [L,QSA]
```

**3. config/database.php**
File ini berisi koneksi database dari praktikum 8.
```php
<?php
// config/database.php

class Database {
    private $host = "localhost";
    private $db_name = "praktikum8";
    private $username = "root";
    private $password = "";
    public $conn;

    public function getConnection() {
        $this->conn = null;
        try {
            $this->conn = new PDO("mysql:host=" . $this->host . ";dbname=" . $this->db_name, $this->username, $this->password);
            $this->conn->exec("set names utf8");
        } catch(PDOException $exception) {
            echo "Connection error: " . $exception->getMessage();
        }
        return $this->conn;
    }
}
?>
```

4. views/header.php
File ini berisi bagian kepala dari HTML dan navigasi.
```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Modular PHP</title>
    <link href="<?php echo BASE_URL; ?>/assets/css/style.css" rel="stylesheet">
</head>
<body>
    <div class="container">
        <header>
            <h1>Modularisasi Menggunakan Require</h1>
        </header>
        <nav>
            <a href="<?php echo BASE_URL; ?>/dashboard">Home</a>
            <a href="<?php echo BASE_URL; ?>/user/list">Daftar User</a>
            <a href="<?php echo BASE_URL; ?>/user/add">Tambah User</a>
            <a href="<?php echo BASE_URL; ?>/auth/login">Login</a>
            <a href="<?php echo BASE_URL; ?>/auth/logout">Logout</a>
        </nav>
```

**5. views/footer.php**
File ini berisi bagian penutup dari HTML.
```php
        <footer>
            <p>&copy; 2021, Informatika, Universitas Pelita Bangsa</p>
        </footer>
    </div>
</body>
</html
```

**6. views/dashboard.php**
File ini berisi konten halaman dashboard.
```php
<div class="content">
    <h2>Ini Halaman Home</h2>
    <p>Ini adalah bagian content dari halaman.</p>
</div>
```

**7. modules/user/list.php**
File ini berisi kode untuk menampilkan daftar user. Kita akan menggunakan koneksi database dari praktikum 8.
```php
<?php
// modules/user/list.php

// Include file database
require_once '../../config/database.php';

// Buat koneksi database
$database = new Database();
$db = $database->getConnection();

// Query untuk mengambil data user
$query = "SELECT * FROM users";
$stmt = $db->prepare($query);
$stmt->execute();

$users = $stmt->fetchAll(PDO::FETCH_ASSOC);
?>

<div class="content">
    <h2>Daftar User</h2>
    <table border="1">
        <tr>
            <th>ID</th>
            <th>Nama</th>
            <th>Email</th>
            <th>Aksi</th>
        </tr>
        <?php foreach ($users as $user): ?>
        <tr>
            <td><?php echo $user['id']; ?></td>
            <td><?php echo $user['nama']; ?></td>
            <td><?php echo $user['email']; ?></td>
            <td>
                <a href="edit.php?id=<?php echo $user['id']; ?>">Edit</a>
                <a href="delete.php?id=<?php echo $user['id']; ?>">Hapus</a>
            </td>
        </tr>
        <?php endforeach; ?>
    </table>
</div>
```

**8. modules/user/add.php**
File ini berisi form untuk menambah user.
```php
<?php
// modules/user/add.php

// Include file database
require_once '../../config/database.php';

// Jika form disubmit
if ($_POST) {
    $database = new Database();
    $db = $database->getConnection();

    $nama = $_POST['nama'];
    $email = $_POST['email'];

    $query = "INSERT INTO users (nama, email) VALUES (?, ?)";
    $stmt = $db->prepare($query);
    $stmt->bindParam(1, $nama);
    $stmt->bindParam(2, $email);

    if ($stmt->execute()) {
        echo "User berhasil ditambahkan.";
    } else {
        echo "Gagal menambahkan user.";
    }
}
?>

<div class="content">
    <h2>Tambah User</h2>
    <form method="post">
        <label for="nama">Nama:</label>
        <input type="text" name="nama" required>
        <br>
        <label for="email">Email:</label>
        <input type="email" name="email" required>
        <br>
        <button type="submit">Simpan</button>
    </form>
</div>
```

**9. modules/auth/login.php**
File ini berisi form login.
```php
<div class="content">
    <h2>Login</h2>
    <form method="post">
        <label for="username">Username:</label>
        <input type="text" name="username" required>
        <br>
        <label for="password">Password:</label>
        <input type="password" name="password" required>
        <br>
        <button type="submit">Login</button>
    </form>
</div>
```

**10. modules/auth/logout.php**
File ini berisi proses logout.
```php
<?php
// modules/auth/logout.php

// Contoh: menghapus session dan redirect ke halaman login
session_destroy();
header('Location: ' . BASE_URL . '/auth/login');
exit;
```

**11. assets/css/style.css**
Kita bisa menambahkan style dasar di sini.
```css
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
}

.container {
    width: 80%;
    margin: 0 auto;
}

header {
    background: #f0f0f0;
    padding: 20px;
}

nav {
    background: #333;
    color: white;
    padding: 10px;
}

nav a {
    color: white;
    text-decoration: none;
    padding: 10px;
}

nav a:hover {
    background: #555;
}

.content {
    padding: 20px;
}

footer {
    background: #f0f0f0;
    padding: 10px;
    text-align: center;
}

table {
    width: 100%;
    border-collapse: collapse;
}

table, th, td {
    border: 1px solid #ddd;
}

th, td {
    padding: 8px;
    text-align: left;
}

th {
    background-color: #f2f2f2;
}
```

<img width="1920" height="1080" alt="Screenshot 2025-11-24 184839" src="https://github.com/user-attachments/assets/02fb4b92-efb8-422a-9478-1ab722b24948" />
