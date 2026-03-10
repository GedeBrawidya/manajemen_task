# Panduan Relasi Database Proyek Ini

Dokumen ini menjelaskan struktur relasi database yang ada di aplikasi ini, mencakup relasi antara `User`, `Project`, dan `Task`. Ini dirancang khusus agar kamu bisa belajar bagaimana *Eloquent ORM* di Laravel bekerja dengan tabel migration yang sudah kamu buat.

## 1. Relasi `User` dan `Project` (Many-to-Many)

Dalam aplikasi ini, skenarionya adalah:
- Satu *User* bisa terlibat dalam **banyak** *Project*.
- Satu *Project* bisa memiliki **banyak** *User* (anggota tim).

Karena kedua belah pihak bisa memiliki "banyak", ini dinamakan **Many-to-Many Relationship**.

### Bagaimana Migration-nya Bekerja?
Untuk menghubungkan dua tabel dalam skenario Many-to-Many, database **membutuhkan tabel pembantu (tabel ketiga)** yang disebut **Pivot Table**.
Kabar baiknya, kamu **SUDAH** membuat file migration untuk ini, yaitu:
`database/migrations/0001_01_01_000003_create_project_user_table.php`

Tabel pivot `project_user` ini berisi `project_id` dan `user_id`. Tugasnya sederhana: Mencetak sejarah "user mana saja yang terdaftar di project mana". Aturan standar Laravel adalah nama tabel pivot ini diambil dari kombinasi nama dari masing-masing tabel model yang dikaitkan, dan dibuat secara *singular* serta berurutan abjad (`project` lalu `_` lulu `user`).

### Implementasi di Model
Kamu sudah menuliskan ini di kode model kamu dengan sangat baik menggunakan `belongsToMany`:

**Di Model `Project` (`app/Models/Project.php`):**
```php
public function users()
{
    // Sebuah project melibatkan banyak user (melalui tabel pivot `project_user`)
    return $this->belongsToMany(User::class);
}
```

**Di Model `User` (`app/Models/User.php`):**
```php
public function assigned_projects()
{
    // Seorang user ditugaskan di banyak project (melalui tabel pivot `project_user`)
    return $this->belongsToMany(Project::class);
}
```

---

## 2. Relasi `Project` dan `Task` (One-to-Many)

Skenarionya:
- Di dalam sebuah *Project*, terdapat **banyak** *Task* (tugas-tugas yang harus diselesaikan).
- Tapi sebuah *Task* hanya bisa terikat dan berada di dalam **satu** *Project* saja.

Ini dinamakan **One-to-Many Relationship**.

### Bagaimana Migration-nya Bekerja?
Karena ini One-to-Many, kita **tidak** butuh tabel pivot. Sebagai gantinya, tabel yang berada di sisi "Banyak/Many" (yaitu tabel `tasks`) harus menyimpan ID dari tabel di sisi "Satu/One" (yaitu `projects`).

Oleh karena itu, di file migration `0001_01_01_000003_create_tasks_table.php`, kamu menambahkan kolom:
```php
$table->foreignId('project_id')->constrained()->onDelete('cascade');
```
Langkahmu ini sudah tepat sekali!

### Implementasi di Model

**Di Model `Project` (`app/Models/Project.php`):**
```php
public function tasks()
{
    // Satu project memiliki banyak task
    return $this->hasMany(Task::class);
}
```

**Di Model `Task` (`app/Models/Task.php`):**
```php
public function project()
{
    // Satu task adalah milik satu (merujuk ke) project
    return $this->belongsTo(Project::class);
}
```

---

## 3. Relasi `User` dan `Task` (One-to-Many)

Skenarionya:
- Sebuah *Task* dikerjakan (di-assign) ke **satu** orang *User* spesifik (sebagai penanggung jawab).
- Seorang *User* bisa dikasih/mengerjakan **banyak** *Task* di saat yang bersaman.

Ini juga merupakan **One-to-Many Relationship**.

### Bagaimana Migration-nya Bekerja?
Sama seperti relasi nomor 2 di atas, di tabel `tasks`, kamu menambahkan kolom merujuk ke `users`:
```php
$table->foreignId('assigned_to')->constrained('users')->onDelete('cascade');
```

**Catatan Khusus:** Karena kamu menggunakan nama kolom khusus yaitu `assigned_to` (bukan nama fungsi bawaan standar Laravel, yaitu `user_id`), kamu harus mendefinisikannya secara eksplisit di dalam Model agar Laravel tidak kebingungan.

### Implementasi di Model
**Di Model `User` (`app/Models/User.php`):**
```php
public function tasks()
{
    // User punya banyak task.
    // Kita beritahu Laravel kalau nama field foreign key-nya adalah 'assigned_to'
    return $this->hasMany(Task::class, 'assigned_to');
}
```

**Di Model `Task` (`app/Models/Task.php`):**
```php
public function assignee()
{
    // Task merujuk ke satu penerima tugas (User).
    // Kita panggil nama relasinya "assignee" untuk merujuk model User dengan foreign key 'assigned_to'
    return $this->belongsTo(User::class, 'assigned_to');
}
```

---

## Kesimpulan

Sistem database kamu secara arsitektur sudah **kokoh** karena kamu mengingat ada yang namanya tabel *Pivot* (`project_user`) dan peletakan *Foreign Key* (*constrained id*) yang akurat antar tabel untuk menghindari data berantakan (menggunakan fungsi cascade). File modelnya (terutama `Task`) juga sudah dilengkapi. Lanjutkan kerja bagusnya! 🚀💪
