# Laporan Proyek Belajar Machine Learning Terapan - Faizal Riza

## Project Overview

Dalam era digital saat ini, jumlah buku yang diterbitkan dan tersedia bagi pembaca meningkat secara eksponensial. Fenomena ini sering menyebabkan masalah *information overload*, di mana pengguna merasa kesulitan untuk menemukan buku yang sesuai dengan minat mereka di antara ribuan pilihan yang tersedia. Tanpa sistem penyaringan yang efektif, pengguna mungkin menghabiskan waktu terlalu lama untuk mencari atau bahkan kehilangan minat membaca.

Sistem rekomendasi hadir sebagai solusi untuk mengatasi masalah ini dengan memprediksi preferensi pengguna dan menyarankan item yang paling relevan. Dalam konteks perpustakaan digital atau toko buku *online*, sistem rekomendasi dapat meningkatkan pengalaman pengguna dengan menyajikan judul-judul yang memiliki karakteristik serupa dengan buku yang telah mereka baca atau sukai sebelumnya.

Proyek ini bertujuan untuk membangun sistem rekomendasi buku menggunakan pendekatan *Content-Based Filtering*. Pendekatan ini dipilih karena mampu memberikan rekomendasi berdasarkan fitur spesifik dari buku (dalam hal ini kategori atau genre), sehingga pengguna bisa mendapatkan saran buku yang memiliki kemiripan tema dengan buku favorit mereka.

**Referensi:**

* Pazzani, M. J., & Billsus, D. (2007). Content-based recommendation systems. In *The adaptive web* (pp. 325-341). Springer, Berlin, Heidelberg.
* Schroder, G., Thiele, M., & Lehner, W. (2011). Setting goals and choosing metrics for recommender system evaluations.

## Business Understanding

Proyek ini dikembangkan untuk menyediakan fitur rekomendasi pada platform buku, memudahkan pengguna menemukan buku baru yang relevan berdasarkan buku yang pernah mereka lihat atau sukai.

### Problem Statements

Berdasarkan latar belakang di atas, rumusan masalah yang diajukan adalah:

1. Bagaimana cara melakukan pra-pemrosesan data buku agar fitur-fitur penting seperti kategori dapat digunakan untuk sistem rekomendasi?
2. Bagaimana cara membangun sistem rekomendasi yang dapat menyarankan buku-buku yang mirip berdasarkan kesamaan kategori (*genre*) menggunakan pendekatan *Content-Based Filtering*?

### Goals

Tujuan dari proyek ini adalah:

1. Menghasilkan dataset yang bersih dan terstruktur dengan mengekstraksi fitur kategori yang relevan dari metadata buku.


2. Membuat model sistem rekomendasi *Content-Based Filtering* yang mampu menghasilkan *top-N recommendation* buku dengan tingkat relevansi (presisi) yang tinggi.

### Solution statements

Solusi yang diajukan untuk menyelesaikan masalah ini adalah menggunakan pendekatan **Content-Based Filtering**.

* **Content-Based Filtering**: Pendekatan ini merekomendasikan item yang mirip dengan item yang disukai pengguna di masa lalu. Kesamaan dihitung berdasarkan fitur konten item.
* Algoritma yang digunakan adalah **TF-IDF Vectorizer** untuk merepresentasikan fitur teks (Kategori) menjadi vektor numerik, dan **Cosine Similarity** untuk menghitung derajat kesamaan antar judul buku berdasarkan vektor tersebut.



## Data Understanding

Dataset yang digunakan dalam proyek ini adalah **Book-Crossing Dataset** yang diunduh dari Kaggle. Dataset ini berisi informasi mengenai pengguna, buku, dan penilaian (rating).

Sumber dataset: [Kaggle - Book Crossing Dataset](https://www.kaggle.com/datasets/ruchi798/bookcrossing-dataset).

**Informasi Data:**

* Jumlah entri awal data buku: 1.031.175 entri.


* Data terdiri dari 19 kolom sebelum pembersihan.


* Terdapat 278.858 user dan 271.379 buku dalam dataset asli.



**Variabel-variabel pada Book-Crossing dataset yang digunakan:**

* 
`user_id`: Identitas unik pengguna (Anonymized).


* 
`book_title`: Judul buku.


* 
`book_author`: Penulis buku.


* 
`publisher`: Penerbit buku.


* 
`Category`: Kategori atau genre buku (Fitur utama untuk pemodelan).


* 
`rating`: Penilaian buku oleh pengguna (skala 1-10).



**Exploratory Data Analysis (EDA):**

* **Distribusi Rating**: Data rating didominasi oleh nilai 0 (implisit). Rating eksplisit berkisar antara 1 hingga 10, dengan rating 8 dan 10 cukup dominan.


* **Distribusi Kategori**: Terdapat ribuan kategori unik. Kategori "Fiction" adalah yang paling banyak muncul, diikuti oleh "Juvenile Fiction" dan "Biography & Autobiography".
* 
**Format Kategori**: Ditemukan bahwa penulisan kategori pada data mentah tidak bersih (contoh: `['Social Science']`), sehingga memerlukan pembersihan teks.



## Data Preparation

Berikut adalah tahapan *data preparation* yang dilakukan untuk mempersiapkan data sebelum pemodelan:

1. **Handling Missing Values (`dropna`)**:
* Menghapus baris data yang memiliki nilai kosong (*NaN*) untuk memastikan integritas data saat pemrosesan.




2. **Feature Selection (Drop Columns)**:
* Menghapus kolom yang tidak relevan untuk rekomendasi berbasis konten kategori, yaitu: `Unnamed: 0`, `location`, `isbn`, `img_s`, `img_m`, `img_l`, `city`, `age`, `state`, `Language`, `country`, `year_of_publication`, dan `Summary` .


* *Alasan*: Mengurangi dimensi data dan fokus pada fitur konten (Kategori) serta metadata utama (Judul, Penulis).


3. **Data Cleaning & Filtering**:
* Menghapus baris dengan kategori '9' (data kotor) dan rating 0 (fokus pada buku yang pernah diberi rating).


* Membersihkan teks pada kolom `Category` menggunakan *Regular Expression* (Regex) untuk menghapus karakter simbol seperti kurung siku dan tanda petik, sehingga teks menjadi bersih (contoh: `['Fiction']` menjadi `Fiction`).




4. **Category Filtering (Sampling)**:
* Mengambil sampel data yang mengandung kategori populer tertentu (index 5 s.d 20 dari kategori terbanyak) seperti *Religion*, *Cooking*, *Travel*, dll.
* 
*Alasan*: Untuk efisiensi komputasi dan memastikan kategori yang dimodelkan memiliki jumlah data yang cukup.




5. **Deduplikasi Data**:
* Menghapus data duplikat berdasarkan `book_title`.
* *Alasan*: Sistem rekomendasi berbasis konten memerlukan daftar item unik sebagai basis data (katalog) untuk matriks kesamaan. Duplikasi akan menyebabkan bias pada perhitungan matriks.


* Data akhir yang digunakan berjumlah **13.048 baris**.





## Modeling

Pada tahap ini, model sistem rekomendasi dibangun menggunakan pendekatan **Content-Based Filtering**. Model ini bekerja dengan mempelajari profil item berdasarkan fitur teks (Kategori).

**Tahapan Pemodelan:**

1. **TF-IDF Vectorizer**:
* Digunakan fungsi `TfidfVectorizer` dari library `sklearn`.


* Proses ini mengubah data teks pada kolom `Category` menjadi representasi vektor matriks.
* Fungsi ini menghitung bobot setiap kata (genre) terhadap keseluruhan dokumen (buku). Hasilnya adalah matriks berukuran (13.048, 15), yang merepresentasikan 13.048 buku dan 15 kategori unik.




2. **Cosine Similarity**:
* Digunakan fungsi `cosine_similarity` dari library `sklearn`.


* Menghitung derajat kesamaan (similarity) antar buku dengan mengukur sudut kosinus antara dua vektor dalam matriks TF-IDF.
* Outputnya adalah matriks kesamaan berukuran (13.048, 13.048) di mana nilai 1.0 menunjukkan kesamaan sempurna (buku yang sama).




3. **Mendapatkan Rekomendasi (Top-N)**:
* Dibuat fungsi `book_recommendation` yang menerima parameter `nama_buku` dan `k` (jumlah rekomendasi).


* Fungsi bekerja dengan mencari indeks buku yang diminta, menghitung skor kesamaan dengan buku lain, mengurutkannya dari yang terbesar, dan mengembalikan `k` buku teratas.



**Contoh Hasil Rekomendasi:**
Sistem diminta merekomendasikan buku yang mirip dengan **"McDonald's: Behind the Arches"** (Kategori: *Business_Economics*).

**Output Top-10 Recommendation:**
| Judul Buku | Kategori |
| :--- | :--- |
| FUZZY LOGIC: THE REVOLUTIONARY COMPUTER... | Business_Economics |
| The Motley Fool Investment Guide... | Business_Economics |
| Chicago's Museums: A Complete Guide... | Business_Economics |
| PASSION PROFIT POWER | Business_Economics |
| If You Want to Be Rich & Happy... | Business_Economics |
| Startup: A Silicon Valley Adventure | Business_Economics |
| Staying Alive: Women, Ecology and Development | Business_Economics |
| Leadership and the One Minute Manager... | Business_Economics |
| The Job Hunter's Catalog | Business_Economics |
| The Financially Confident Woman | Business_Economics |
.

## Evaluation

Metrik evaluasi yang digunakan untuk mengukur kinerja model Content-Based Filtering ini adalah **Precision** (Presisi).

### Formula Metrik

Presisi dalam konteks sistem rekomendasi dihitung dengan rumus:

### Hasil Evaluasi

Berdasarkan hasil uji coba pada tahap *Modeling*:

* **Buku Referensi**: "McDonald's: Behind the Arches"
* **Kategori Referensi**: *Business_Economics*
* **Hasil Rekomendasi**: Sistem memberikan 10 rekomendasi buku.
* **Relevansi**: Dari 10 buku yang direkomendasikan, seluruhnya (10 buku) memiliki kategori yang sama dengan buku referensi, yaitu *Business_Economics*.

Maka perhitungan presisinya adalah:


**Kesimpulan:**
Sistem rekomendasi berhasil mengidentifikasi dan menyarankan buku dengan kategori yang sama persis dengan buku yang dicari pengguna. Nilai presisi 100% pada sampel uji ini menunjukkan bahwa representasi fitur kategori menggunakan TF-IDF dan perhitungan jarak menggunakan Cosine Similarity bekerja dengan sangat baik untuk mengelompokkan buku berdasarkan genrenya.