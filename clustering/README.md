# Lampiran Eksperimen — Semantic Clustering

Folder ini berisi **salinan (copy)** seluruh data & hasil pengujian untuk eksperimen perbandingan
**3 dataset × 3 algoritma clustering** (total 9 pengujian), ditambah dataset pendukung.
Semua file adalah salinan dari `main\data\...` dan `main\evaluation\...`; file asli tidak diubah.

- Embedding: **Qwen3-Embedding-8B (4096D)** via OpenRouter (untuk seluruh 9 pengujian inti).
- Panel evaluasi (LLM-as-a-Judge): Gemini 3 Flash, Claude Sonnet 4.6, GLM-5 / GLM-5-turbo.
- Dokumen analisis terkait: `documents\results_analysis\20260410_MultiDataset_Experiment_Analysis.md`
  dan `documents\results_analysis\20260312_Hadoop_WTK_Experiment_Analysis.md`.

---

## 1. Struktur Folder

```
Lampiran_Eksperimen\
├── README.md                  ← file ini
├── datasets\                  ← dataset mentah (raw) + hasil cleaning + embedding
│   ├── antarmuka\
│   ├── ldm\
│   ├── hadoop\
│   ├── kwl_data_amplified_81\ ← dataset 81 kueri (lihat catatan §4)
│   └── stackoverflow\         ← Questions_Formatted.csv (lihat catatan §4)
└── experiments\               ← hasil 3×3 pengujian
    ├── antarmuka\{agglomerative,kmeans,hdbscan}\
    ├── ldm\{agglomerative,kmeans,hdbscan}\
    └── hadoop\{agglomerative,kmeans,hdbscan}\
```

Tiap folder algoritma berisi:
- `clusters\`    — `clusters.csv` (penugasan klaster per kueri), `metrics.json` (metrik geometris),
                   `tuning_log.json`, `run_config.json`.
- `extraction\`  — `extraction_results.json` / `.csv` (label topik + kueri representatif), `run_config.json`.
- `eval_*\`      — `cluster_eval.json` (metrik geometris hasil evaluasi), `extraction_eval.json`
                   (penilaian LLM), `eval_summary.json` (ringkasan + skor agreement).

---

## 2. Provenance Dataset (raw → cleaned)

| Dataset | Topik / `wtk_ref_id` | File mentah (raw) | baris | File cleaned (dipakai pipeline) | N |
|---|---|---|---|---|---|
| **Antarmuka** | Antarmuka (UI/UX), id 21 | `datasets\antarmuka\raw_antarmuka_id21.csv` | 71 | `datasets\antarmuka\cleaned_antarmuka_id21_N71.csv` | **71** |
| **LDM** | Logical Data Model, id 23 | `datasets\ldm\raw_ldm_id23.csv` | 64 | `datasets\ldm\cleaned_ldm_id23_N62.csv` | **62** |
| **Hadoop** | Ekosistem Hadoop, id 29 | `datasets\hadoop\raw_hadoop_id29.csv` | 117 | `datasets\hadoop\cleaned_hadoop_id29_N115.csv` | **115** |
| **KWL Data Amplified** | Database/Query (81 kueri) | `datasets\kwl_data_amplified_81\raw_kwl_data_amplified_N90.csv` | 90 | `datasets\kwl_data_amplified_81\cleaned_kwl_data_amplified_N81.csv` | **81** |

- Skema file **raw WTK** (antarmuka/ldm/hadoop): `id, created_at, reflection, student_id, wtk_ref_id`.
- Skema file **cleaned**: `UserID, Anonymized_User_ID, Cleaned_Content`.
- Skema **raw KWL**: `UserID, WantToKnow`.
- File `*.parquet` di tiap folder dataset = embedding vektor (Qwen 4096D) untuk dataset tsb.
  Folder `kwl_data_amplified_81\` berisi 3 embedding (satu per model: Qwen, Indo-S-BERT, BGE-M3),
  semuanya bertanggal `20260226` — varian yang dipakai pada eksperimen pembanding 81-kueri (H19N24).

---

## 3. Pemetaan 9 Pengujian (Run IDs)

| Dataset | Algoritma | Eval ID | Cluster run | Extraction run |
|---|---|---|---|---|
| Antarmuka | Agglomerative | `20260409_180955` | `20260409_171851` | `20260409_171851` |
| Antarmuka | K-Means | `20260409_181433` | `20260409_173328` | `20260409_173328` |
| Antarmuka | HDBSCAN | `20260409_183312` | `20260409_173525` | `20260409_173525` |
| LDM | Agglomerative | `20260409_231236` | `20260409_203516` | `20260409_203516` |
| LDM | K-Means | `20260409_232057` | `20260409_203810` | `20260409_203810` |
| LDM | HDBSCAN | `20260409_233935` | `20260409_211423` | `20260409_211423` |
| Hadoop | Agglomerative | `20260305_181907` (W1) / `20260309_182328` (W2) | `20260305_164840` | `20260305_164840` |
| Hadoop | HDBSCAN | `20260305_190752` (W1) / `20260309_182946` (W2) | `20260305_185730` | `20260305_185730` |
| Hadoop | K-Means | `20260305_191817` (W1) / `20260309_183418` (W2) | `20260305_191240` | `20260305_191240` |

**Catatan Hadoop W1 vs W2:** Pengujian Hadoop dievaluasi dua kali atas **output clustering yang identik**.
W1 (`20260305_*`) menghasilkan `cluster_eval.json` (metrik geometris) + penilaian LLM awal.
W2 (`20260309_*`) hanya **mengulang evaluasi semantik LLM** (skor kualitas final yang dipakai di dokumen
analisis: Agglo 4.64, HDBSCAN 4.29, K-Means 4.06) pada clustering yang sama — sehingga metrik geometrisnya
diwarisi dari W1. Karena itu folder `eval_W2_*` tidak memuat `cluster_eval.json`.

---

## 4. ⚠️ Catatan Penting tentang "Dataset Stack Overflow"

Dua hal yang **harus** diperhatikan sebelum dikutip di skripsi:

1. **Dataset 81 kueri = "KWL Data Amplified", bukan dump mentah Stack Overflow.**
   File `cleaned_kwl_data_amplified_N81.csv` berisi 81 pertanyaan bertema DBMS/query-optimization
   (campuran Bahasa Indonesia + Inggris, UserID berpola sintetis `U001…U081`). Ini adalah set
   *amplified* (kemungkinan hasil augmentasi/sintesis), **belum terbukti** berasal langsung dari Stack Overflow.
   (Catatan: `cleaned_forum_data.csv` di repo asli identik byte-per-byte dengan file ini.)

2. **File asli berformat Stack Overflow** ada di `datasets\stackoverflow\Questions_Formatted_N1000.csv`
   (1000 baris; kolom `Id, OwnerUserId, CreationDate, Score, Title, Body` — format dump StackExchange).
   File ini **tidak digunakan** pada 9 pengujian inti maupun pada eksperimen 81-kueri. Disertakan di sini
   hanya karena diminta. Pastikan asal-usulnya sebelum melabelinya sebagai sumber dataset eksperimen.

3. **Metadata `run_config.json` sudah dinormalisasi pada salinan lampiran ini** (lihat §6). Pada file asli
   di `main\`, field `input_csv`/`cleaned_csv` Hadoop sempat tertulis `kwl_data_amplified.csv` (basi/template
   tidak ter-update); sumber Hadoop yang benar adalah **`wtk_ref_id = 29`** — diverifikasi dari
   `embeddings_parquet` (`embeddings_29_...`) dan isi `clusters.csv` (115 kueri bertema "Ekosistem Hadoop").
   Path tersebut **sudah dikoreksi** di salinan lampiran.

---

## 5. MANIFEST.csv

`MANIFEST.csv` mendaftar seluruh file di folder ini beserta kategori, ukuran, dan jumlah baris (untuk CSV).
Kolom: `RelativePath, Category, Ext, SizeBytes, SizeKB, Rows`. Kategori: `dataset_raw`, `dataset_cleaned`,
`dataset_embedding`, `dataset_stackoverflow`, `cluster_output`, `extraction_output`, `evaluation`, `readme`.

