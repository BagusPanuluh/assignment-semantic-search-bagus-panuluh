# Hasil pengujian relevansi

Model embedding: gemini-embedding-001; LLM: gemini-3.8-flash.

## langsung: Bagaimana cara melacak paket yang sedang dikirim?

FAQ acuan: 10

### Kandidat retrieval

- FAQ 10: Bagaimana cara melacak paket yang sedang dikirim? (distance=0.3984, similarity=0.8008)
- FAQ 52: Kenapa nomor resi saya belum bisa dilacak di website kurir? (distance=0.5133, similarity=0.7433)
- FAQ 12: Bagaimana jika paket saya belum sampai melebihi estimasi? (distance=0.5237, similarity=0.7381)

### Jawaban Gemini

Halo! Berdasarkan FAQ 10, nomor resi akan muncul di halaman 'Pesanan Saya' setelah barang dikirim, dan dapat digunakan untuk melacak posisi paket di website kurir terkait.

### Analisis retrieval

FAQ acuan ditemukan pada peringkat pertama; retrieval sesuai target.

Review kualitas jawaban: belum ditinjau manusia. Cocokkan angka, syarat, dan setiap klaim dengan FAQ.

## langsung: Berapa lama proses refund setelah pengembalian disetujui?

FAQ acuan: 25

### Kandidat retrieval

- FAQ 25: Berapa lama proses refund setelah pengembalian disetujui? (distance=0.3739, similarity=0.8131)
- FAQ 27: Ke mana dana refund akan dikembalikan? (distance=0.5834, similarity=0.7083)
- FAQ 41: Apakah cashback dari promo langsung masuk ke saldo? (distance=0.6019, similarity=0.6990)

### Jawaban Gemini

Halo! Dana refund biasanya diproses dalam 3-7 hari kerja setelah barang dikembalikan diterima dan diverifikasi penjual (berdasarkan FAQ 25).

### Analisis retrieval

FAQ acuan ditemukan pada peringkat pertama; retrieval sesuai target.

Review kualitas jawaban: belum ditinjau manusia. Cocokkan angka, syarat, dan setiap klaim dengan FAQ.

## parafrase: Paketku telat lebih dari tiga hari dari jadwal, harus lapor ke siapa?

FAQ acuan: 12

### Kandidat retrieval

- FAQ 12: Bagaimana jika paket saya belum sampai melebihi estimasi? (distance=0.3957, similarity=0.8021)
- FAQ 14: Kenapa status pengiriman saya tidak berubah selama beberapa hari? (distance=0.4986, similarity=0.7507)
- FAQ 16: Apa yang harus dilakukan jika paket diterima dalam kondisi rusak? (distance=0.5261, similarity=0.7370)

### Jawaban Gemini

Halo! Berdasarkan **FAQ 12**, jika paket Anda melebihi estimasi pengiriman lebih dari 3 hari, silakan hubungi **customer service** dengan menyertakan nomor resi agar dapat ditelusuri lebih lanjut.

### Analisis retrieval

FAQ acuan ditemukan pada peringkat pertama; retrieval sesuai target.

Review kualitas jawaban: belum ditinjau manusia. Cocokkan angka, syarat, dan setiap klaim dengan FAQ.

## parafrase: Bisa nggak belanja dibayar nyicil dengan kartu kredit?

FAQ acuan: 20

### Kandidat retrieval

- FAQ 20: Apakah saya bisa membayar dengan cicilan? (distance=0.3965, similarity=0.8017)
- FAQ 18: Apakah ada biaya tambahan untuk pembayaran menggunakan kartu kredit? (distance=0.6455, similarity=0.6772)
- FAQ 22: Apakah bisa membayar menggunakan lebih dari satu metode sekaligus? (distance=0.6597, similarity=0.6701)

### Jawaban Gemini

Halo! Bisa, fitur cicilan tersedia untuk transaksi dengan kartu kredit tertentu pada nominal minimal yang ditentukan pihak bank (FAQ 20).

### Analisis retrieval

FAQ acuan ditemukan pada peringkat pertama; retrieval sesuai target.

Review kualitas jawaban: belum ditinjau manusia. Cocokkan angka, syarat, dan setiap klaim dengan FAQ.

## di luar FAQ: Berapa harga saham perusahaan ini hari ini?

FAQ acuan: None

### Kandidat retrieval

- FAQ 43: Apakah stok yang tertera di aplikasi selalu update secara real-time? (distance=0.7108, similarity=0.6446)
- FAQ 58: Apakah bisa memesan dalam jumlah besar untuk kebutuhan reseller? (distance=0.7456, similarity=0.6272)
- FAQ 40: Bagaimana cara mendapatkan info promo terbaru? (distance=0.7477, similarity=0.6261)

### Jawaban Gemini

Saya belum memiliki informasi tersebut dalam FAQ. Silakan hubungi customer service.

### Analisis retrieval

Tidak ada jawaban di FAQ. Periksa apakah LLM mengakui informasi tidak tersedia, tanpa mengarang fakta.

Review kualitas jawaban: belum ditinjau manusia. Cocokkan angka, syarat, dan setiap klaim dengan FAQ.

## Keterbatasan

Lima query adalah sampel kecil. Retrieval yang tepat belum menjamin jawaban bebas halusinasi. Review kualitas jawaban tetap perlu dilakukan secara manual.