# Upload API Documentation

Curl-oriented reference for AutoRender upload endpoints: **multipart upload** (large files in parts) and **direct upload** (single multipart/form request).

## Base URLs

- **Production:** `https://upload.autorender.io`
- **Local development:** use your app URL (for example `http://localhost:3000` or `http://localhost:8080`)

## Authentication

Endpoints on AutoRender require API key authentication:

1. **Header:** `x-api-key: YOUR_API_KEY`
2. **Authorization:** `Authorization: Bearer YOUR_API_KEY`

Presigned URLs returned by multipart **start** are called with `PUT` only; do not send your API key to those URLs.

---

## 1. Multipart upload (large files)

Three steps: **start** (JSON + API key) → **upload parts** (presigned `PUT` to storage) → **complete** (JSON + API key).

### 1a. Start session

**Endpoint:** `POST /api/v1/multipart/start`

#### Request body (JSON)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file_name` | string | Yes | Original file name |
| `size` | number | Yes | Total size in bytes |
| `format` | string | Yes | MIME type (e.g. `video/mp4`) |
| `folder` | string | No | Workspace folder path |
| `tags` | string[] | No | Tags |
| `metadata` | object | No | Custom metadata |
| `custom_id` | string | No | Your tracking id |
| `random_prefix` | boolean | No | Random prefix on stored name |
| `ttl_seconds` | number | No | Presigned URL lifetime |

#### Response

```json
{
  "session_id": "<MULTIPART_SESSION_UUID>",
  "part_size": 10485760,
  "parts": ["<PRESIGNED_URL_1>", "<PRESIGNED_URL_2>"]
}
```

#### Curl example

```bash
curl -X POST "https://upload.autorender.io/api/v1/multipart/start" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file_name": "big-video.mp4",
    "size": 73400320,
    "format": "video/mp4",
    "folder": "uploads/videos",
    "tags": ["campaign", "raw"],
    "metadata": { "source": "mobile-app" },
    "custom_id": "vid_001",
    "random_prefix": false,
    "ttl_seconds": 3600
  }'
```

### 1b. Upload parts (presigned PUT)

- Split the file into chunks of **`part_size`** bytes (last chunk may be smaller).
- For each chunk index `i`, `PUT` raw bytes to `parts[i]` with `Content-Type: application/octet-stream`.

Example after saving the start response as `start-response.json`:

```bash
PART_SIZE=$(jq -r '.part_size' start-response.json)
split -b "$PART_SIZE" "./big-video.mp4" /tmp/mpart_

i=0
for chunk in /tmp/mpart_*; do
  url=$(jq -r ".parts[$i]" start-response.json)
  curl -X PUT "$url" \
    -H "Content-Type: application/octet-stream" \
    --data-binary @"$chunk"
  i=$((i + 1))
done
```

### 1c. Complete session

**Endpoint:** `POST /api/v1/multipart/complete`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `session_id` | string | Yes | From start response |

```bash
curl -X POST "https://upload.autorender.io/api/v1/multipart/complete" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "<MULTIPART_SESSION_UUID>"
  }'
```

---

## 2. Direct file upload

Upload a file in one request using API key authentication (multipart form data).

**Endpoint:** `POST /api/v1/uploads`

### Request headers

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| `x-api-key` | string | Yes* | API key (*or use Bearer below) |
| `Authorization` | string | Yes* | `Bearer YOUR_API_KEY` |
| `Content-Type` | string | Yes | `multipart/form-data` (set automatically by curl `-F`) |

### Form fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | file | Yes | File binary |
| `file_name` | string | Yes | Final name |
| `folder` | string | No | Folder path |
| `tags` | string | No | Comma-separated tags |
| `transform` | string | No | Image transform string (e.g. `w_800,h_600,q_90`) |
| `metadata` | string | No | JSON string |
| `custom_id` | string | No | Custom id |
| `random_prefix` | string | No | `"true"` for random suffix |

### Response

```json
{
  "success": true,
  "data": {
    "id": "file-id",
    "file_no": "FILE123",
    "name": "my-image.jpg",
    "url": "https://...",
    "path": "uploads",
    "width": 1920,
    "height": 1080,
    "format": "jpg",
    "file_size": 1024000,
    "workspace_no": "WS123"
  }
}
```

### Curl examples

#### Minimal

```bash
curl -X POST \
  https://upload.autorender.io/api/v1/uploads \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -F "file=@/path/to/your/image.jpg" \
  -F "file_name=my-image.jpg"
```

#### Full

```bash
curl -X POST \
  https://upload.autorender.io/api/v1/uploads \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -F "file=@/path/to/your/image.jpg" \
  -F "file_name=my-image.jpg" \
  -F "folder=uploads/my-folder" \
  -F "tags=tag1,tag2,tag3" \
  -F "transform=w_800,h_600,q_90" \
  -F "metadata={\"user_id\":\"12345\",\"category\":\"profile\"}" \
  -F "custom_id=my-custom-id-123" \
  -F "random_prefix=true"
```

#### Bearer header

```bash
curl -X POST \
  https://upload.autorender.io/api/v1/uploads \
  -H "Authorization: Bearer YOUR_API_KEY_HERE" \
  -F "file=@/path/to/your/image.jpg" \
  -F "file_name=document.jpg" \
  -F "folder=documents" \
  -F "transform=w_1200,h_800,q_85"
```

#### Local example

```bash
curl -X POST \
  http://localhost:8080/api/v1/uploads \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -F "file=@/path/to/your/image.jpg" \
  -F "file_name=test.jpg" \
  -F "folder=uploads" \
  -F "transform=w_500,h_500,q_80"
```

---

## Workflow example (multipart + direct)

```bash
# Multipart: start → upload parts (see sections 1a–1b) → complete
curl -s -X POST "https://upload.autorender.io/api/v1/multipart/complete" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"session_id":"<MULTIPART_SESSION_UUID>"}'

# Direct upload (single request)
curl -s -X POST \
  https://upload.autorender.io/api/v1/uploads \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -F "file=@/path/to/your/image.jpg" \
  -F "file_name=test-image.jpg" \
  -F "folder=uploads" \
  -F "transform=w_800,h_600,q_90"
```

---

## Error responses

### 400 Bad Request

```json
{
  "success": false,
  "error": "Validation error message"
}
```

### 401 Unauthorized

```json
{
  "success": false,
  "error": "Unauthorized",
  "message": "Invalid or missing API key"
}
```

### 500 Internal Server Error

```json
{
  "success": false,
  "error": "Upload failed",
  "message": "Detailed error message"
}
```

---

## Notes

1. **Multipart:** Total bytes uploaded across parts must match **`size`** from start. Parts must match **`part_size`** ordering from the API. If presigned URLs expire, start a new session.

2. **File size limits:** Direct uploads are bounded by server policy (historically on the order of 100MB for form posts); use multipart for larger assets.

3. **Image transformations:** Apply to image files via **`transform`** on direct upload (e.g. `w_800,h_600,q_90`).

4. **Folder paths:** Leading and trailing slashes are normalized; nested paths are supported.

5. **Direct upload validation:** Requires **`file`** and **`file_name`** in form data; other fields are optional.

---

## Local development paths

```text
POST http://localhost:3000/api/v1/multipart/start
POST http://localhost:3000/api/v1/multipart/complete
POST http://localhost:3000/api/v1/uploads
```

(Adjust host and port to match your API process.)
