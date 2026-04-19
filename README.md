# CloudComputingProject
CloudComputingProject
# BSM470 - Bulut Bilişim Proje Ödevi

---

## 📋 Proje Özeti

Bu proje, Sakarya Üniversitesi Bilgisayar Mühendisliği Bölümü **BSM470 - Bulut Bilişim** dersinin tamamlayıcı ödevi kapsamında gerçekleştirilmiştir.

**Hedef:** Üç katmanlı bir bulut mimarisi kurmak ve CRUD işlemlerini gerçekleştirmek.

- **İstemci Katmanı:** Windows bilgisayar (Tarayıcı)
- **Uygulama Katmanı:** VMware üzerinde Ubuntu VM (Apache + PHP)
- **Veri Katmanı:** DigitalOcean Cloud (MariaDB)

---

## 🏗️ Sistem Mimarisi

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                   │
│  Windows İstemci (Tarayıcı)                                      │
│  └─→ b221210056.com                                              │
│      └─→ [hosts: 192.168.211.56]                                 │
│          │                                                         │
│          ├─→ VM Web/App Sunucusu (192.168.211.56)                │
│          │   ├─ OS: Ubuntu Desktop 22.04 LTS                     │
│          │   ├─ Web Server: Apache2                              │
│          │   ├─ Language: PHP 8.1                                │
│          │   ├─ User: b221@210056                                │
│          │   └─ Port: 80 (HTTP), 22 (SSH), 3306 (MySQL)         │
│          │                                                         │
│          └─→ Cloud DB Sunucusu (64.226.120.38)                   │
│              ├─ Provider: DigitalOcean                           │
│              ├─ OS: Ubuntu 22.04 LTS                             │
│              ├─ Database: MariaDB 10.11                          │
│              ├─ Hostname: root@650012122b                        │
│              ├─ Port: 3306 (MySQL) [Only VM Access]             │
│              └─ Firewall: UFW (Port 22: All, Port 3306: VM IP)  │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Öğrenci Numarası Konfigürasyonu

| Kural | Değer | Türetme |
|-------|-------|---------|
| **Öğrenci No** | `b221210056` | - |
| **WEB/App Kullanıcı** | `b221` | İlk 4 karakter |
| **WEB/App Hostname** | `210056` | Kalan 6 karakter |
| **Terminal Görünümü** | `b221@210056:~$` | kullanıcı@hostname |
| **VM IP Adresi** | `192.168.211.56` | Son 2 rakam = 56 |
| **Cloud Hostname** | `650012122b` | Öğrenci no'nun tersi (sağdan sola) |
| **Alan Adı** | `b221210056.com` | Öğrenci no + .com |

---

## 🛠️ Teknik Stack

### Backend
- **Web Server:** Apache2
- **Scripting Language:** PHP 8.1
- **Database:** MariaDB 10.11

### Infrastructure
- **Local VM:** VMware Workstation Pro (Ubuntu 22.04)
- **Cloud Platform:** DigitalOcean (Ubuntu 22.04 Droplet)
- **SSH:** OpenSSH Server
- **Firewall:** UFW (Uncomplicated Firewall)

### Tools
- **Version Control:** Git
- **Terminal:** Ubuntu Terminal, PuTTY
- **Testing:** nmap, telnet, mysql-client

---

## 📦 Kurulum Adımları

### Ön Koşullar
- VMware Workstation (veya VirtualBox)
- Windows OS (İstemci bilgisayar)
- DigitalOcean Hesabı
- SSH client (PuTTY veya WSL)

### 1️⃣ VM Kurulumu (Local)

```bash
# 1. Ubuntu Desktop VM oluştur (VMware Workstation)
# Specs: 2 CPU, 4GB RAM, 50GB Disk

# 2. VM'e bağlan ve hostname ayarla
ssh b221@192.168.211.56
sudo nano /etc/hostname
# 210056 yazıp kaydet

# 3. Statik IP yapılandır
sudo nano /etc/netplan/01-network-manager-all.yaml
# network:
#   ethernets:
#     ens33:
#       dhcp4: no
#       addresses: [192.168.211.56/24]
#       routes: [{to: default, via: 192.168.211.2}]

# 4. Paketleri kur
sudo apt update && sudo apt install -y \
  openssh-server apache2 php libapache2-mod-php php-mysql mariadb-server

# 5. Servisleri başlat
sudo systemctl enable --now apache2 ssh mariadb
```

### 2️⃣ Windows İstemci Yapılandırması

```cmd
# hosts dosyasını admin ile aç
notepad C:\Windows\System32\drivers\etc\hosts

# Aşağıdaki satırı ekle:
192.168.211.56    b221210056.com
```

### 3️⃣ Cloud DB Kurulumu (DigitalOcean)

```bash
# DigitalOcean'da Ubuntu 22.04 Droplet oluştur
# SSH bağlantı yap
ssh root@64.226.120.38

# 1. Hostname ayarla
sudo hostnamectl set-hostname 650012122b
echo "127.0.1.1 650012122b" >> /etc/hosts

# 2. MariaDB kur
sudo apt update && sudo apt install -y mariadb-server ufw

# 3. MariaDB'de DB ve USER oluştur
mysql -u root
CREATE DATABASE proje_db;
CREATE USER 'webuser'@'176.232.133.143' IDENTIFIED BY 'cloudSifre123';
GRANT ALL ON proje_db.* TO 'webuser'@'176.232.133.143';
CREATE TABLE proje_db.users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100)
);
INSERT INTO proje_db.users (name, email) VALUES ('Test User', 'test@example.com');
FLUSH PRIVILEGES;
EXIT;

# 4. bind-address değiştir
sudo sed -i 's/bind-address.*/bind-address = 0.0.0.0/' /etc/mysql/mariadb.conf.d/50-server.cnf
sudo systemctl restart mariadb

# 5. UFW Firewall ayarla
sudo ufw allow 22/tcp
sudo ufw allow from 176.232.133.143 to any port 3306
sudo ufw enable
```

### 4️⃣ PHP CRUD Uygulaması

```php
// /var/www/html/index.php
<?php
// Cloud DB bağlantısı
$servername = "64.226.120.38";
$username = "webuser";
$password = "cloudSifre123";
$dbname = "proje_db";

$conn = new mysqli($servername, $username, $password, $dbname);
if ($conn->connect_error) die("DB Hatası: " . $conn->connect_error);

// CREATE
if ($_POST['action'] == 'create') {
  $name = $_POST['name'];
  $email = $_POST['email'];
  $conn->query("INSERT INTO users (name, email) VALUES ('$name', '$email')");
}

// DELETE
if ($_POST['action'] == 'delete') {
  $id = $_POST['id'];
  $conn->query("DELETE FROM users WHERE id=$id");
}

// READ
$result = $conn->query("SELECT * FROM users");
?>

<!DOCTYPE html>
<html>
<head><title>CRUD Uygulaması</title></head>
<body>
  <h1>Kullanıcı Yönetimi</h1>
  
  <form method="POST">
    <input type="hidden" name="action" value="create">
    <input type="text" name="name" placeholder="Ad" required>
    <input type="email" name="email" placeholder="E-posta" required>
    <button type="submit">Ekle</button>
  </form>

  <h2>Kullanıcılar</h2>
  <table border="1">
    <tr><th>ID</th><th>Ad</th><th>E-posta</th><th>İşlem</th></tr>
    <?php while($row = $result->fetch_assoc()) { ?>
      <tr>
        <td><?php echo $row['id']; ?></td>
        <td><?php echo $row['name']; ?></td>
        <td><?php echo $row['email']; ?></td>
        <td>
          <form method="POST" style="display:inline;">
            <input type="hidden" name="action" value="delete">
            <input type="hidden" name="id" value="<?php echo $row['id']; ?>">
            <button type="submit">Sil</button>
          </form>
        </td>
      </tr>
    <?php } ?>
  </table>
</body>
</html>
```

---

## ✅ Test Sonuçları

### 1. Terminal Kimlik Doğrulama
```bash
$ ssh b221@192.168.211.56
b221@210056:~$ whoami; hostname
b221
210056
```
**Status:** ✅ Başarılı

### 2. Network Konfigürasyonu
```bash
b221@210056:~$ ip addr show ens33
inet 192.168.211.56/24 brd 192.168.211.255 scope global ens33
```
**Status:** ✅ Başarılı

### 3. Servis Durumu
```bash
b221@210056:~$ sudo systemctl status apache2 ssh mariadb
● apache2.service    Active: active (running)
● ssh.service        Active: active (running)
● mariadb.service    Active: active (running)
```
**Status:** ✅ Başarılı

### 4. Port Taraması
```bash
b221@210056:~$ nmap -p 22,80,3306 192.168.211.56
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
```
**Status:** ✅ Başarılı

### 5. Telnet HTTP Testi
```bash
b221@210056:~$ telnet 192.168.211.56 80
Connected to 192.168.211.56
HTTP/1.1 200 OK
```
**Status:** ✅ Başarılı

### 6. Cloud DB Bağlantısı
```bash
b221@210056:~$ telnet 64.226.120.38 3306
Connected to 64.226.120.38
```
**Status:** ✅ Başarılı

### 7. MySQL SELECT Sorgusu
```bash
b221@210056:~$ mysql -u webuser -p -h 64.226.120.38 proje_db
MariaDB [proje_db]> SELECT * FROM users;
+----+-----------+------------------+
| id | name      | email            |
+----+-----------+------------------+
|  1 | Test User | test@example.com |
+----+-----------+------------------+
```
**Status:** ✅ Başarılı

---

## 🔄 CRUD İşlemleri

### CREATE (Oluştur)
- ✅ Web arayüzünden yeni kullanıcı ekle
- ✅ Cloud DB'ye başarıyla kaydedildi
- ✅ Test: "Engin Cavak" / "engin@example.com" eklendi

### READ (Oku)
- ✅ Sayfayı açtığında mevcut kullanıcılar listeleniyor
- ✅ Cloud DB'den canlı sorgu yapılıyor
- ✅ Test User ve yeni eklenen kayıtlar görünüyor

### UPDATE (Güncelle)
- ✅ Tasarlanmıştır (future enhancement)

### DELETE (Sil)
- ✅ Her kullanıcı satırında "Sil" butonu var
- ✅ Cloud DB'den kayıt siliniyor
- ✅ Test: Kayıtlar başarıyla silindi

---

## 🔐 Güvenlik

- **Firewall:** UFW — Port 3306 sadece VM'den erişilebilir (176.232.133.143)
- **SSH:** OpenSSH Server — Tuş çifti ile bağlantı
- **DB User:** Limited privileges — Yalnızca proje_db üzerinde işlem
- **Kimliklendirme:** Öğrenci numarasına dayalı hostname ve kullanıcı adları

---

## 📸 Proje Ekran Görüntüleri

| Bölüm | Açıklama |
|-------|----------|
| Terminal | b221@210056 ve root@650012122b terminal |
| Hosts | Windows hosts dosyası yapılandırması |
| Web App | b221210056.com'a erişim ve CRUD sayfası |
| Network | IP adresleri ve network konfigürasyonu |
| Services | systemctl status çıktıları |
| Cloud DB | Cloud sunucu bağlantı testleri |

---

## 📝 Proje Dosyaları

```
.
├── README.md                          # Bu dosya
├── index.html                         # Portfolio sayfası
├── b221210056_BulutBilisim_Rapor.docx # Detaylı rapor
└── screenshots/                       # Proje ekran görüntüleri
    ├── vm-terminal.png
    ├── cloud-db.png
    ├── crud-demo.png
    └── network-test.png
```

---

## 🎓 Öğrenci Bilgileri

| Bilgi | Değer |
|-------|-------|
| **Öğrenci No** | b221210056 |
| **Adı Soyadı** | Engin Cavak |
| **Üniversite** | Sakarya Üniversitesi |
| **Fakülte** | Bilgisayar ve Bilişim Bilimleri |
| **Bölüm** | Bilgisayar Mühendisliği |
| **Ders** | BSM470 - Bulut Bilişim |
| **Teslim Tarihi** | 1 Mayıs 2026, 19:19 |
| **Sunum Tarihi** | 12 Mayıs 2026 (numara sonu çift) |

---

## 🌐 Canlı Demo

Portfolio sayfasını ziyaret et:
- **GitHub Pages:** [https://b221210056.github.io/b221210056](https://b221210056.github.io/b221210056)
- **İnteraktif CRUD Demo:** Sayfada canlı veri ekle/sil
- **Sistem Mimarisi:** Visual diyagram ve teknik özellikler

---

## 🚀 Gelecek İyileştirmeler

- [ ] UPDATE işlemi ekle (CRUD tamamlanması)
- [ ] Web arayüzü refactor (Bootstrap/Tailwind)
- [ ] API katmanı (RESTful API)
- [ ] Authentication (Login/Register)
- [ ] Database backup otomasyonu
- [ ] Monitoring ve logging
- [ ] Docker containerization

---

## 📚 Kaynaklar

- [Sakarya Üniversitesi](https://www.sakarya.edu.tr/)
- [DigitalOcean Docs](https://docs.digitalocean.com/)
- [Apache Documentation](https://httpd.apache.org/docs/)
- [MariaDB Reference Manual](https://mariadb.com/kb/en/)
- [PHP Manual](https://www.php.net/manual/)

---

## 📧 İletişim

- **E-posta:** engin.cavak@example.com
- **GitHub:** [@b221210056](https://github.com/b221210056)
- **LinkedIn:** [Engin Cavak](https://linkedin.com)

---

## 📄 Lisans

Bu proje Sakarya Üniversitesi BSM470 Bulut Bilişim dersinin ödevi kapsamında gerçekleştirilmiştir.

---

## ✨ Teşekkürler

Projede destek ve katkısı olan:
- Arş. Gör. / Öğretim Görevlileri
- Kütüphane ve Bilişim Merkezi (KBM)
- DigitalOcean (Ücretsiz Kredi)
- GitHub (Static Hosting)

---

**Status:** ✅ Proje başarıyla tamamlanmıştır.

*Son güncelleme: 19 Nisan 2026 | Version 1.0*
