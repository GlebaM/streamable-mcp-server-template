# UploadThing REST API (Quick Reference)

Source docs:

- [OpenAPI docs page](https://docs.uploadthing.com/api-reference/openapi-spec)
- [OpenAPI JSON spec](https://api.uploadthing.com/openapi-spec.json)

## API Basics

- API title/version: `UploadThing REST API` / `6.10.0`
- Base URL: `https://api.uploadthing.com`
- Auth: API key in header `x-uploadthing-api-key`
- Global security: all endpoints require `ApiKeyAuth`
- Common error statuses: `400`, `401`, `500` (some endpoints also use `403` or `404`)

## Core Endpoints

### Upload lifecycle

- `POST /v6/prepareUpload`
  - Request fields: `callbackUrl`, `callbackSlug`, `files`, `routeConfig`, `metadata`
  - Required: `callbackUrl`, `callbackSlug`, `files`, `routeConfig`
  - Response: array of upload targets (either `PresignedPostURLs` or `MultipartUploadURLs`)

- `POST /v7/prepareUpload`
  - Request fields: `fileName`, `fileSize`, `slug`, `fileType`, `customId`,
    `contentDisposition`, `acl`, `expiresIn`
  - Required: `fileName`, `fileSize`
  - Enums:
    - `contentDisposition`: `inline` | `attachment`
    - `acl`: `public-read` | `private`
  - Response: `{ key, url }`

- `POST /v6/uploadFiles`
  - Request fields: `files`, `acl`, `metadata`, `contentDisposition`
  - Required: `files`
  - Enums:
    - `acl`: `public-read` | `private`
    - `contentDisposition`: `inline` | `attachment`

- `POST /v6/completeMultipart`
  - Request fields: `fileKey`, `uploadId`, `etags`

- `GET /v6/pollUpload/:fileKey`
  - Poll upload status by `fileKey`

### File management

- `POST /v6/listFiles`
  - Request fields: `limit`, `offset`
  - Response includes `hasMore` and `files[]` with: `id`, `customId`, `key`,
    `name`, `status`, `size`, `uploadedAt`

- `POST /v6/renameFiles`
  - Request fields: `updates`

- `POST /v6/deleteFiles`
  - Request fields: `files`, `fileKeys`, `customIds`

- `POST /v6/updateACL`
  - Request fields: `updates`

- `POST /v6/requestFileAccess`
  - Request fields: `fileKey`, `customId`, `expiresIn`
  - Response includes `ufsUrl` (and deprecated `url`)

### App usage and callbacks

- `POST /v7/getAppInfo`
- `POST /v6/getUsageInfo`
- `GET /v6/serverCallback`
- `POST /v6/serverCallback` (fields: `fileKey`, `callbackData`)
- `POST /v6/failureCallback` (fields: `fileKey`, `uploadId`)

## RouteConfig Notes (`/v6/prepareUpload`)

`routeConfig` supports either:

- an array of file type strings (e.g. `image`, `video`, `audio`, `pdf`,
  `text`, `blob`, or specific MIME types), or
- an object keyed by file type/MIME, where each value can define:
  - `maxFileSize` (example: `4MB`)
  - `maxFileCount` (example: `1`)
  - `contentDisposition` (`inline` | `attachment`)
  - `acl` (`public-read` | `private`)

## Minimal Request Example

```bash
curl -X POST "https://api.uploadthing.com/v7/prepareUpload" \
  -H "Content-Type: application/json" \
  -H "x-uploadthing-api-key: YOUR_API_KEY" \
  -d '{
    "fileName": "example.png",
    "fileSize": 12345,
    "acl": "public-read"
  }'
```
