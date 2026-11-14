# Retrieval Test Results

- Embedding model: `gemini-embedding-001` (dimensi 3072)
- Vector DB: FAISS `IndexFlatIP` (cosine similarity), 59 vektor
- top_k = 3

> Analisis relevansi hasil pengujian ini ada di [README.md](../README.md#3-hasil-pengujian--analisis-relevansi) dan di bagian akhir notebook.

## Ringkasan

| No | Tipe | Query | Expected FAQ | Retrieved top-3 | Top-1 sim | Hit@1 | Hit@3 |
|---|---|---|---|---|---|---|---|
| 1 | Direct | Bagaimana cara membatalkan pesanan? | [3] | [3, 24, 8] | 0.780 | ✅ | ✅ |
| 2 | Direct | Metode pembayaran apa saja yang bisa digunakan? | [17] | [17, 22, 20] | 0.791 | ✅ | ✅ |
| 3 | Paraphrase | Barang yang saya beli datang dalam keadaan pecah, saya harus gimana? | [16] | [16, 54, 24] | 0.768 | ✅ | ✅ |
| 4 | Paraphrase | Kalau saya retur barang, kapan uang saya balik? | [25] | [25, 27, 24] | 0.789 | ✅ | ✅ |
| 5 | Paraphrase | Saya gak bisa masuk ke akun saya, kenapa ya? | [55] | [55, 32, 34] | 0.792 | ✅ | ✅ |
| 6 | Paraphrase | Bisa bayar pakai GoPay atau OVO nggak? | [17] | [17, 22, 37] | 0.721 | ✅ | ✅ |
| 7 | Out-of-KB | Apakah bisa kirim barang ke luar negeri? | - | [15, 9, 5] | 0.710 | - | - |
| 8 | Out-of-KB | Berapa harga iPhone 16 Pro di toko kalian? | - | [58, 1, 57] | 0.643 | - | - |

## Detail per query

### 1. [Direct] Bagaimana cara membatalkan pesanan?

Expected FAQ: [3]

| Rank | FAQ id | Kategori | Pertanyaan FAQ | Similarity |
|---|---|---|---|---|
| 1 | 3 | Pemesanan | Bagaimana cara membatalkan pesanan yang sudah dibuat? | 0.780 |
| 2 | 24 | Pengembalian | Bagaimana prosedur pengembalian barang yang tidak sesuai? | 0.714 |
| 3 | 8 | Pemesanan | Kenapa pesanan saya otomatis dibatalkan sistem? | 0.703 |

**Jawaban LLM:**

> Halo Kak! Pesanan Kakak dapat dibatalkan dengan mengikuti langkah-langkah berikut:
> 
> 1. Buka halaman 'Pesanan Saya'.
> 2. Lakukan pembatalan selama status pesanan masih 'Diproses'. 
> 
> Perlu diketahui bahwa setelah status berubah menjadi 'Dikirim', pembatalan sudah tidak dapat dilakukan ya Kak.
> 
> Sumber: [FAQ #3]

### 2. [Direct] Metode pembayaran apa saja yang bisa digunakan?

Expected FAQ: [17]

| Rank | FAQ id | Kategori | Pertanyaan FAQ | Similarity |
|---|---|---|---|---|
| 1 | 17 | Pembayaran | Metode pembayaran apa saja yang tersedia? | 0.791 |
| 2 | 22 | Pembayaran | Apakah bisa membayar menggunakan lebih dari satu metode sekaligus? | 0.731 |
| 3 | 20 | Pembayaran | Apakah saya bisa membayar dengan cicilan? | 0.704 |

**Jawaban LLM:**

> Halo! Kami mendukung beberapa metode pembayaran untuk transaksi Anda, yaitu transfer bank, kartu kredit/debit, e-wallet, dan pembayaran melalui minimarket rekanan. 
> 
> Sumber: [FAQ #17]

### 3. [Paraphrase] Barang yang saya beli datang dalam keadaan pecah, saya harus gimana?

Expected FAQ: [16]

| Rank | FAQ id | Kategori | Pertanyaan FAQ | Similarity |
|---|---|---|---|---|
| 1 | 16 | Pengiriman | Apa yang harus dilakukan jika paket diterima dalam kondisi rusak? | 0.768 |
| 2 | 54 | Pengembalian | Saya menerima barang yang salah, apa yang harus saya lakukan? | 0.728 |
| 3 | 24 | Pengembalian | Bagaimana prosedur pengembalian barang yang tidak sesuai? | 0.715 |

**Jawaban LLM:**

> Halo! Mohon maaf atas kendala paket yang diterima dalam kondisi pecah atau rusak. Berikut adalah langkah-langkah yang harus Anda lakukan:
> 
> 1. Segera ambil foto atau video kondisi paket saat diterima.
> 2. Laporkan melalui menu komplain dalam waktu 2x24 jam sejak barang diterima.
> 
> Sumber: [FAQ #16]

### 4. [Paraphrase] Kalau saya retur barang, kapan uang saya balik?

Expected FAQ: [25]

| Rank | FAQ id | Kategori | Pertanyaan FAQ | Similarity |
|---|---|---|---|---|
| 1 | 25 | Pengembalian | Berapa lama proses refund setelah pengembalian disetujui? | 0.789 |
| 2 | 27 | Pengembalian | Ke mana dana refund akan dikembalikan? | 0.732 |
| 3 | 24 | Pengembalian | Bagaimana prosedur pengembalian barang yang tidak sesuai? | 0.729 |

**Jawaban LLM:**

> Halo! Dana refund biasanya akan diproses dalam waktu 3-7 hari kerja setelah barang dikembalikan diterima dan diverifikasi oleh penjual. Dana tersebut akan dikembalikan ke metode pembayaran awal atau ke saldo akun, tergantung pada opsi yang Anda pilih saat pengajuan.
> 
> Sumber: [FAQ #25], [FAQ #27]

### 5. [Paraphrase] Saya gak bisa masuk ke akun saya, kenapa ya?

Expected FAQ: [55]

| Rank | FAQ id | Kategori | Pertanyaan FAQ | Similarity |
|---|---|---|---|---|
| 1 | 55 | Akun | Kenapa akun saya tiba-tiba tidak bisa login? | 0.792 |
| 2 | 32 | Akun | Saya lupa password akun, bagaimana cara mengaturnya ulang? | 0.743 |
| 3 | 34 | Akun | Apakah bisa memiliki lebih dari satu akun dengan email yang sama? | 0.686 |

**Jawaban LLM:**

> Halo! Akun Anda dapat terkunci sementara karena adanya kesalahan login berulang atau terindikasi adanya aktivitas mencurigakan. Untuk mengatasinya, silakan lakukan reset password atau hubungi customer service kami.
> 
> Sumber: [FAQ #55]

### 6. [Paraphrase] Bisa bayar pakai GoPay atau OVO nggak?

Expected FAQ: [17]

| Rank | FAQ id | Kategori | Pertanyaan FAQ | Similarity |
|---|---|---|---|---|
| 1 | 17 | Pembayaran | Metode pembayaran apa saja yang tersedia? | 0.721 |
| 2 | 22 | Pembayaran | Apakah bisa membayar menggunakan lebih dari satu metode sekaligus? | 0.704 |
| 3 | 37 | Promo | Bagaimana cara menggunakan kode voucher saat checkout? | 0.666 |

**Jawaban LLM:**

> Halo! Platform kami mendukung pembayaran menggunakan e-wallet. Anda dapat memilih opsi tersebut saat melakukan transaksi pembayaran.
> 
> Sumber: [FAQ #17]

### 7. [Out-of-KB] Apakah bisa kirim barang ke luar negeri?

Expected FAQ: - (tidak ada di knowledge base)

| Rank | FAQ id | Kategori | Pertanyaan FAQ | Similarity |
|---|---|---|---|---|
| 1 | 15 | Pengiriman | Apakah bisa memilih jasa pengiriman sendiri saat checkout? | 0.710 |
| 2 | 9 | Pengiriman | Berapa lama estimasi waktu pengiriman barang? | 0.709 |
| 3 | 5 | Pemesanan | Apakah saya bisa mengubah alamat pengiriman setelah pesanan dibuat? | 0.702 |

**Jawaban LLM:**

> Mohon maaf, saya belum memiliki informasi mengenai hal tersebut di FAQ kami. Jika Anda membutuhkan bantuan lebih lanjut, silakan hubungi customer service kami.
> 
> Sumber: -

### 8. [Out-of-KB] Berapa harga iPhone 16 Pro di toko kalian?

Expected FAQ: - (tidak ada di knowledge base)

| Rank | FAQ id | Kategori | Pertanyaan FAQ | Similarity |
|---|---|---|---|---|
| 1 | 58 | Pemesanan | Apakah bisa memesan dalam jumlah besar untuk kebutuhan reseller? | 0.643 |
| 2 | 1 | Pemesanan | Bagaimana cara melakukan pemesanan di platform ini? | 0.633 |
| 3 | 57 | Produk | Bagaimana cara membandingkan beberapa produk sekaligus? | 0.621 |

**Jawaban LLM:**

> Mohon maaf, saya belum memiliki informasi mengenai hal tersebut di FAQ kami. Silakan menghubungi customer service kami untuk mendapatkan bantuan lebih lanjut.
> 
> Sumber: -
