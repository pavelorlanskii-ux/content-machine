# Higgsfield API Contract

Документ фиксирует подтверждённый контракт Higgsfield DoP Turbo API для генерации video scene в проекте Content Machine.

## Назначение

Higgsfield DoP Turbo используется как AI Video Provider для генерации video scene из keyframe image.

```text
scene_package
  ↓
keyframe_url
  ↓
Higgsfield DoP Turbo
  ↓
scene mp4
  ↓
Scene QA
  ↓
Final Assembly Layer
```

## Важный вывод

DoP Turbo работает как `image-to-video`.

Для генерации сцены через API нужны:

```text
image_url
prompt
```

Опционально можно использовать `motions`, но для первого подтверждённого MVP-теста было использовано:

```json
"motions": null
```

## Base URL

```text
https://platform.higgsfield.ai
```

```env
HIGGSFIELD_API_BASE_URL=https://platform.higgsfield.ai
```

## Авторизация

Используются те же headers, что и для Soul:

```http
hf-api-key: <api_key_id>
hf-secret: <api_key_secret>
Content-Type: application/json
```

Локальные переменные:

```env
HIGGSFIELD_API_KEY_ID=your_higgsfield_api_key_id_here
HIGGSFIELD_API_KEY=your_higgsfield_api_key_secret_here
```

## Endpoints

```http
GET  /requests/{request_id}/status
POST /requests/{request_id}/cancel
GET  /v1/motions
POST /v1/image2video/dop
POST /higgsfield-ai/dop/turbo
```

Для MVP используем:

```http
POST /higgsfield-ai/dop/turbo
```

## Create DoP Turbo generation

```http
POST https://platform.higgsfield.ai/higgsfield-ai/dop/turbo
```

## Working request body

Подтверждённый рабочий body:

```json
{
  "seed": null,
  "prompt": "A cinematic close-up of a premium hockey jersey on a design table, modern sports-tech studio, AI interface glow in the background, black white teal orange accents, vertical 9:16, premium commercial look",
  "motions": null,
  "image_url": "https://d3u0tzju9qauci.cloudfront.net/...",
  "enhance_prompt": true
}
```

## Required fields

```text
image_url
prompt
```

## Field descriptions

### `image_url`

Публичный URL первого кадра.

В текущем pipeline берётся из Soul completed-response:

```text
keyframe_url = Soul images[0].url
```

### `prompt`

Описание движения и сцены.

Берётся из:

```text
scene_package.visual_prompt
```

или из отдельного `dop_prompt`.

### `motions`

Motion presets Higgsfield.

В Playground доступны motion presets, например:

```text
General
Handheld
Glam
Glowshift
Hyperlapse
Incline
Innerlight
Jelly Drift
Freezing
Garden Bloom
Head Tracking
```

В первом подтверждённом API-тесте массив строк вызвал validation error:

```json
"motions": ["General"]
```

Рабочий вариант для MVP:

```json
"motions": null
```

Перед использованием presets нужно проверить точную схему через:

```http
GET /v1/motions
```

### `seed`

Для MVP:

```json
"seed": null
```

### `enhance_prompt`

Для MVP:

```json
"enhance_prompt": true
```

## Create response

Реальный успешный ответ `POST /higgsfield-ai/dop/turbo`:

```json
{
  "status": "queued",
  "request_id": "5c5013c6-6fe8-415e-9cbf-cfca61812259",
  "status_url": "https://platform.higgsfield.ai/requests/5c5013c6-6fe8-415e-9cbf-cfca61812259/status",
  "cancel_url": "https://platform.higgsfield.ai/requests/5c5013c6-6fe8-415e-9cbf-cfca61812259/cancel"
}
```

## Status endpoint

```http
GET https://platform.higgsfield.ai/requests/{request_id}/status
```

## Status values

```text
queued
in_progress
nsfw
failed
completed
canceled
```

## Completed response

Реальный completed-response для DoP Turbo подтверждён в n8n:

```json
{
  "status": "completed",
  "request_id": "5c5013c6-6fe8-415e-9cbf-cfca61812259",
  "status_url": "https://platform.higgsfield.ai/requests/5c5013c6-6fe8-415e-9cbf-cfca61812259/status",
  "cancel_url": "https://platform.higgsfield.ai/requests/5c5013c6-6fe8-415e-9cbf-cfca61812259/cancel",
  "video": {
    "url": "https://cloud-cdn.higgsfield.ai/..."
  }
}
```

Внутренний mapping:

```text
scene_video_url = video.url
```

## Normalized scene video result

```json
{
  "scene_id": "scene_01",
  "scene_video_status": "ready",
  "video_request_id": "5c5013c6-6fe8-415e-9cbf-cfca61812259",
  "scene_video_url": "https://cloud-cdn.higgsfield.ai/...",
  "source": "higgsfield-dop-turbo",
  "completed_at": "2026-05-27T15:08:56.086Z"
}
```

## Confirmed full flow

Подтверждён реальный n8n flow:

```text
Higgsfield Soul
  ↓
images[0].url
  ↓
Higgsfield DoP Turbo
  ↓
video.url
```

## n8n notes

В n8n UI может отображаться preview-ошибка:

```text
[ERROR: access to env vars denied]
```

При этом реальное выполнение node работает, если контейнер запущен с:

```env
N8N_BLOCK_ENV_ACCESS_IN_NODE=false
```

## Open questions

Осталось проверить:

- точную схему `motions` через `GET /v1/motions`;
- можно ли безопасно использовать motion preset `General` через API;
- срок жизни URL из `images[0].url` и `video.url`;
- rate limits;
- стабильность качества на разных темах;
- поведение при batch generation.

## Next steps

1. Сохранить export тестового n8n workflow в `workflows/`.
2. Добавить normalized result samples в `data/outputs`.
3. Проверить `GET /v1/motions`.
4. Масштабировать pipeline с `scene_01` на все сцены.
