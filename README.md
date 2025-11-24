# lab9_php_modular

# *Nama : Muhammad Ridho Hafiedz*
# *Nim  : 312410195*
# *Kelas: TI.24.A2*

```
project/
├── index.php              # Front Controller
├── .htaccess              # URL Rewriting
├── config/
│   └── database.php       # Konfigurasi DB
├── views/                 # Template Tampilan
│   ├── header.php
│   ├── footer.php
│   └── dashboard.php
├── modules/               # Logika Aplikasi
│   ├── user/
│   │   ├── list.php
│   │   └── add.php
│   └── auth/
│       ├── login.php
│       └── logout.php
└── assets/               # Resource Statis
    ├── css/
    └── js/
```

# istem Routing yang Diimplementasi

**1. Front Controller Pattern** 

File `index.php` berfungsi sebagai single entry point untuk seluruh aplikasi:
```php
// Konfigurasi dasar
define('BASE_URL', 'http://localhost/project');
$page = $_GET['page'] ?? 'dashboard';

// Mapping halaman yang valid
$pages = [
    'dashboard' => 'views/dashboard.php',
    'user/list' => 'modules/user/list.php',
    // ... lainnya
];

// Load halaman berdasarkan parameter
if (isset($pages[$page]) && file_exists($pages[$page])) {
    require_once $pages[$page];
} else {
    // Fallback ke halaman error
}
```

**2. URL Routing Mechanism**

Sebelum routing:
```
http://localhost/project/index.php?page=user/list
```

Setelah routing dengan .htaccess:
```
http://localhost/project/user/list
```

**3. .htaccess Configuration**
```apache
RewriteEngine On

# Redirect semua request ke index.php
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^([a-zA-Z0-9\/_-]+)$ index.php?page=$1 [QSA,L]
```

Penjelasan rules:
- `RewriteCond %{REQUEST_FILENAME} !-f` - Skip jika file benar-benar ada
- `RewriteCond %{REQUEST_FILENAME} !-d` - Skip jika direktori benar-benar ada
- `RewriteRule ^([a-zA-Z0-9\/_-]+)$` - Tangkap semua karakter valid URL
- `index.php?page=$1` - Arahkan ke front controller dengan parameter

# Sistem Template & Layout

**Header Template (views/header.php)**
```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Sistem Modular - Praktikum 9</title>
    <style>
        /* Global styles */
        .container { width: 80%; margin: 0 auto; }
        nav a { color: white; padding: 0.5rem 1rem; }
        .content { padding: 2rem; min-height: 400px; }
    </style>
</head>
<body>
    <div class="container">
        <header><h1>Sistem Modular dengan PHP</h1></header>
        <nav>
            <a href="<?php echo BASE_URL; ?>/dashboard">Dashboard</a>
            <a href="<?php echo BASE_URL; ?>/user/list">Daftar User</a>
            <!-- Navigation links -->
        </nav>
```

**Footer Template (views/footer.php)**
```php
        <footer>
            <p>&copy; <?php echo date('Y'); ?>, Informatika, Universitas Pelita Bangsa</p>
        </footer>
    </div>
</body>
</html>
```

**Penggunaan Template di Halaman**
```php
<?php require('header.php'); ?>
<div class="content">
    <h2>Ini Halaman Content</h2>
    <p>Konten spesifik halaman</p>
</div>
<?php require('footer.php'); ?>
```

# Modul modul yang diterapkan

**1. Modul User Management**

***user/list.php*** - Menampilkan daftar pengguna:
```php
<div class="content">
    <h2>Daftar Pengguna</h2>
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Nama</th>
                <th>Email</th>
                <th>Role</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            <!-- Data pengguna -->
            <tr>
                <td>1</td>
                <td>Agung Nugroho</td>
                <td>agung@pelitabangsa.ac.id</td>
                <td>Admin</td>
                <td>
                    <button>Edit</button>
                    <button>Hapus</button>
                </td>
            </tr>
        </tbody>
    </table>
</div>
```

***user/add.php*** - Form tambah pengguna baru:
```php
<div class="content">
    <h2>Tambah Pengguna Baru</h2>
    <form method="POST" action="">
        <div class="form-group">
            <label for="nama">Nama Lengkap:</label>
            <input type="text" id="nama" name="nama" required>
        </div>
        <!-- Field lainnya -->
        <button type="submit">Simpan Pengguna</button>
    </form>
</div>
```

**2. Modul Authentication**

***auth/login.php*** - Form login sistem:
```php
<div class="content">
    <h2>Login</h2>
    <form method="POST" action="">
        <div class="form-group">
            <label for="username">Username:</label>
            <input type="text" id="username" name="username" required>
        </div>
        <div class="form-group">
            <label for="password">Password:</label>
            <input type="password" id="password" name="password" required>
        </div>
        <button type="submit">Login</button>
    </form>
</div>
```

# Konfigurasi Database
***config/database.php*** - Menggunakan PDO untuk koneksi database:
```php
class Database {
    private $host = "localhost";
    private $db_name = "praktikum8";
    private $username = "root";
    private $password = "";
    public $conn;
    
    public function getConnection() {
        $this->conn = null;
        try {
            $this->conn = new PDO("mysql:host=" . $this->host . ";dbname=" . $this->db_name, 
                                $this->username, $this->password);
            $this->conn->exec("set names utf8");
        } catch(PDOException $exception) {
            echo "Connection error: " . $exception->getMessage();
        }
        return $this->conn;
    }
}
```

<img width="1920" height="1080" alt="Screenshot 2025-11-24 184839" src="https://github.com/user-attachments/assets/86bbea1f-dbe2-4953-bd56-96bf4cf1e7e5" />

# Flow Request-Response:
1. **User** mengakses URL: `http://localhost/project/user/list`
2. **.htaccess** menangkap request dan mengarahkan ke `index.php?page=user/list`
3. **Front Controller** (`index.php`) memproses parameter `page`
4. **Router** mencari mapping yang sesuai untuk *user/list*
5. **System** memuat:
  - `views/header.php` (template header)
  - `modules/user/list.php` (konten spesifik)
  - `views/footer.php` (template footer)
  -
6. **Output** dikembalikan ke browser sebagai halaman HTML lengkap
