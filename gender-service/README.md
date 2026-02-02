# Gender Verification AI Microservice

A **privacy-first, stateless** microservice for gender verification using live camera images. Built for anonymous chat platforms implementing controlled anonymity to prevent catfishing and fake accounts.

## 🔒 Privacy Guarantees

This service is designed with privacy as the core principle:

| Guarantee | Implementation |
|-----------|----------------|
| **No Image Persistence** | Images are processed entirely in-memory using `io.BytesIO` |
| **Immediate Cleanup** | Image bytes and numpy arrays are explicitly deleted in `finally` block after inference |
| **No Logging of Image Data** | Only metadata (success/error status) is logged, never image content |
| **No Database** | Completely stateless - no storage of any kind |
| **No Authentication Data** | No user identification or tracking |
| **Memory-Only Processing** | Uses PIL and numpy for in-memory image processing |

### How Privacy is Enforced in Code

```python
finally:
    # CRITICAL: Clean up image data immediately
    if image_bytes is not None:
        del image_bytes
    if image_array is not None:
        del image_array
    await image.close()
```

## 🚀 Quick Start

### Local Development

```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the server
uvicorn main:app --reload --port 8000
```

### Docker

```bash
# Build the image
docker build -t gender-verification .

# Run the container
docker run -p 8000:8000 gender-verification
```

## 📡 API Reference

### POST /verify-gender

Verify gender from a live camera image.

**Request:**
- Method: `POST`
- Content-Type: `multipart/form-data`
- Body: Single image file (field name: `image`)

**Supported Formats:**
- JPEG (.jpg, .jpeg)
- PNG (.png)
- WebP (.webp)

**Success Response:**
```json
{
  "verified": true,
  "gender": "M"
}
```

**Error Responses:**
```json
{
  "verified": false,
  "error": "Face not detected"
}
```

```json
{
  "verified": false,
  "error": "Multiple faces detected. Please ensure only one face is visible."
}
```

```json
{
  "verified": false,
  "error": "Invalid file type. Allowed types: image/jpeg, image/png, image/webp"
}
```

### GET /health

Health check endpoint for container orchestration.

```json
{
  "status": "healthy",
  "service": "gender-verification"
}
```

### GET /

Service information.

```json
{
  "service": "Gender Verification API",
  "version": "1.0.0",
  "privacy": "No images are stored. All processing is done in-memory.",
  "endpoint": "POST /verify-gender"
}
```

## 🧪 Testing

### Using cURL

```bash
# Test with a local image
curl -X POST "http://localhost:8000/verify-gender" \
  -H "accept: application/json" \
  -H "Content-Type: multipart/form-data" \
  -F "image=@/path/to/face.jpg"

# Health check
curl http://localhost:8000/health
```

### Using Python

```python
import requests

url = "http://localhost:8000/verify-gender"

with open("face.jpg", "rb") as f:
    response = requests.post(url, files={"image": f})
    print(response.json())
```

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    FastAPI Application                       │
├─────────────────────────────────────────────────────────────┤
│  POST /verify-gender                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ Validate    │→ │ Detect Face │→ │ Classify Gender     │  │
│  │ File Type   │  │ (OpenCV DNN)│  │ (Caffe Model)       │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│          ↓                ↓                    ↓             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              FINALLY: Delete All Image Data            │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 🔧 Technology Stack

- **Framework:** FastAPI (async, high-performance)
- **Face Detection:** OpenCV DNN (SSD-based detector)
- **Gender Classification:** Caffe model (pre-trained CNN)
- **Image Processing:** Pillow, NumPy, OpenCV
- **Server:** Uvicorn (ASGI)

## ⚠️ Edge Cases Handled

| Case | Response |
|------|----------|
| No face detected | `{"verified": false, "error": "Face not detected"}` |
| Multiple faces | `{"verified": false, "error": "Multiple faces detected..."}` |
| Invalid file type | HTTP 400 with error message |
| Empty file | HTTP 400 with error message |
| Corrupted image | HTTP 400 with error message |

## 📋 Models Used

The service automatically downloads the following models on first run:

1. **Face Detection**: `res10_300x300_ssd_iter_140000.caffemodel` (~10MB)
   - SSD-based face detector from OpenCV
   
2. **Gender Classification**: `gender_net.caffemodel` (~44MB)
   - Caffe model trained for gender classification

## 🔐 Security Considerations

1. **No persistent storage** - Images cannot be retrieved after processing
2. **No external logging** - Image data is never sent to external services
3. **Non-root container** - Docker runs as unprivileged user
4. **Minimal dependencies** - Reduces attack surface

## 📄 License

MIT License
