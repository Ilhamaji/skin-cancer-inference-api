<div align="center">

# Skin Cancer Inference API

REST API klasifikasi gambar berbasis **Hapi.js + TensorFlow.js** — prediksi kanker kulit dari gambar, dideploy ke Google Cloud.

[![Status](https://img.shields.io/badge/status-complete-brightgreen?style=flat-square)](#)
[![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org)
[![Hapi](https://img.shields.io/badge/Hapi.js-21-FFB300?style=flat-square)](https://hapi.dev)
[![TensorFlow](https://img.shields.io/badge/TensorFlow.js-3-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)](https://www.tensorflow.org/js)

</div>

---

## Tentang

Submission kelas **Belajar Penerapan Machine Learning dengan Google Cloud** (Dicoding). Model klasifikasi gambar di-serve sebagai REST API, dengan hasil prediksi disimpan ke **Firestore** agar bisa ditelusuri kembali.

Model membedakan dua kelas — **Cancer** dan **Non-cancer** — dengan ambang keyakinan 50%: di bawah ambang tersebut, hasil dipaksa menjadi *Non-cancer* untuk menghindari false positive.

## Endpoint

| Method | Path | Deskripsi |
| --- | --- | --- |
| `POST` | `/predict` | Upload gambar (`multipart/form-data`, maks 1 MB) → label + confidence + saran. |
| `GET` | `/predict/histories` | Riwayat seluruh prediksi yang tersimpan di Firestore. |

### Contoh

```bash
curl -X POST http://localhost:8080/predict \
  -F "image=@gambar.jpg"
```

```json
{
  "status": "success",
  "message": "Model is predicted successfully",
  "data": {
    "id": "abc123",
    "result": "Non-cancer",
    "suggestion": "Penyakit kanker tidak terdeteksi.",
    "confidenceScore": 0.97,
    "createdAt": "2026-01-01T00:00:00.000Z"
  }
}
```

## Cara Kerja

```
Gambar JPEG
    │
    ▼
decodeJpeg ──► resizeNearestNeighbor([224, 224]) ──► expandDims ──► toFloat
    │
    ▼
model.predict()  ──►  confidenceScore = max(score) * 100
    │
    ▼
label = argMax(score)                     classes = ["Cancer", "Non-cancer"]
    │
    ▼
if confidenceScore < 50  ──►  label = "Non-cancer"
    │
    ▼
Simpan ke Firestore (id via nanoid) ──► response JSON
```

## Tech Stack

| Bagian | Teknologi |
| --- | --- |
| Runtime | Node.js |
| Framework | Hapi.js 21 + `@hapi/inert` |
| Model | TensorFlow.js Node 3.21.1 |
| Database | Google Cloud Firestore |
| Config | `dotenv` |
| ID | `nanoid` |

## Menjalankan

### 1. Clone & install

```bash
git clone https://github.com/Ilhamaji/skin-cancer-inference-api.git
cd skin-cancer-inference-api
npm install
```

### 2. Environment

```env
HOST=0.0.0.0
PORT=8080
MODEL_URL=https://storage.googleapis.com/bucket-anda/model/
```

### 3. Kredensial Google Cloud

Unduh service account key dan set:

```bash
export GOOGLE_APPLICATION_CREDENTIALS="./serviceaccountkey.json"
```

### 4. Jalankan

```bash
npm start
# atau mode development:
npm run dev
```

Server berjalan di `http://localhost:8080`.

## Struktur

```
index.js                        # bootstrap Hapi server
src/
  routes.js                     # definisi route /predict
  handlers.js                   # handler predict & histories
  services/
    loadModel.js                # memuat model TF dari Google Cloud Storage
    inferenceService.js         # preprocessing, prediksi, penyimpanan
    storeData.js                # operasi Firestore
  Exception/inputError.js       # error class untuk input tidak valid
```

## Catatan Penting

> ⚠️ Project ini dibuat untuk **tujuan edukasi**, bukan alat diagnosis medis. Jangan gunakan hasil prediksi sebagai dasar keputusan klinis.

- `@tensorflow/tfjs-node` dipin ke versi **3.21.1** — versi lain punya masalah kompatibilitas native binding.
- Ukuran upload dibatasi 1 MB; gambar yang lebih besar akan ditolak.

## Lisensi

ISC.

---

<div align="center">
Dibuat oleh <a href="https://github.com/Ilhamaji">Ilham Aji</a>
</div>
