# 🛒 Aplikasi Toko Kita - CRUD Produk dengan REST API

## 👤 Informasi Praktikan
- **Nama**: Naufal Aulia Pratama
- **NIM**: H1D023036
- **Shift Awal / Baru**: E / A


## 📱 Deskripsi
Aplikasi **Toko Kita** adalah aplikasi mobile e-commerce sederhana yang dibuat menggunakan **Flutter** dan **CodeIgniter 4** sebagai backend REST API. Aplikasi ini mendemonstrasikan implementasi lengkap operasi CRUD (Create, Read, Update, Delete) produk dengan sistem autentikasi berbasis token.

Pengguna dapat mendaftar, login, melihat daftar produk, menambah produk baru, mengubah data produk, dan menghapus produk. Semua data disimpan di database MySQL dan diakses melalui REST API yang aman.

## 🏗️ Arsitektur Aplikasi

### 1. Struktur Projek
```
tokokita/
├── lib/                           # Flutter Frontend
│   ├── main.dart                  # Entry point aplikasi
│   ├── helpers/
│   │   ├── api_url.dart          # Konfigurasi endpoint API
│   │   └── user_info.dart        # Manajemen token & user data
│   ├── model/
│   │   ├── produk.dart           # Model Produk
│   │   ├── login.dart            # Model Response Login
│   │   └── registrasi.dart       # Model Response Registrasi
│   └── ui/
│       ├── login_page.dart       # Halaman Login
│       ├── registrasi_page.dart  # Halaman Registrasi
│       ├── produk_page.dart      # Halaman List Produk
│       ├── produk_form.dart      # Halaman Form Tambah/Edit
│       └── produk_detail.dart    # Halaman Detail Produk
│
└── toko-api/                      # CodeIgniter 4 Backend
    ├── app/
    │   ├── Controllers/
    │   │   ├── LoginController.php
    │   │   ├── RegistrasiController.php
    │   │   └── ProdukController.php
    │   └── Models/
    │       ├── MMember.php
    │       ├── MLogin.php
    │       └── MProduk.php
    └── public/
        └── index.php             # Entry point API
```

### 2. State Management
Aplikasi ini menggunakan **StatefulWidget** untuk manajemen state pada setiap halaman. Data produk diambil dari API menggunakan package `http` dan disimpan dalam state lokal. Ketika terjadi perubahan data (tambah, edit, hapus), aplikasi akan memanggil fungsi `getData()` untuk memperbarui tampilan.

#### Contoh State Management di ProdukPage
```dart
class _ProdukPageState extends State<ProdukPage> {
  List<Produk> listProduk = [];

  @override
  void initState() {
    super.initState();
    getData();  // Load data saat halaman dibuka
  }

  Future<void> getData() async {
    try {
      http.Response response = await http.get(
        Uri.parse(ApiUrl.listProduk)
      );
      final data = json.decode(response.body);
      
      if (data['code'] == 200) {
        setState(() {
          listProduk = (data['data'] as List)
            .map((json) => Produk.fromJson(json))
            .toList();
        });
      }
    } catch (e) {
      // Error handling
    }
  }
}
```

### 3. API Integration
Komunikasi dengan backend menggunakan HTTP requests:

#### GET - Mengambil Data
```dart
http.get(Uri.parse(ApiUrl.listProduk))
```

#### POST - Menambah Data
```dart
http.post(
  Uri.parse(ApiUrl.createProduk),
  headers: {"Content-Type": "application/json"},
  body: json.encode(data),
)
```

#### PUT - Mengubah Data
```dart
http.put(
  Uri.parse(ApiUrl.updateProduk(id)),
  headers: {"Content-Type": "application/json"},
  body: json.encode(data),
)
```

#### DELETE - Menghapus Data
```dart
http.delete(Uri.parse(ApiUrl.deleteProduk(id)))
```

### 4. Autentikasi & Token Management
Setelah login berhasil, API mengirimkan token yang disimpan menggunakan `SharedPreferences`:

```dart
// Simpan token
UserInfo().setToken(login.token!);
UserInfo().setUserID(login.userID.toString());
UserInfo().setEmail(login.userEmail!);

// Ambil token
String? token = await UserInfo().getToken();

// Logout (hapus semua data)
await UserInfo().logout();
```

### 5. UI Components
Aplikasi menggunakan komponen Material Design standar Flutter dengan widget custom untuk konsistensi tampilan.

#### Widget ItemProduk
```dart
class ItemProduk extends StatelessWidget {
  final Produk produk;
  final VoidCallback? onUpdate;

  const ItemProduk({
    super.key, 
    required this.produk, 
    this.onUpdate
  });

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () async {
        await Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => ProdukDetail(produk: produk),
          ),
        );
        if (onUpdate != null) onUpdate!();
      },
      child: Card(
        child: ListTile(
          title: Text(produk.namaProduk!),
          subtitle: Text("Rp. ${produk.hargaProduk}"),
        ),
      ),
    );
  }
}
```

## ✨ Fitur Aplikasi

### 1. Registrasi Page
<img src="screenshots/ss%20registrasi%20tokokita.jpg" width="300" alt="Registrasi Page">

Pengguna dapat mendaftar dengan mengisi:
- **Nama** (minimal 3 karakter)
- **Email** (format email valid)
- **Password** (minimal 6 karakter)
- **Konfirmasi Password** (harus sama dengan password)

Setelah registrasi berhasil, data disimpan ke database dengan password ter-hash menggunakan `password_hash()` di backend.

### 2. Login Page
<img src="screenshots/ss%20login%20tokokita.jpg" width="300" alt="Login Page">

Pengguna login menggunakan email dan password. Backend akan:
- Validasi kredensial
- Generate token unik (100 karakter random)
- Simpan token ke tabel `login`
- Return token dan data user ke Flutter

Token disimpan di `SharedPreferences` untuk digunakan pada request selanjutnya.

### 3. List Produk Page
<img src="screenshots/ss%20list%20produk%20tokokita.jpg" width="300" alt="List Produk Page">

Menampilkan semua produk dari database dalam bentuk list card. Fitur:
- **Auto-load data** saat halaman dibuka
- **Loading indicator** saat fetch data
- **Pull to refresh** setelah operasi CRUD
- **Floating Action Button (+)** untuk tambah produk
- **Drawer menu** dengan opsi logout

### 4. Tambah Produk
<img src="screenshots/ss tambah produk tokokita.jpg" width="300" alt="Tambah Produk Page">

Form untuk menambah produk baru dengan field:
- **Kode Produk** (wajib diisi)
- **Nama Produk** (wajib diisi)
- **Harga** (wajib diisi, hanya angka)

Data dikirim ke API endpoint `POST /produk` dan otomatis muncul di list setelah berhasil disimpan.

### 5. Edit Produk
<img src="screenshots/ss%20ubah%20produk%20tokokita.jpg" width="300" alt="Edit Produk Page">

Form yang sama dengan tambah produk, namun:
- Field ter-isi data produk yang akan diedit
- Tombol berubah menjadi "Ubah"
- Data dikirim ke `PUT /produk/{id}`
- List di-refresh setelah update berhasil

### 6. Detail Produk
<img src="screenshots/ss%20detail%20produk%20tokokita.jpg" width="300" alt="Detail Produk Page">

Menampilkan informasi lengkap produk:
- Kode Produk
- Nama Produk
- Harga (format Rupiah)

Dengan aksi:
- **Tombol EDIT**: Membuka form edit dengan data pre-filled
- **Tombol DELETE**: Menampilkan dialog konfirmasi, lalu menghapus produk

### 7. Logout
Menu logout di drawer akan:
- Hapus semua data dari `SharedPreferences` (token, userID, email)
- Redirect ke halaman login
- Clear navigation stack (tidak bisa back)
