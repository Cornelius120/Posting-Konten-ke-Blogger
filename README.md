# Dokumentasi Panel Admin Posting Blogger

File ini berisi panduan instalasi untuk **Alat Posting Blogger (Panel Admin)** dan konfigurasi keamanan **Firestore Database**.

---

## 1. Cara Pemasangan di Blogger

Kode HTML/JS yang telah dibuat (`panel_admin.html`) dirancang untuk dipasang pada sebuah **Halaman Statis (Page)** di Blogger, bukan posting biasa. Ini berfungsi sebagai **Panel Admin** rahasia.

### **Langkah-langkah Instalasi**

#### **1. Siapkan Kode**

* Buka file HTML yang sudah dibersihkan: `panel_admin.html`.
* Lakukan penyesuaian pada bagian-bagian berikut:

  **A. Konfigurasi Firebase & Blog ID**
  * Ganti `const firebaseConfig = { ... }` dengan konfigurasi asli Firebase Project Anda.
  * Ganti `const BLOG_ID = "..."` dengan ID blog Anda.

  **B. Konfigurasi TinyMCE (Editor)**
  * Cari tag script untuk TinyMCE dan ganti `LINK_TINYMCE` dengan URL CDN yang menyertakan API Key Anda (Dapatkan gratis di [tiny.cloud](https://www.tiny.cloud/)).
  * Formatnya harus seperti ini:
    ```
    <!-- Ganti dengan cdn dari tinymce yang sudah ada api keynya (penting agar tinymce bisa di load) -->
    <script src="LINK_TINYMCE" referrerpolicy="origin" crossorigin="anonymous"></script>
    ```

#### **2. Buat Halaman Baru**

1. Masuk ke Dashboard Blogger.
2. Buka menu **Halaman (Pages)**.
3. Klik **Halaman Baru (New Page)**.
4. Beri judul, misalnya *Admin Panel* atau nama lain yang tidak mudah ditebak.

#### **3. Masukkan Kode HTML**

1. Ubah mode editor ke **Tampilan HTML (HTML View)**.
2. Hapus semua isi yang ada.
3. Copy seluruh isi file `panel_admin.html` yang sudah diedit tadi dan paste ke editor.

#### **4. Pengaturan Halaman**

* Pada bagian **Options/Pilihan**, set:
  * **Komentar:** *Jangan izinkan, sembunyikan yang ada*.
* Klik **Publikasikan (Publish)**.

#### **5. Akses Panel Admin**

Buka halaman tersebut. Panel admin siap digunakan.

---

## 2. Konfigurasi Firestore Database (Rules)

Salin kode berikut dan tempelkan di tab **Rules** pada Firestore Database di Firebase Console.

### **Rules Firestore**

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    match /users/{userId} {
      allow read: if true;
      allow create: if request.auth != null && request.auth.uid == userId;
      allow update: if request.auth != null && request.auth.uid == userId;
      allow delete: if false;
    }

    match /messages/{messageId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow update: if request.auth != null;
      allow delete: if request.auth != null && (
        resource.data.senderId == request.auth.uid ||
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin' ||
        (get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'moderator' && resource.data.senderRole != 'admin')
      );
    }
  }
}
```
download
content_copy
expand_less
