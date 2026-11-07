<div align="center">

# Skin Cancer Inference API

An image-classification REST API built on **Hapi.js + TensorFlow.js** — predicts skin cancer from a photo, deployed to Google Cloud.

[![Status](https://img.shields.io/badge/status-complete-brightgreen?style=flat-square)](#)
[![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org)
[![Hapi](https://img.shields.io/badge/Hapi.js-21-FFB300?style=flat-square)](https://hapi.dev)
[![TensorFlow](https://img.shields.io/badge/TensorFlow.js-3-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)](https://www.tensorflow.org/js)

**English** · [Bahasa Indonesia](README.id.md)

</div>

---

## About

A submission for Dicoding's **Belajar Penerapan Machine Learning dengan Google Cloud** course. An image-classification model is served as a REST API, and each prediction is stored in **Firestore** so it can be reviewed later.

The model separates two classes — **Cancer** and **Non-cancer** — with a 50% confidence threshold. Below that threshold the result is forced to *Non-cancer* to avoid false positives.

## Endpoints

| Method | Path | Description |
| --- | --- | --- |
| `POST` | `/predict` | Upload an image (`multipart/form-data`, max 1 MB) → label, confidence, and suggestion. |
| `GET` | `/predict/histories` | Every prediction stored in Firestore. |

### Example

```bash
curl -X POST http://localhost:8080/predict \
  -F "image=@photo.jpg"
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

> The `suggestion` field is returned in Indonesian, matching the original coursework requirements.

## How It Works

```
JPEG image
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
Store to Firestore (id via nanoid) ──► JSON response
```

## Tech Stack

| Layer | Technology |
| --- | --- |
| Runtime | Node.js |
| Framework | Hapi.js 21 + `@hapi/inert` |
| Model | TensorFlow.js Node 3.21.1 |
| Database | Google Cloud Firestore |
| Config | `dotenv` |
| IDs | `nanoid` |

## Getting Started

### 1. Clone and install

```bash
git clone https://github.com/Ilhamaji/skin-cancer-inference-api.git
cd skin-cancer-inference-api
npm install
```

### 2. Environment

```env
HOST=0.0.0.0
PORT=8080
MODEL_URL=https://storage.googleapis.com/your-bucket/model/
```

### 3. Google Cloud credentials

Download a service account key and point the SDK at it:

```bash
export GOOGLE_APPLICATION_CREDENTIALS="./serviceaccountkey.json"
```

### 4. Run

```bash
npm start
# or in development mode:
npm run dev
```

The server listens on `http://localhost:8080`.

## Project Layout

```
index.js                        # Hapi server bootstrap
src/
  routes.js                     # /predict route definitions
  handlers.js                   # predict and histories handlers
  services/
    loadModel.js                # loads the TF model from Google Cloud Storage
    inferenceService.js         # preprocessing, prediction, persistence
    storeData.js                # Firestore operations
  Exception/inputError.js       # error class for invalid input
```

## Important Notes

> ⚠️ This project is for **educational purposes only**, not a medical diagnostic tool. Do not base clinical decisions on its output.

- `@tensorflow/tfjs-node` is pinned to **3.21.1** — other versions have native binding compatibility issues.
- Uploads are capped at 1 MB; larger images are rejected.

## License

ISC.

---

<div align="center">
Built by <a href="https://github.com/Ilhamaji">Ilham Aji</a>
</div>
