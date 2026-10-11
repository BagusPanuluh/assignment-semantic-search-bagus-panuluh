# Semantic Search & LLM — Asisten FAQ E-commerce

Prototype asisten customer service e-commerce yang mencari FAQ berdasarkan makna pertanyaan, lalu menghasilkan jawaban menggunakan Gemini. Seluruh kode berada dalam satu file `notebook/starter_notebook_semantic_search.ipynb`, dengan delapan section mengikuti starter notebook dan assignment.

## 1. Problem Statement dan Objective

Customer service menerima banyak pertanyaan berulang tentang pemesanan, pengiriman, pembayaran, dan pengembalian barang. Pelanggan sering memakai kata atau frasa berbeda dari pertanyaan dalam FAQ, sehingga pencarian berdasarkan kata kunci saja dapat melewatkan informasi yang sesuai.

Tujuan proyek ini adalah:

- Mengubah setiap FAQ menjadi embedding menggunakan Gemini Embeddings API.
- Menyimpan embedding dalam FAISS dan mengambil FAQ yang paling relevan terhadap pertanyaan pengguna.
- Menyusun prompt dengan Persona, Task, Context, dan Format.
- Menggunakan Gemini untuk menjawab berdasarkan FAQ hasil retrieval.
- Menguji query langsung, parafrase, dan pertanyaan yang tidak memiliki jawaban dalam knowledge base.

Sistem merupakan prototype awal sebelum pengembangan RAG yang lebih lengkap. Sistem hanya menggunakan FAQ yang disediakan, tanpa akses ke akun atau transaksi pelanggan.

## 2. Knowledge Base

Sumber data adalah `data/ecommerce_faq_knowledge_base.csv` yang disertakan dalam assignment. Dataset berisi **59 FAQ dalam 8 kategori**.

Kolom yang digunakan:

- `faq_id`: identitas unik setiap FAQ.
- `category`: kategori pertanyaan.
- `question`: pertanyaan FAQ.
- `answer`: jawaban atau kebijakan yang tersedia.

Jumlah entri per kategori:

- Pemesanan: 9.
- Pengiriman: 9.
- Pembayaran: 8.
- Pengembalian: 8.
- Akun: 7.
- Produk: 7.
- Promo: 6.
- Customer Service: 5.

Satu FAQ menjadi satu unit retrievable dengan format:

```text
Q: {question} A: {answer}
```

Kolom `content` dibuat di DataFrame dalam memori. CSV asli tidak ditulis ulang. Chunking dokumen panjang tidak digunakan karena setiap entri FAQ sudah menjadi satu unit pencarian.

## 3. Struktur Proyek

```text
semantic-search-assignment/
├── data/
│   └── ecommerce_faq_knowledge_base.csv
├── notebook/
│   └── starter_notebook_semantic_search.ipynb
├── artifacts/
│   ├── faq.index
│   └── embeddings.npy
├── documentation/
│   └── retrieval_test_results.md
├── requirements.txt
├── README.md
└── .gitignore
```

File `.env` digunakan secara lokal untuk API key dan konfigurasi model. File tersebut tidak disertakan dalam repository. Folder `.venv` merupakan environment lokal dan juga tidak perlu diunggah. Tidak ada file `.py` aplikasi terpisah.

## 4. Alur Sistem

**Ingest → Embed → Store → Retrieve → Generate**

1. **Ingest:** Pandas memuat CSV dan menampilkan jumlah FAQ, kolom, serta distribusi kategori. Pertanyaan dan jawaban digabung menjadi `content`.
2. **Embed:** `get_embedding()` memanggil Gemini Embeddings API. Embedding diperiksa dimensi dan nilainya, lalu dinormalisasi. Dokumen memakai task `RETRIEVAL_DOCUMENT`, sedangkan query memakai `RETRIEVAL_QUERY`.
3. **Store:** Embedding dimasukkan ke FAISS `IndexFlatL2`. Jumlah vektor diperiksa agar sama dengan jumlah FAQ. Index dan embedding disimpan di folder `artifacts/`.
4. **Retrieve:** `semantic_search(query, top_k=3)` mengubah query menjadi embedding dan mengambil FAQ terdekat. Hasil memuat ID, kategori, pertanyaan, jawaban, distance, dan similarity.
5. **Generate:** `build_prompt()` menyisipkan FAQ hasil retrieval ke Context. `generate_answer()` mengirim prompt ke Gemini dan mengembalikan jawaban serta kandidat FAQ yang digunakan sebagai context.

Delapan section notebook tetap mengikuti urutan: import/load data, unit retrievable, embedding, FAISS, semantic search, prompt/context, integrasi LLM, dan pengujian relevansi.

### Model dan Skor

Konfigurasi default dalam notebook:

- Embedding: `gemini-embedding-001`.
- LLM: `gemini-3.8-flash`.
- Dimensi embedding: 768.
- Jumlah kandidat retrieval: 3.
- Temperature jawaban: 0.3.

FAISS menghasilkan **squared L2 distance**. Karena vektor telah dinormalisasi, notebook menghitung cosine similarity dengan:

```text
similarity = 1 - distance / 2
```

Distance lebih rendah menunjukkan vektor lebih dekat; similarity lebih tinggi menunjukkan kemiripan makna lebih besar. Skor ini bukan probabilitas jawaban benar.

### Desain Prompt

- **Persona:** asisten customer service e-commerce yang ramah.
- **Task:** menjawab hanya berdasarkan FAQ, tanpa menambahkan fakta, angka, atau kebijakan yang tidak tersedia.
- **Context:** pertanyaan dan jawaban dari FAQ hasil semantic search.
- **Format:** jawaban singkat dalam bahasa Indonesia, mempertahankan syarat dan batas waktu, serta menyebut nomor FAQ pendukung bila jawaban tersedia.

Jika context tidak menyediakan jawaban, prompt mengarahkan Gemini untuk mengakui belum memiliki informasi dan menyarankan customer service. Kandidat yang ditemukan FAISS belum tentu menjawab query.

## 5. Cara Menjalankan

### Menyiapkan Virtual Environment

Gunakan Python 3.11 atau lebih baru. Buka PowerShell di root proyek. Jika berada di lokasi proyek saat ini:

```powershell
cd "D:\AI Engineering\semantic-search-assignment"
```

Jika `.venv` belum tersedia, buat terlebih dahulu:

```powershell
python -m venv .venv
```

Aktifkan environment dan instal dependensi:

```powershell
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Jika aktivasi diblokir oleh PowerShell, gunakan interpreter environment secara langsung tanpa aktivasi:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Dependensi utama meliputi Pandas, NumPy, FAISS CPU, Google Gen AI SDK, python-dotenv, dan Jupyter. `nbclient` tersedia untuk pemeriksaan eksekusi notebook.

### Menyiapkan API Key

Buat file `.env` di root proyek jika belum ada. Jika sudah ada, edit file tersebut tanpa menimpanya. Isi konfigurasi berikut dengan API key milik Anda pada file lokal:

```dotenv
GEMINI_API_KEY=isi_api_key_anda
GEMINI_EMBEDDING_MODEL=gemini-embedding-001
GEMINI_LLM_MODEL=gemini-3.8-flash
EMBEDDING_DIMENSION=768
```

Jangan menaruh API key dalam notebook, README, atau repository. Gemini memerlukan koneksi internet, akses model, dan kuota API.

### Membuka Notebook

Perintah berikut dapat dipakai tanpa mengaktifkan environment:

```powershell
.\.venv\Scripts\python.exe -m ipykernel install --prefix .venv --name python3 --display-name "Semantic Search (.venv)"
.\.venv\Scripts\jupyter.exe notebook notebook/starter_notebook_semantic_search.ipynb
```

Pilih kernel **Semantic Search (.venv)**, kemudian **Restart Kernel and Run All**. Notebook dapat dijalankan dari root proyek atau folder `notebook/`.

Urutan eksekusi penting karena fungsi retrieval dan LLM membutuhkan variabel, embedding, serta index dari section sebelumnya. Tanpa API key, bagian yang memerlukan Gemini dilewati; hasil baru tidak dibuat. Output lama yang tersimpan dalam notebook bukan bukti keberhasilan eksekusi baru tanpa key.

Run All dengan key membuat ulang embedding, index, dan laporan pengujian, sehingga menggunakan kuota API dan menimpa hasil sebelumnya.

### Mencoba Pertanyaan Sendiri

Setelah section 1–7 berhasil dijalankan, gunakan cell baru:

```python
# Retrieval saja.
results = semantic_search("Paket saya terlambat, harus bagaimana?", top_k=3)
display(pd.DataFrame(results))

# Retrieval dan jawaban Gemini.
answer, sources = generate_answer("Apakah bisa membayar dengan cicilan?")
print(answer)
display(pd.DataFrame(sources))

# Chat interaktif; ketik keluar untuk berhenti.
chat()
```

`chat()` tidak dipanggil otomatis, sehingga Run All tidak berhenti menunggu input. Variabel `sources` berisi kandidat context; tidak semua kandidat otomatis merupakan sumber yang mendukung setiap klaim jawaban.

Untuk keluar dari virtual environment yang sudah diaktifkan:

```powershell
deactivate
```

## 6. Hasil Pengujian dan Analisis Relevansi

Bagian ini menggunakan hasil yang sudah tersimpan dalam notebook dan [laporan pengujian](documentation/retrieval_test_results.md). Pengujian API tidak dijalankan ulang saat penulisan README.

Embedding tersimpan memiliki shape **(59, 768)**. FAISS index berisi **59 vektor dengan dimensi 768**, sesuai jumlah entri knowledge base.

Pengujian mencakup dua query langsung, dua parafrase, dan satu query di luar FAQ.

### Query 1 — Langsung: Pelacakan Paket

**Query:** “Bagaimana cara melacak paket yang sedang dikirim?”

- FAQ acuan: 10.
- Kandidat berurutan: 10, 52, 12.
- Distance peringkat pertama: 0.3984; similarity: 0.8008.
- Jawaban menjelaskan nomor resi di halaman Pesanan Saya setelah barang dikirim dan pelacakan melalui website kurir.

**Analisis:** FAQ yang tepat berada di peringkat pertama. Jawaban sesuai FAQ 10 dan tidak mengaku sudah melacak paket pengguna. FAQ 52 dan 12 berkaitan dengan pengiriman, tetapi membahas kasus lain; keberadaannya tidak mengganggu jawaban pada sampel ini.

### Query 2 — Langsung: Waktu Refund

**Query:** “Berapa lama proses refund setelah pengembalian disetujui?”

- FAQ acuan: 25.
- Kandidat berurutan: 25, 27, 41.
- Distance peringkat pertama: 0.3739; similarity: 0.8131.
- Jawaban menyebut 3–7 hari kerja setelah barang yang dikembalikan diterima dan diverifikasi penjual.

**Analisis:** Retrieval tepat dan jawaban mempertahankan syarat awal perhitungan waktu. Waktu refund tidak digeser menjadi sejak pengajuan atau persetujuan. FAQ 27 dan 41 membahas pengembalian dana/cashback dengan konteks berbeda, tetapi jawaban tetap mengikuti FAQ acuan.

### Query 3 — Parafrase: Paket Terlambat

**Query:** “Paketku telat lebih dari tiga hari dari jadwal, harus lapor ke siapa?”

- FAQ acuan: 12.
- Kandidat berurutan: 12, 14, 16.
- Distance peringkat pertama: 0.3957; similarity: 0.8021.
- Jawaban menyarankan menghubungi customer service dengan nomor resi bila keterlambatan melebihi estimasi lebih dari 3 hari.

**Analisis:** Sistem menghubungkan kata “telat” dan “jadwal” dengan FAQ tentang keterlambatan melewati estimasi. Jawaban mempertahankan batas lebih dari 3 hari serta kebutuhan nomor resi. Ini mendukung kemampuan pencarian makna pada parafrase yang diuji. FAQ 14/16 tetap berupa kandidat terkait, tetapi bukan jawaban utama untuk query ini.

### Query 4 — Parafrase: Cicilan

**Query:** “Bisa nggak belanja dibayar nyicil dengan kartu kredit?”

- FAQ acuan: 20.
- Kandidat berurutan: 20, 18, 22.
- Distance peringkat pertama: 0.3965; similarity: 0.8017.
- Jawaban menyatakan cicilan tersedia untuk kartu kredit tertentu pada nominal minimal yang ditentukan bank.

**Analisis:** Kata “nyicil” berhasil dipetakan ke FAQ tentang cicilan. Jawaban mempertahankan syarat kartu kredit dan nominal minimal sesuai bank, tanpa mengarang nominal, tenor, atau bunga. FAQ 18/22 membahas metode pembayaran lain, tetapi tidak menjadi dasar jawaban pada sampel ini.

### Query 5 — Di Luar FAQ: Harga Saham

**Query:** “Berapa harga saham perusahaan ini hari ini?”

- Tidak ada FAQ acuan karena knowledge base tidak memuat informasi harga saham.
- Kandidat berurutan: 43, 58, 40.
- Distance peringkat pertama: 0.7108; similarity: 0.6446.
- Gemini mengakui belum memiliki informasi tersebut dalam FAQ dan menyarankan customer service.

**Analisis:** Kandidat terdekat tidak menjawab pertanyaan saham. Meski FAISS tetap menemukan hasil dengan similarity 0.6446, Gemini tidak mengarang harga atau mengutip FAQ yang tidak relevan. Sampel ini menunjukkan penanganan ketidaktahuan sesuai instruksi prompt.

### Ringkasan Hasil

- Pada empat query dengan FAQ acuan, FAQ target ditemukan di peringkat pertama: **Hit@1 = 4/4 (100%)** dan **Hit@3 = 4/4 (100%)**.
- Kedua query langsung dan kedua parafrase berhasil pada sampel yang diuji.
- Query di luar FAQ menghasilkan pengakuan keterbatasan informasi.
- Berdasarkan perbandingan jawaban tersimpan dengan CSV, keempat jawaban yang tersedia mempertahankan fakta serta syarat utama FAQ; tidak terlihat penambahan angka atau kebijakan yang tidak didukung pada lima sampel ini.

Query di luar FAQ tidak dimasukkan dalam perhitungan hit rate karena tidak memiliki FAQ target. Angka 100% hanya berlaku untuk empat query acuan ini, bukan ukuran akurasi pada seluruh pertanyaan pelanggan. Analisis kualitas jawaban di atas merupakan peninjauan atas hasil tersimpan, bukan penilaian otomatis atau klaim bebas halusinasi.

## 7. Batasan dan Asumsi

- Sistem hanya menjawab dari FAQ; tidak mengakses status pesanan, akun, stok, atau harga pasar secara langsung.
- Knowledge base kecil dan statis. Lima query belum mewakili seluruh kategori, variasi bahasa, atau pertanyaan ambigu.
- FAISS selalu mengembalikan tetangga terdekat, termasuk untuk pertanyaan di luar domain. Tidak ada threshold similarity yang dikalibrasi untuk menyatakan informasi tersedia.
- Prompt mengarahkan grounding, tetapi tidak menjamin seluruh jawaban berikutnya bebas halusinasi. Nomor FAQ yang disebut LLM belum divalidasi otomatis.
- Jawaban berupa teks biasa. Tidak ada riwayat percakapan, retry otomatis, cache embedding, atau validasi metadata index.
- Index dibuat ulang saat Run All. Jika CSV, model, atau dimensi berubah, seluruh alur harus dijalankan ulang agar embedding dan index konsisten.
- Akses model, koneksi, dan kuota API memengaruhi keberhasilan eksekusi. Hasil Gemini dapat berbeda ketika pengujian diulang.

## 8. Output dan Pengumpulan Assignment

Output yang tersedia:

- Kode pemrosesan FAQ, embedding, FAISS, semantic search, prompt, integrasi LLM, dan pengujian dalam satu notebook.
- `artifacts/faq.index` dan `artifacts/embeddings.npy`.
- Hasil lima query dalam output notebook serta `documentation/retrieval_test_results.md`.
- Analisis relevansi dan kualitas jawaban dalam README ini.
- `requirements.txt` untuk instalasi dependensi.

Kumpulkan seluruh output dalam satu repository GitHub dengan format nama **`assignment-semantic-search-nama-peserta`**. Repository harus dapat diakses mentor, melalui repository publik atau pemberian akses sesuai ketentuan assignment. Jangan unggah `.env`, API key, `.venv`, cache, atau file sementara.

Notebook masih memiliki catatan Markdown lama yang menyatakan pengujian ditunda, dan laporan otomatis masih memberi pengingat review manual. Status hasil aktual dalam README ini mengacu pada output pengujian yang sudah tersimpan serta file artifacts yang tersedia. Tahap unggah GitHub dilakukan terpisah dari penulisan dokumentasi ini.
