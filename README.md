# Dokumentasi Panel Admin Posting Blogger

File ini berisi panduan lengkap mulai dari konfigurasi Firebase, pemasangan **Alat Posting Blogger (Panel Admin)**, hingga pengaturan keamanan database.

---

## 1. Persiapan Firebase Project

Sebelum memasang kode, Anda wajib mengatur project di Firebase Console agar fitur Login dan Database berfungsi.

### **A. Aktifkan Authentication**
1. Buka [Firebase Console](https://console.firebase.google.com/).
2. Masuk ke project Anda.
3. Di menu sebelah kiri, pilih **Build** > **Authentication**.
4. Klik **Get Started**.
5. Pilih tab **Sign-in method**.
6. Klik **Google**, lalu aktifkan tombol **Enable**.
7. Masukkan nama project dan email support, lalu **Save**.

### **B. Aktifkan Firestore Database**
1. Di menu sebelah kiri, pilih **Build** > **Firestore Database**.
2. Klik **Create Database**.
3. Pilih lokasi server (pilih yang terdekat, misal: *asia-southeast2* untuk Jakarta).
4. Klik **Start in Test Mode** (nanti kita ganti Rules-nya) atau **Production Mode**.
5. Klik **Create**.

---

## 2. Persiapan Kode (`panel_admin.html`)

Buka file HTML/JS utama (`panel_admin.html`) di text editor Anda dan lakukan penyesuaian berikut:

### **A. Konfigurasi Firebase**
Cari bagian `const firebaseConfig = { ... }` dan ganti isinya dengan konfigurasi project Anda.
*   *Cara mendapatkan config:* Di Firebase Console > Project Overview > Klik ikon Web (</>) > Register App > Copy bagian `firebaseConfig`.

### **B. Konfigurasi Blog ID**
Cari bagian `const BLOG_ID = "..."`.
*   Ganti isinya dengan **ID Blog** Anda (bisa dilihat di URL saat membuka dashboard Blogger, angka panjang setelah `blogID=`).

> **Catatan:** Editor teks menggunakan **QuillJS** yang sudah terintegrasi secara otomatis (tanpa API Key), jadi tidak perlu konfigurasi tambahan untuk editor.

---

## 3. Cara Pemasangan di Blogger

Panel Admin ini akan dipasang pada **Halaman Statis (Page)** rahasia di Blogger.

### **Langkah-langkah:**

1. Masuk ke Dashboard Blogger.
2. Buka menu **Halaman (Pages)**.
3. Klik **Halaman Baru (New Page)**.
4. Beri judul halaman (misalnya: *Admin Panel* atau nama acak agar sulit ditebak orang lain).
5. Ubah mode editor dari *Tampilan Menulis* ke **Tampilan HTML (HTML View)**.
6. Hapus semua kode bawaan yang ada di editor.
7. **Copy seluruh isi file** `panel_admin.html` yang sudah Anda edit tadi, lalu **Paste** ke editor Blogger.
8. Pada menu **Options/Pilihan** (sebelah kanan):
   * Pilih **Jangan izinkan, sembunyikan yang ada** pada bagian Komentar pembaca.
9. Klik **Publikasikan (Publish)**.

Panel Admin Anda sekarang sudah aktif.

---

## 4. Konfigurasi Keamanan Firestore (Rules)

Agar database aman dan hanya bisa diakses oleh pengguna yang login, ganti aturan database Anda.

1. Buka Firebase Console > **Firestore Database**.
2. Masuk ke tab **Rules**.
3. Hapus semua kode yang ada, lalu tempelkan kode di bawah ini:

```js
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
Klik Publish.

