# Upload API Documentation

This document provides comprehensive curl examples for all three upload API endpoints.

## Base URLs

- **Production:** `https://dev-api.autorender.io`
- **Local Development:** `http://localhost:8080`

## Authentication

All endpoints (except token-based upload) require API key authentication. You can provide your API key in two ways:

1. **Header:** `x-api-key: YOUR_API_KEY`
2. **Authorization:** `Authorization: Bearer YOUR_API_KEY`

---

## 1. Generate Upload Token

Generate a short-lived token for client-side uploads. The token contains workspace, user, and upload configuration.

**Endpoint:** `POST /api/v1/generate-token`

**Authentication:** Required (API Key)

### Request Body Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file_name` | string | ✅ Yes | File name for the uploaded file (e.g., `avatar.jpg`) |
| `folder` | string | ❌ No | Folder path where the file will be stored (e.g., `user-uploads/avatars/`) |
| `tags` | string[] | ❌ No | Array of tags to attach to the file (e.g., `["user-upload", "avatar"]`) |
| `transform` | string | ❌ No | Image transformation string (e.g., `w_400,h_400,fit_cover,q_80`) |
| `max_file_size` | number | ❌ No | Maximum file size in bytes (default: 5MB) |
| `ttl_seconds` | number | ❌ No | Token expiration time in seconds (default: 300 seconds / 5 minutes) |
| `metadata` | object | ❌ No | Custom metadata object (e.g., `{"key": "value"}`) |
| `custom_id` | string | ❌ No | Custom identifier for the file |
| `random_prefix` | boolean | ❌ No | If `true`, adds a random suffix to the filename |
| `allow_override` | object | ❌ No | Allow client to override certain fields during upload |
| `allow_override.folder` | boolean | ❌ No | Allow client to override folder path |
| `allow_override.tags` | boolean | ❌ No | Allow client to override tags |
| `allow_override.transform` | boolean | ❌ No | Allow client to override transformation string |

### Response

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "signature": "abc123...",
  "expire_at": 1234567890,
  "public_key": "pub_WS123",
  "workspace_id": "WS123",
  "policy": {
    "folder": "user-uploads/avatars/",
    "tags": ["user-upload", "avatar"],
    "max_file_size": 5242880,
    "allow_override": {
      "folder": false,
      "tags": true,
      "transform": false
    }
  }
}
```

### Curl Examples

#### Minimal Request (Required Fields Only)

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/generate-token \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "file_name": "my-image.jpg"
  }'
```

#### Full Request (All Fields)

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/generate-token \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "file_name": "avatar.jpg",
    "folder": "user-uploads/avatars",
    "tags": ["user-upload", "avatar", "profile"],
    "transform": "w_400,h_400,fit_cover,q_80",
    "max_file_size": 10485760,
    "ttl_seconds": 3600,
    "metadata": {
      "user_id": "12345",
      "category": "profile"
    },
    "custom_id": "user-avatar-12345",
    "random_prefix": true,
    "allow_override": {
      "folder": false,
      "tags": true,
      "transform": false
    }
  }'
```

#### Using Authorization Bearer Header

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/generate-token \
  -H "Authorization: Bearer YOUR_API_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "file_name": "document.pdf",
    "folder": "documents",
    "ttl_seconds": 7200
  }'
```

---

## 2. Upload File with Token

Upload a file using a token generated from the previous endpoint. No API key required - authentication is handled by the token.

**Endpoint:** `POST /api/v1/uploads/{token}`

**Authentication:** Token-based (token in URL path)

### URL Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | string | ✅ Yes | The upload token from step 1 (in URL path) |

### Request Headers

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| `Content-Type` | string | ✅ Yes | MIME type of the file (e.g., `image/jpeg`, `image/png`, `application/octet-stream`) |
| `Content-Length` | number | ❌ No | File size in bytes (optional, but recommended) |

### Request Body

The request body should be the raw binary file data.

### Response

```json
{
  "success": true,
  "data": {
    "id": "file-id",
    "file_no": "FILE123",
    "name": "avatar.jpg",
    "url": "https://...",
    "path": "user-uploads/avatars",
    "width": 400,
    "height": 400,
    "format": "jpg",
    "file_size": 102400,
    "workspace_no": "WS123"
  }
}
```

### Curl Examples

#### Upload JPEG Image

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/uploads/YOUR_TOKEN_HERE \
  -H "Content-Type: image/jpeg" \
  --data-binary @/path/to/your/image.jpg
```

#### Upload PNG Image

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/uploads/YOUR_TOKEN_HERE \
  -H "Content-Type: image/png" \
  --data-binary @/path/to/your/image.png
```

#### Upload with Content-Length Header

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/uploads/YOUR_TOKEN_HERE \
  -H "Content-Type: image/jpeg" \
  -H "Content-Length: $(stat -f%z /path/to/your/image.jpg)" \
  --data-binary @/path/to/your/image.jpg
```

#### Upload WebP Image

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/uploads/YOUR_TOKEN_HERE \
  -H "Content-Type: image/webp" \
  --data-binary @/path/to/your/image.webp
```

#### Upload Generic Binary File

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/uploads/YOUR_TOKEN_HERE \
  -H "Content-Type: application/octet-stream" \
  --data-binary @/path/to/your/file.bin
```

**Note:** Replace `YOUR_TOKEN_HERE` with the actual token value from the generate-token response.

---

## 3. Direct File Upload

Upload a file directly using API key authentication. This is the simplest method for server-side uploads.

**Endpoint:** `POST /api/v1/uploads`

**Authentication:** Required (API Key)

### Request Headers

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| `x-api-key` | string | ✅ Yes | Your API key (alternative to Authorization header) |
| `Authorization` | string | ❌ No | Bearer token with API key (alternative to x-api-key header) |
| `Content-Type` | string | ✅ Yes | Must be `multipart/form-data` (curl automatically sets this with `-F` flag) |

**Note:** When using curl with the `-F` flag, the `Content-Type: multipart/form-data` header is automatically set. You don't need to specify it manually.

### Request Body (Multipart Form Data)

**Important:** Validation is handled server-side in the controller. The `file_name` field is required and must be provided in the form data.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | file | ✅ Yes | The file to upload (binary data) |
| `file_name` | string | ✅ Yes | File name for the uploaded file (e.g., `my-image.jpg`) |
| `folder` | string | ❌ No | Folder path where the file will be stored (e.g., `uploads/my-folder`) |
| `tags` | string | ❌ No | Comma-separated tags (e.g., `tag1,tag2,tag3`) |
| `transform` | string | ❌ No | Image transformation string (e.g., `w_800,h_600,q_90`) - same as generate-token API |
| `metadata` | string | ❌ No | JSON string for custom metadata (e.g., `{"key": "value"}`) |
| `custom_id` | string | ❌ No | Custom identifier for the file |
| `random_prefix` | string | ❌ No | Set to `"true"` to add a random suffix to filename |

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

### Curl Examples

#### Minimal Request (Required Fields Only)

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/uploads \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -F "file=@/path/to/your/image.jpg" \
  -F "file_name=my-image.jpg"
```

**Note:** The `-F` flag automatically sets `Content-Type: multipart/form-data`. Both `file` and `file_name` are required fields.

#### Full Request (All Fields)

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/uploads \
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

#### Using Authorization Bearer Header

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/uploads \
  -H "Authorization: Bearer YOUR_API_KEY_HERE" \
  -F "file=@/path/to/your/image.jpg" \
  -F "file_name=document.jpg" \
  -F "folder=documents" \
  -F "transform=w_1200,h_800,q_85"
```

#### Upload with Image Transformations

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/uploads \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -F "file=@/path/to/your/image.jpg" \
  -F "file_name=thumbnail.jpg" \
  -F "folder=processed-images" \
  -F "transform=w_500,h_500,fit_cover,q_80" \
  -F "tags=processed,thumbnail"
```

#### Upload with Custom Metadata

```bash
curl -X POST \
  https://dev-api.autorender.io/api/v1/uploads \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -F "file=@/path/to/your/image.jpg" \
  -F "file_name=avatar.jpg" \
  -F "folder=user-uploads" \
  -F "metadata={\"user_id\":\"12345\",\"upload_source\":\"mobile_app\",\"timestamp\":\"2025-01-19T10:00:00Z\"}" \
  -F "custom_id=user-12345-avatar"
```

#### Local Development Example

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

## Complete Workflow Example

Here's a complete example showing all three endpoints in sequence:

```bash
# Step 1: Generate upload token
TOKEN_RESPONSE=$(curl -s -X POST \
  https://dev-api.autorender.io/api/v1/generate-token \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "file_name": "test-image.jpg",
    "folder": "uploads",
    "tags": ["test", "example"],
    "transform": "w_800,h_600,q_90",
    "ttl_seconds": 3600
  }')

# Extract token from response (requires jq)
TOKEN=$(echo $TOKEN_RESPONSE | jq -r '.token')

echo "Generated token: $TOKEN"

# Step 2: Upload file with token
UPLOAD_RESPONSE=$(curl -s -X POST \
  https://dev-api.autorender.io/api/v1/uploads/$TOKEN \
  -H "Content-Type: image/jpeg" \
  --data-binary @/path/to/your/image.jpg)

echo "Upload response: $UPLOAD_RESPONSE"

# Step 3: Alternative - Direct upload (no token needed)
DIRECT_UPLOAD_RESPONSE=$(curl -s -X POST \
  https://dev-api.autorender.io/api/v1/uploads \
  -H "x-api-key: YOUR_API_KEY_HERE" \
  -F "file=@/path/to/your/image.jpg" \
  -F "file_name=test-image.jpg" \
  -F "folder=uploads" \
  -F "transform=w_800,h_600,q_90")

echo "Direct upload response: $DIRECT_UPLOAD_RESPONSE"
```

---

## Error Responses

All endpoints may return the following error responses:

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

1. **Token Expiration:** Tokens expire after the TTL specified (default: 5 minutes). Generate a new token if it expires.

2. **File Size Limits:** 
   - Default max file size for token uploads: 5MB
   - Can be customized via `max_file_size` in token generation
   - Direct uploads: 100MB limit

3. **Image Transformations:** 
   - Only applies to image files
   - Use `transform` field (same field name in both generate-token and direct upload APIs)
   - Format: `w_{width},h_{height},q_{quality}`
   - Example: `w_800,h_600,q_90`

4. **Folder Paths:** 
   - Leading and trailing slashes are automatically removed
   - Nested folders are supported (e.g., `uploads/subfolder/nested`)

5. **Content Types:** 
   - Token uploads accept: `image/*` and `application/octet-stream`
   - Direct uploads use `multipart/form-data` (validation handled in controller, not via JSON schema)

6. **Direct Upload Validation:**
   - The direct upload endpoint uses `multipart/form-data`, which is validated server-side in the controller
   - Required fields: `file` (the actual file) and `file_name` (string)
   - All other fields are optional
   - If `file_name` is missing or empty, the request will return a 400 error

---

## Local Development

For local development, replace the base URL:

```bash
# Local development URL
http://localhost:8080/api/v1/generate-token
http://localhost:8080/api/v1/uploads/{token}
http://localhost:8080/api/v1/uploads
```

