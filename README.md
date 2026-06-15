# RAG Panduan Skripsi S-1 Ilmu Komputer

Sistem **Retrieval-Augmented Generation (RAG)** untuk menjawab pertanyaan seputar pedoman tugas akhir/skripsi Program Studi S-1 Ilmu Komputer, Fakultas Ilmu Komputer dan Teknologi Informasi, Universitas Sumatera Utara.

## 📋 Deskripsi

Proyek ini mengimplementasikan sistem tanya jawab cerdas berbasis RAG yang memungkinkan mahasiswa untuk mengajukan pertanyaan tentang pedoman skripsi dan mendapatkan jawaban yang akurat berdasarkan dokumen resmi prodi. Sistem ini menggunakan pendekatan **semantic chunking** untuk membagi dokumen secara cerdas, **vector database** (Qdrant) untuk penyimpanan dan pencarian, serta **LLM** (Groq dengan model Llama 4) untuk menghasilkan jawaban yang kontekstual.

## ✨ Fitur Utama

- **📄 Dokumentasi Lengkap**: Menggunakan 4 dokumen pedoman skripsi resmi dalam format Markdown
- **✂️ Semantic Chunking**: Pembagian dokumen berbasis semantik dengan sentence embedding dan percentile breakpoint detection
- **🔍 Vector Retrieval**: Pencarian semantik menggunakan Qdrant dengan cosine similarity
- **🎯 CrossEncoder Reranking**: Peningkatan akurasi dengan reranking menggunakan CrossEncoder (ms-marco-MiniLM)
- **🤖 LLM Generation**: Jawaban dihasilkan oleh Groq (meta-llama/llama-4-scout-17b-16e-instruct)
- **💬 Mode Interaktif**: Tanya jawab berulang dalam satu sesi
- **📊 Evaluasi RAGAS**: Evaluasi kualitas sistem RAG menggunakan framework RAGAS

## 🏗️ Arsitektur

```
┌─────────────────────────────────────────────────────────────┐
│                     RAG Pipeline                             │
├─────────────────────────────────────────────────────────────┤
│  1. Document Loading → Load 4 markdown files                │
│  2. Semantic Chunking → Split by semantic similarity         │
│  3. Vector Embedding → paraphrase-multilingual-MiniLM-L12-v2│
│  4. Vector Storage → Qdrant (cloud server)                   │
│  5. Query Processing → User question input                   │
│  6. Retrieval (Top-K=25) → Vector similarity search          │
│  7. Reranking (Top-5) → CrossEncoder refinement              │
│  8. Prompt Generation → Context + Query template             │
│  9. LLM Generation → Groq (Llama 4)                          │
│ 10. Answer Display → Markdown formatted response             │
└─────────────────────────────────────────────────────────────┘
```

## 📦 Dokumen Sumber

Sistem ini menggunakan 4 dokumen pedoman resmi:

| File | Deskripsi | Ukuran |
|------|-----------|--------|
| `Pedoman tulisan tanpa gambar.md` | Pedoman penulisan skripsi lengkap | 61,483 karakter |
| `Pedoman skripsi tambahan.md` | Pedoman tambahan pelaksanaan TA | 25,807 karakter |
| `Pedoman tulisan hanya gambar.md` | Pedoman penulisan (versi gambar) | 14,206 karakter |
| `Prosedur Tugas Akhir Prodi S-1 Ilmu Komputer.md` | Prosedur administrasi TA | 10,154 karakter |

## 🛠️ Teknologi

### Core Stack
- **Python 3.12+**: Bahasa pemrograman utama
- **Jupyter Notebook**: Environment development
- **Kaggle**: Platform eksekusi notebook

### AI/ML Libraries
- **sentence-transformers**: Embedding model & CrossEncoder
  - `paraphrase-multilingual-MiniLM-L12-v2` (384 dimensi)
  - `cross-encoder/ms-marco-MiniLM-L-12-v2` (reranking)
- **semantic-text-splitter**: Semantic chunking algorithm
- **groq**: LLM API client
- **qdrant-client**: Vector database client

### Supporting Libraries
- `python-dotenv`: Environment variable management
- `glob`, `os`: File operations
- `numpy`: Numerical computations
- `tqdm`: Progress bars
- `IPython.display`: Markdown rendering

## ⚙️ Konfigurasi

### Environment Variables

Buat file `.env` dengan variabel berikut:

```env
QDRANT_URL=https://your-qdrant-cloud-url
QDRANT_API_KEY=your-qdrant-api-key
GROQ_API_KEY=your-groq-api-key
```

### Parameters

| Parameter | Value | Deskripsi |
|-----------|-------|-----------|
| `EMBEDDING_MODEL` | `paraphrase-multilingual-MiniLM-L12-v2` | Model untuk sentence embedding |
| `VECTOR_DIM` | `384` | Dimensi vektor embedding |
| `WINDOW_SIZE` | `2` | Window size untuk semantic chunking |
| `MIN_SENTENCES` | `2` | Minimum kalimat per chunk |
| `PERCENTILE_THRESH` | `80` | Breakpoint threshold (percentile) |
| `TOP_K` | `25` | Jumlah chunks yang di-retrieve |
| `TOP_N_RERANK` | `5` | Jumlah chunks setelah reranking |
| `GROQ_MODEL` | `meta-llama/llama-4-scout-17b-16e-instruct` | Model LLM |
| `MAX_TOKENS_OUTPUT` | `2048` | Maksimum token output LLM |
| `MAX_CHARS_PER_CHUNK` | `1500` | Maksimum karakter per chunk dalam context |

## 🚀 Cara Penggunaan

### 1. Persiapan Environment

```bash
# Install dependencies
pip install sentence-transformers qdrant-client groq semantic-text-splitter \
            python-dotenv numpy tqdm
```

### 2. Setup Credentials

Simpan API credentials di Kaggle Secrets atau file `.env`:

```python
from kaggle_secrets import UserSecretsClient
user_secrets = UserSecretsClient()

QDRANT_URL = user_secrets.get_secret("QDRANT_URL")
QDRANT_API_KEY = user_secrets.get_secret("QDRANT_API_KEY")
GROQ_API_KEY = user_secrets.get_secret("GROQ_API_KEY")
```

### 3. Jalankan Notebook

Buka dan jalankan notebook `RAG + Semantic Chunking.ipynb` secara berurutan:

1. **Konfigurasi & Load Environment Variables**
2. **Membaca File Markdown** - Load semua dokumen pedoman
3. **Semantic Chunking** - Bagi dokumen menjadi 191 chunks
4. **Vector Embedding** - Convert chunks ke vectors (384 dimensi)
5. **Simpan ke Qdrant** - Upload vectors ke vector database
6. **Input Query** - Masukkan pertanyaan
7. **Embedding Query** - Convert query ke vector
8. **Retrieval** - Cari top-25 chunks dari Qdrant
9. **Reranking** - Refine ke top-5 dengan CrossEncoder
10. **Generate Answer** - LLM menghasilkan jawaban
11. **Mode Interaktif** - Tanya jawab berulang (opsional)

### 4. Contoh Pertanyaan

```
🔍 Masukkan pertanyaan Anda: Apa langkah-langkah maju seminar proposal?
```

**Sample Output:**
```
**Langkah-Langkah Maju Seminar Proposal**

Berikut adalah langkah-langkah untuk maju seminar proposal:

1. **Mendapatkan Persetujuan Seminar Proposal**:
   - Mengirimkan file proposal dengan format `PROPOSAL_NIM.pdf`
   - Mengirimkan file Form Persetujuan Seminar Proposal yang sudah diisi 
     dan ditandatangani oleh Dosen Pembimbing

2. **Melakukan Bimbingan Proposal**:
   - Telah melakukan bimbingan proposal dengan kedua dosen pembimbing
   - Proposal sudah di-acc oleh kedua dosen pembimbing

3. **Mengikuti Workshop Seminar Proposal**:
   - Melakukan submission kembali dengan proposal yang sudah siap
```

## 📊 Evaluasi

Proyek ini menyertakan evaluasi menggunakan **RAGAS framework**:

- **File Hasil**: 
  - `ragas_hasil_20260614_091706.json` - Detail metrics
  - `ragas_hasil_20260614_091706.csv` - Format spreadsheet

- **Metrics yang Diukur**:
  - Answer Relevance
  - Context Precision
  - Context Recall
  - Faithfulness
  - (dan metrics RAGAS lainnya)

## 📁 Struktur Direktori

```
RAG Panduan S-1 Skripsi Ilmu Komputer/
├── RAG + Semantic Chunking.ipynb          # Notebook utama (implementasi lengkap)
├── rag-semantic-chunking.ipynb            # Notebook alternatif
├── convert-pdf-md.ipynb                   # Konversi PDF ke Markdown
├── output_md/                             # Dokumen sumber Markdown
│   ├── Pedoman skripsi tambahan.md
│   ├── Pedoman tulisan hanya gambar.md
│   ├── Pedoman tulisan tanpa gambar.md
│   └── Prosedur Tugas Akhir Prodi S-1 Ilmu Komputer.md
├── ragas_hasil_20260614_091706.json       # Hasil evaluasi RAGAS (JSON)
├── ragas_hasil_20260614_091706.csv        # Hasil evaluasi RAGAS (CSV)
└── README.md                              # Dokumentasi ini
```

## 🔍 Detail Implementasi

### Semantic Chunking Algorithm

1. **Sentence Tokenization**: Pisahkan teks menjadi kalimat menggunakan regex
2. **Sliding Window**: Buat groups dengan window size 2
3. **Embedding Groups**: Encode semua groups menggunakan sentence transformer
4. **Similarity Calculation**: Hitung cosine similarity antar group berurutan
5. **Breakpoint Detection**: Identifikasi breakpoint pada percentile ke-80
6. **Segment Merging**: Gabungkan segmen dengan minimum 2 kalimat
7. **Result**: 191 chunks dari 4 dokumen (rata-rata 5.8 kalimat/chunk)

### Retrieval & Reranking Pipeline

```
Query → Embed (384d) → Qdrant Search (Top-25) → CrossEncoder Rerank → Top-5
```

- **Retrieval Score**: Cosine similarity (0.0 - 1.0)
- **Rerank Score**: CrossEncoder logits (bisa negatif)
- **Final Context**: Top-5 reranked chunks dalam prompt template

### Prompt Template

```
Use the following pieces of information to answer the user's question.
If you don't know the answer, just say that you don't know, don't try to make up an answer.

Context:
[Source 1: prosedur.md]
...

Query: {user_question}

Answer the question and provide additional helpful information,
based on the pieces of information, if applicable. Be succinct.
```

## 📈 Statistik Sistem

- **Total Dokumen**: 4 files
- **Total Karakter**: ~111,650 karakter
- **Total Chunks**: 191 chunks
- **Rata-rata Kalimat/Chunk**: 5.8
- **Min Kalimat/Chunk**: 2
- **Max Kalimat/Chunk**: 44
- **Vector Dimension**: 384
- **Average Prompt Tokens**: ~1,199 tokens
- **Average Completion Tokens**: ~231 tokens

## 🎓 Topik yang Bisa Dijawab

Sistem dapat menjawab pertanyaan tentang:

✅ Persyaratan seminar proposal & hasil  
✅ Prosedur pengajuan judul & exum  
✅ Format penulisan skripsi  
✅ Penentuan dosen pembimbing  
✅ Batas waktu & deadline  
✅ Plagiarisme & cek turnitin  
✅ MBKM & kaitannya dengan TA  
✅ Persyaratan administrasi  
✅ Tahapan sidang meja hijau  
✅ Dan topik lainnya sesuai pedoman  

## 🤝 Kontribusi

Kontribusi sangat terbuka! Berikut cara berkontribusi:

1. Fork repository ini
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

## 📝 Lisensi

Proyek ini dibuat untuk keperluan akademis Program Studi S-1 Ilmu Komputer, Fasilkom-TI, Universitas Sumatera Utara.

## 👥 Author

**Brian Maxwell Ketaren**  
Program Studi S-1 Ilmu Komputer  
Fakultas Ilmu Komputer dan Teknologi Informasi  
Universitas Sumatera Utara

## 🙏 Acknowledgments

- **Universitas Sumatera Utara** - Pedoman skripsi resmi
- **Qdrant** - Vector database platform
- **Groq** - LLM inference API
- **Hugging Face** - Sentence Transformers library
- **RAGAS** - Evaluation framework
- **Kaggle** - Platform development & execution

## 📞 Contact

Untuk pertanyaan atau kolaborasi, silakan hubungi melalui:
- GitHub Issues
- Email: [your-email@students.usu.ac.id](mailto:your-email@students.usu.ac.id)

---

⭐ **Star this repo jika bermanfaat!**
