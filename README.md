# Semantic Search & LLM Application: Asisten FAQ E-Commerce

Prototype asisten customer service yang menjawab pertanyaan pelanggan berdasarkan knowledge base FAQ. Sistem mencari entri FAQ yang paling relevan secara **semantik** (berdasarkan makna, bukan kecocokan kata kunci) menggunakan **Gemini Embeddings + FAISS**, lalu menyusun jawaban yang natural dengan **Gemini LLM**. Jawaban hanya boleh berasal dari FAQ yang ditemukan.

Prototype ini menjadi tahap awal sebelum dikembangkan menjadi sistem RAG (Retrieval-Augmented Generation) penuh.

---

## 1. Problem Statement

### Objective
Tim customer service sebuah e-commerce menerima ratusan pertanyaan repetitif setiap hari seputar pemesanan, pengiriman, pembayaran, dan pengembalian barang. Tujuan prototype ini:

1. **Menemukan** entri FAQ paling relevan untuk pertanyaan pelanggan, **walaupun pelanggan memakai kata-kata berbeda** dari FAQ (sinonim, parafrase, bahasa informal).
2. **Menjawab** pertanyaan dengan bahasa yang natural menggunakan LLM, **berdasarkan FAQ yang ditemukan saja**.
3. **Mengakui ketidaktahuan**: jika pertanyaan tidak tercakup di FAQ, sistem menyampaikannya dengan jujur dan mengarahkan pelanggan ke customer service, alih-alih mengarang jawaban.

### Sumber & struktur knowledge base
- **File:** [`data/ecommerce_faq_knowledge_base.csv`](data/ecommerce_faq_knowledge_base.csv). File ini disediakan oleh program dan **tidak diubah**; semua transformasi dilakukan di memori.
- **Ukuran:** 59 entri, 4 kolom, tanpa missing value dan tanpa duplikat.

| Kolom | Tipe | Keterangan |
|---|---|---|
| `faq_id` | int | ID unik entri (1–59) |
| `category` | str | Kategori topik (8 kategori) |
| `question` | str | Pertanyaan FAQ (5–11 kata) |
| `answer` | str | Jawaban FAQ (11–23 kata) |

| Kategori | Pemesanan | Pengiriman | Pembayaran | Pengembalian | Akun | Produk | Promo | Customer Service |
|---|---|---|---|---|---|---|---|---|
| Jumlah entri | 9 | 9 | 8 | 8 | 7 | 7 | 6 | 5 |

### Batasan & asumsi
- Sistem **hanya menjawab berdasarkan 59 FAQ** yang tersedia, tanpa pengetahuan umum LLM, data pesanan, atau data produk real-time.
- Sistem **tidak mengakses data pelanggan** (status pesanan, saldo, resi). Pertanyaan seperti "di mana paket saya?" hanya dijawab dengan prosedur umum dari FAQ.
- Pertanyaan diasumsikan dalam **Bahasa Indonesia**, termasuk bahasa informal.
- Setiap entri FAQ pendek, sehingga **satu entri = satu unit retrievable** (chunking tidak diperlukan).
- Prototype ini **single-turn**: tidak ada riwayat percakapan.

---

## 2. Alur Sistem

```
 ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐
 │  INGEST    │──▶│   EMBED    │──▶│   STORE    │──▶│  RETRIEVE  │──▶│  GENERATE  │
 │ CSV→Pandas │   │ Gemini Emb │   │   FAISS    │   │ top-k FAQ  │   │ Prompt+LLM │
 └────────────┘   └────────────┘   └────────────┘   └────────────┘   └────────────┘
```

| Tahap | Implementasi | Detail |
|---|---|---|
| **Ingest** | `pandas.read_csv` → eksplorasi data → kolom `content` | Setiap entri digabung menjadi `Kategori: … / Pertanyaan: … / Jawaban: …`. Kategori ikut disertakan sebagai sinyal topik tambahan. |
| **Embed** | `gemini-embedding-001` | Dikirim per batch. `task_type="RETRIEVAL_DOCUMENT"` untuk FAQ, `"RETRIEVAL_QUERY"` untuk query (embedding asimetris yang dioptimasi untuk retrieval). Hasil: matriks **(59, 3072)**, dinormalisasi L2. |
| **Store** | `faiss.IndexIDMap(faiss.IndexFlatIP(3072))` | Inner product atas vektor ternormalisasi = **cosine similarity** (makin mendekati 1 makin mirip). ID index = `faq_id`. Terverifikasi `index.ntotal == 59`. Disimpan ke [`vector_store/faq_index.faiss`](vector_store/). |
| **Retrieve** | `semantic_search(query, top_k=3)` | Query di-embed lalu dicari di FAISS. Fungsi mengembalikan top-3 FAQ beserta `similarity`. |
| **Generate** | `build_prompt()` + `generate_answer()` | Top-3 FAQ disisipkan ke prompt terstruktur, lalu dikirim ke Gemini (`gemini-3.5-flash-lite`, dengan fallback ke `gemini-3.1-flash-lite`) dengan `temperature=0.2`. |

### Desain prompt (Persona, Task, Context, Format)

| Bagian | Isi |
|---|---|
| **Persona** | "Asisten CS", asisten virtual CS e-commerce yang ramah dan sopan, berbahasa Indonesia. |
| **Task** | Jawab **hanya** dari CONTEXT; dilarang mengarang angka, durasi, biaya, atau kebijakan. Pilih FAQ yang relevan dan abaikan sisanya. Jika informasi tidak ada, gunakan kalimat penolakan baku dan arahkan ke customer service. Jika pertanyaan hanya sebagian terjawab, sebutkan bagian yang belum tersedia. |
| **Context** | Top-3 hasil semantic search, masing-masing berformat `[FAQ #id] (kategori, similarity)` + pertanyaan + jawaban. |
| **Format** | 2–4 kalimat atau poin bernomor untuk langkah-langkah, diakhiri `Sumber: [FAQ #id]` (atau `Sumber: -`) agar jawaban bisa diaudit. |

Template lengkap ada di Section 6 notebook.

---

## 3. Hasil Pengujian & Analisis Relevansi

Pengujian memakai 8 query: 2 direct, 4 parafrase/sinonim, dan 2 di luar knowledge base. Setiap query diberi *expected FAQ id* (ground truth yang ditentukan manual) sehingga relevansi retrieval diukur dengan **Hit@1** (FAQ benar di peringkat 1) dan **Hit@3** (FAQ benar ada di top-3).

| # | Tipe | Query | Expected | Retrieved top-3 | Top-1 sim | Hit@1 | Jawaban LLM |
|---|---|---|---|---|---|---|---|
| 1 | Direct | Bagaimana cara membatalkan pesanan? | 3 | **3**, 24, 8 | 0.780 | ✅ | ✅ Langkah pembatalan + batasan status "Dikirim" |
| 2 | Direct | Metode pembayaran apa saja yang bisa digunakan? | 17 | **17**, 22, 20 | 0.791 | ✅ | ✅ Menyebut 4 metode sesuai FAQ |
| 3 | Parafrase | Barang yang saya beli datang dalam keadaan **pecah**, saya harus gimana? | 16 | **16**, 54, 24 | 0.768 | ✅ | ✅ Foto/video + lapor ≤ 2x24 jam |
| 4 | Parafrase | Kalau saya **retur** barang, kapan **uang saya balik**? | 25 | **25**, 27, 24 | 0.789 | ✅ | ✅ Menggabungkan FAQ #25 + #27 |
| 5 | Parafrase | Saya gak bisa **masuk ke akun** saya, kenapa ya? | 55 | **55**, 32, 34 | 0.792 | ✅ | ✅ Penyebab akun terkunci + solusi |
| 6 | Parafrase | Bisa bayar pakai **GoPay atau OVO** nggak? | 17 | **17**, 22, 37 | 0.721 | ✅ | ⚠️ Benar ("e-wallet didukung"), tapi tidak menyebut bahwa merek spesifik tidak dirinci di FAQ |
| 7 | Out-of-KB | Apakah bisa kirim barang ke luar negeri? | – | 15, 9, 5 | 0.710 | – | ✅ Penolakan baku, `Sumber: -` |
| 8 | Out-of-KB | Berapa harga iPhone 16 Pro di toko kalian? | – | 58, 1, 57 | 0.643 | – | ✅ Penolakan baku, `Sumber: -` |

**Metrik retrieval (6 query in-KB): Hit@1 = 6/6 (100%), Hit@3 = 6/6 (100%).**

| Tipe query | Rata-rata top-1 similarity | Min | Max |
|---|---|---|---|
| Direct | 0.785 | 0.780 | 0.791 |
| Parafrase | 0.768 | 0.721 | 0.792 |
| Out-of-KB | 0.677 | 0.643 | 0.710 |

Detail lengkap tiap query (top-3, skor, dan jawaban LLM utuh) tersedia di [`documentation/retrieval_test_results.md`](documentation/retrieval_test_results.md).

### Analisis

**Retrieval yang akurat.** Semua query yang ada di KB mendapatkan FAQ yang benar di rank 1. Query parafrase menunjukkan semantic search benar-benar menangkap **makna**, bukan kata kunci:
- *"pecah"* ↔ *"rusak"*, *"datang"* ↔ *"diterima"* (#3): tidak ada kata kunci yang sama sama sekali.
- *"retur"*, *"uang balik"* ↔ *"pengembalian"*, *"refund"* (#4): bahasa informal dipetakan ke istilah formal.
- *"masuk ke akun"* ↔ *"login"* (#5): sinonim lintas bahasa ikut tertangkap.
- *"GoPay/OVO"* ↔ *"e-wallet"* (#6): model embedding memahami bahwa GoPay dan OVO adalah e-wallet.

**Hasil yang kurang relevan.** Kandidat di rank 2–3 sering satu topik tetapi tidak menjawab pertanyaan. Contohnya, query #1 (batal pesanan) juga mengambil FAQ #24 (pengembalian barang). Ini wajar untuk top-k = 3, dan LLM berhasil mengabaikannya karena prompt menyatakan bahwa tidak semua context pasti relevan. Query #6 memiliki top-1 similarity paling rendah di antara query in-KB (0.721) karena hubungan "GoPay/OVO → e-wallet" membutuhkan pengetahuan dunia, bukan sekadar kemiripan kata.

**Respons terhadap query di luar KB.** FAISS **selalu** mengembalikan top-k, berapa pun skornya. Pada query #7 ("kirim ke luar negeri"), FAQ pengiriman yang mirip topiknya ikut ter-retrieve. LLM tidak tergoda menyimpulkan dari FAQ #9 ("luar pulau") dan menjawab dengan penolakan baku. Ini membuktikan instruksi grounding di prompt bekerja.

**Temuan penting soal threshold.** Rentang similarity sempit dan **saling tumpang tindih**: query out-of-KB #7 (0.710) hampir setara dengan query in-KB #6 (0.721). Threshold similarity saja **tidak cukup** untuk memisahkan pertanyaan yang bisa dan tidak bisa dijawab. Kalau dipasang di 0.72, query valid #6 ikut tertolak. Pada prototype ini, penjaga utama terhadap halusinasi adalah **instruksi di prompt**, bukan skor retrieval.

**Kualitas jawaban LLM.** Jawaban faktual, ringkas, dan hanya memakai fakta dari FAQ: angka seperti "2x24 jam" dan "3–7 hari kerja" dikutip tepat tanpa tambahan. Langkah-langkah diformat sebagai poin bernomor, dan setiap jawaban menyertakan sumber `[FAQ #id]`. LLM juga mampu **menyintesis** beberapa FAQ (#4: durasi refund dan tujuan dana sekaligus).

---

## 4. Keterbatasan Sistem

1. **Tidak ada ambang relevansi yang andal.** Karena skor in-KB dan out-of-KB tumpang tindih (0.71 vs 0.72), sistem selalu mengirim top-3 ke LLM dan sepenuhnya bergantung pada kepatuhan LLM terhadap prompt. Jika LLM gagal mematuhi instruksi, halusinasi bisa terjadi.
2. **Jawaban parsial kurang presisi.** Pada query #6, LLM menjawab "e-wallet didukung" tanpa menyebut bahwa FAQ tidak merinci merek e-wallet tertentu. Pelanggan bisa menganggap GoPay/OVO pasti didukung. Aturan "jawaban parsial" di prompt belum selalu dipatuhi oleh model kecil (flash-lite).
3. **Output LLM non-deterministik.** Meskipun `temperature=0.2`, wording jawaban bisa berbeda antar-run. Hasil retrieval (embedding + FAISS) bersifat stabil.
4. **Knowledge base kecil dan statis.** 59 FAQ mencakup topik umum saja. Pertanyaan spesifik (harga produk, status pesanan pribadi, pengiriman internasional) tidak bisa dijawab. Setiap perubahan FAQ memerlukan pembuatan ulang embedding dan index.
5. **Single-turn, tanpa memori percakapan.** Pertanyaan lanjutan seperti "kalau yang itu berapa lama?" tidak dipahami konteksnya.
6. **Evaluasi terbatas.** Pengujian memakai 8 query dengan ground truth manual. Evaluasi yang lebih kuat memerlukan set query yang lebih besar dan beragam (typo, multi-intent, campuran bahasa Inggris), serta penilaian kualitas jawaban yang lebih sistematis.
7. **Ketergantungan pada API eksternal.** Pipeline membutuhkan koneksi internet dan kuota Gemini API (free tier memiliki rate limit). Model `text-embedding-004` dan `gemini-2.0-flash` di starter notebook sudah tidak tersedia, sehingga diganti dengan `gemini-embedding-001` dan `gemini-3.5-flash-lite`.

**Ide pengembangan (menuju RAG):** hybrid search (BM25 + embedding), re-ranking, pemisahan "jawab/tolak" dengan klasifikasi terpisah, penambahan metadata filter per kategori, dan evaluasi otomatis (misalnya LLM-as-judge untuk faithfulness).

---

## 5. Cara Menjalankan

### Prasyarat
- Python 3.10+ (diuji dengan Python 3.14)
- API key Gemini dari [Google AI Studio](https://aistudio.google.com/apikey)

### Langkah
```bash
# 1. Clone repository
git clone <url-repository>
cd semantic-search-assignment

# 2. Buat & aktifkan virtual environment
python -m venv .venv
# Windows (PowerShell)
.venv\Scripts\Activate.ps1
# macOS / Linux
source .venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Buat file .env di root project berisi API key
echo GEMINI_API_KEY=your_api_key_here > .env
```

5. Buka [`notebook/starter_notebook_semantic_search.ipynb`](notebook/starter_notebook_semantic_search.ipynb) di VS Code atau Jupyter, pilih kernel `.venv`, lalu **Run All**.

   Atau jalankan dari terminal:
   ```bash
   jupyter nbconvert --to notebook --execute --inplace notebook/starter_notebook_semantic_search.ipynb
   ```
   (perintah ini memerlukan `pip install nbconvert`)

Notebook akan membuat ulang `vector_store/faq_index.faiss`, `vector_store/faq_metadata.csv`, dan `documentation/retrieval_test_results.md`.

> Jika muncul error `429 RESOURCE_EXHAUSTED`, kuota Gemini API sedang habis. Notebook otomatis melakukan retry dan fallback ke model LLM lain; jika tetap gagal, tunggu beberapa menit lalu jalankan ulang.

---

## 6. Struktur Project

```
semantic-search-assignment/
├── data/
│   └── ecommerce_faq_knowledge_base.csv      # knowledge base asli (tidak diubah)
├── notebook/
│   └── starter_notebook_semantic_search.ipynb # pipeline lengkap: ingest → embed → store → retrieve → generate → uji
├── vector_store/
│   ├── faq_index.faiss                       # FAISS index (59 vektor × 3072 dimensi)
│   └── faq_metadata.csv                      # metadata FAQ untuk memetakan ID index → entri
├── documentation/
│   └── retrieval_test_results.md             # hasil mentah pengujian retrieval & jawaban LLM
├── .gitignore                                # mengecualikan .env & .venv
├── README.md
└── requirements.txt
```

## 7. Tools
Python · Pandas · NumPy · FAISS (`faiss-cpu`) · Google Gemini API (`google-genai`): `gemini-embedding-001`, `gemini-3.5-flash-lite` · python-dotenv · Jupyter
