# Tugas_UTS_Web2
```php
<?php
/**
 * ========================================================
 * LAB KEAMANAN WEB - SIMULASI SQL INJECTION (VULNERABLE)
 * Modul: Sistem Otentikasi Pengguna untuk Tugas UTS
 * ========================================================
 */

// 1. KONFIGURASI DATABASE
class Database {
    private $host = '127.0.0.1';
    private $db   = 'auth_lab'; // Pastikan nama database ini sama di phpMyAdmin
    private $user = 'root';
    private $pass = ''; 
    public $pdo;

    public function __construct() {
        $dsn = "mysql:host=$this->host;dbname=$this->db;charset=utf8mb4";
        try {
            $this->pdo = new PDO($dsn, $this->user, $this->pass, [
                PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            ]);
        } catch (\PDOException $e) {
            die("Kesalahan Sistem: Koneksi database terputus. Pastikan MySQL di XAMPP menyala dan database 'auth_lab' sudah dibuat.");
        }
    }
}

// 2. LOGIKA OTENTIKASI (SENGAJA DIBUAT RENTAN/VULNERABLE)
class Authenticator {
    private $db;
    public $debugLog = '';

    public function __construct($database) {
        $this->db = $database->pdo;
    }

    public function login($username, $password) {
        // [!] TITIK KERENTANAN: Menggabungkan input langsung ke string query
        $query = "SELECT id, username, role FROM users WHERE username = '$username' AND password = '$password' LIMIT 1";
        
        $this->debugLog = $query;

        try {
            // Mengeksekusi query tanpa filter (Sangat Berbahaya di Real App)
            $stmt = $this->db->query($query);
            return $stmt->fetch();
        } catch (PDOException $e) {
            $this->debugLog .= "\n\n[Database Error]: " . $e->getMessage();
            return false;
        }
    }
}

// 3. PROSES REQUEST
$db = new Database();
$auth = new Authenticator($db);
$pesan = null;
$userData = null;
$isPost = isset($_SERVER['REQUEST_METHOD']) && $_SERVER['REQUEST_METHOD'] === 'POST';

if ($isPost) {
    $user_input = $_POST['username'] ?? '';
    $pass_input = $_POST['password'] ?? '';
    
    $userData = $auth->login($user_input, $pass_input);
    
    if ($userData) {
        $pesan = "<div class='alert success'>Berhasil Otentikasi. Sesi dimulai sebagai: <strong>{$userData['role']}</strong></div>";
    } else {
        $pesan = "<div class='alert error'>Akses Ditolak. Kredensial tidak valid.</div>";
    }
}
?>

<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Secure Portal - Admin Login</title>
    <style>
        :root { --primary: #2563eb; --bg: #f8fafc; --text: #1e293b; }
        body { font-family: 'Segoe UI', sans-serif; background-color: var(--bg); color: var(--text); display: flex; flex-direction: column; align-items: center; padding: 40px 20px; min-height: 100vh; }
        .card { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1); width: 100%; max-width: 400px; }
        .card h2 { margin-top: 0; font-size: 1.5rem; color: #0f172a; text-align: center; margin-bottom: 24px;}
        .form-group { margin-bottom: 16px; }
        .form-group label { display: block; margin-bottom: 6px; font-weight: 500; font-size: 0.9rem; }
        .form-group input { width: 100%; padding: 12px; border: 1px solid #cbd5e1; border-radius: 6px; box-sizing: border-box; }
        .btn { background-color: var(--primary); color: white; border: none; padding: 12px; width: 100%; border-radius: 6px; font-weight: 600; cursor: pointer; transition: 0.2s; margin-top: 10px; }
        .btn:hover { background-color: #1d4ed8; }
        .alert { padding: 12px; border-radius: 6px; margin-bottom: 20px; font-size: 0.9rem; text-align: center; }
        .alert.success { background-color: #dcfce7; color: #166534; border: 1px solid #bbf7d0; }
        .alert.error { background-color: #fee2e2; color: #991b1b; border: 1px solid #fecaca; }
        .debug-console { margin-top: 30px; background: #0f172a; color: #10b981; padding: 20px; border-radius: 8px; width: 100%; max-width: 700px; font-family: 'Courier New', monospace; font-size: 0.85rem; overflow-x: auto; border-left: 5px solid #10b981; }
        .debug-header { color: #94a3b8; font-weight: bold; margin-bottom: 10px; border-bottom: 1px solid #334155; padding-bottom: 5px; text-transform: uppercase; letter-spacing: 1px; }
    </style>
</head>
<body>

    <div class="card">
        <h2>Portal Manajemen</h2>
        
        <?php if($pesan) echo $pesan; ?>

        <form method="POST" action="">
            <div class="form-group">
                <label>Username Identitas</label>
                <input type="text" name="username" placeholder="admin / user" required autocomplete="off">
            </div>
            <div class="form-group">
                <label>Kata Sandi</label>
                <input type="password" name="password" placeholder="••••••••">
            </div>
            <button type="submit" class="btn">Otorisasi Masuk</button>
        </form>
    </div>

    <?php if ($isPost): ?>
    <div class="debug-console">
        <div class="debug-header">▶ SQL Tracer / Developer Console</div>
        <p><strong>[SQL QUERY]:</strong><br> <span style="color: #fcd34d;"><?php echo nl2br(htmlspecialchars($auth->debugLog)); ?></span></p>
        <?php if($userData): ?>
            <hr style="border: 0; border-top: 1px solid #334155; margin: 15px 0;">
            <p style="color: #38bdf8;"><strong>[OBJECT RETURNED]:</strong><br> 
            <pre><?php print_r($userData); ?></pre></p>
        <?php endif; ?>
    </div>
    <?php endif; ?>

</body>
</html>

```


## Hasil
<img width="1808" height="1355" alt="Screenshot 2026-04-28 095718" src="https://github.com/user-attachments/assets/cab2dc55-d8e1-4e61-ae46-28da547ed60e" />
<img width="1761" height="1219" alt="Screenshot 2026-04-28 095648" src="https://github.com/user-attachments/assets/074ea3d5-51d8-492e-ba4a-dc9c880fe2be" />
<img width="1571" height="1389" alt="Screenshot 2026-04-28 093327" src="https://github.com/user-attachments/assets/fe4c1126-48c7-48a2-a042-5dcdaaf367a7" />
