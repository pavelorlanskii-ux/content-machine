# Higgsfield Soul API Contract

Документ фиксирует подтверждённый контракт Higgsfield Soul API для генерации keyframe images в проекте Content Machine.

## Назначение

Higgsfield Soul используется как image generation layer перед Higgsfield DoP Turbo.

```text
scene_package
  ↓
keyframe_prompt
  ↓
Higgsfield Soul
  ↓
keyframe image
  ↓
keyframe_url
  ↓
Higgsfield DoP Turbo
  ↓
scene mp4
```

## Base URL

```text
https://platform.higgsfield.ai
```

```env
HIGGSFIELD_API_BASE_URL=https://platform.higgsfield.ai
```

## Авторизация

Подтверждённые headers:

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

Реальные значения не коммитить.

## Endpoints

```http
GET  /requests/{request_id}/status
POST /requests/{request_id}/cancel
POST /v1/text2image/soul
GET  /v1/text2image/soul-styles
POST /soul
```

Для MVP используем:

```http
POST /v1/text2image/soul
```

## Create Soul image generation

```http
POST https://platform.higgsfield.ai/v1/text2image/soul
```

## Working request body

Подтверждённый рабочий body для MVP:

```json
{
  "params": {
    "seed": null,
    "prompt": "A premium cinematic keyframe for a vertical social video",
    "quality": "720p",
    "batch_size": 1,
    "enhance_prompt": true,
    "style_strength": 1,
    "width_and_height": "1152x2048"
  },
  "webhook": null
}
```

## Required fields

```text
params
params.prompt
params.width_and_height
```

## Field notes

### `params.prompt`

Берётся из:

```text
keyframe_prompt.prompt
```

### `params.width_and_height`

Для вертикального видео 9:16 используем:

```text
1152x2048
```

Другие значения из API Reference:

```text
2048x1152
2048x1536
1536x2048
1344x2016
```

### `params.quality`

Для MVP:

```text
720p
```

Для production-тестов можно проверять:

```text
1080p
```

### `params.batch_size`

Для MVP:

```json
"batch_size": 1
```

### `params.enhance_prompt`

Для MVP:

```json
"enhance_prompt": true
```

### `params.style_strength`

Для MVP:

```json
"style_strength": 1
```

## Поля, которые не отправляем без значения

В тесте API возвращал UUID validation error, если отправлять пустые или `null` reference/style поля.

В базовом MVP не отправляем:

```text
style_id
image_reference
custom_reference_id
custom_reference_strength
```

Эти поля добавлять только когда есть валидный UUID или валидный reference object.

## Example request body for `idea_001`

```json
{
  "params": {
    "seed": null,
    "prompt": "A premium cinematic keyframe of a hockey jersey close-up on a design table, modern sports-tech studio, subtle AI interface glow in the background, black white teal orange accents, high-end commercial product photography, vertical 9:16, clean composition, shallow depth of field",
    "quality": "720p",
    "batch_size": 1,
    "enhance_prompt": true,
    "style_strength": 1,
    "width_and_height": "1152x2048"
  },
  "webhook": null
}
```

## Create response

Реальный успешный ответ `POST /v1/text2image/soul`:

```json
{
  "id": "5185f35b-6849-492d-9343-61dadcc9f97f",
  "type": "text2image_soul",
  "created_at": "2026-05-27T14:04:38.106870Z",
  "jobs": [
    {
      "id": "ce1076ae-c22a-46bd-8cae-ec6897d8cb18",
      "job_set_type": "text2image_soul",
      "status": "queued",
      "results": null
    }
  ]
}
```

Для проверки статуса используем верхний `id`:

```text
request_id = response.id
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

Реальный completed-response для Soul подтверждён в n8n:

```json
{
  "status": "completed",
  "request_id": "5185f35b-6849-492d-9343-61dadcc9f97f",
  "status_url": "https://platform.higgsfield.ai/requests/5185f35b-6849-492d-9343-61dadcc9f97f/status",
  "cancel_url": "https://platform.higgsfield.ai/requests/5185f35b-6849-492d-9343-61dadcc9f97f/cancel",
  "images": [
    {
      "url": "https://d3u0tzju9qauci.cloudfront.net/..."
    }
  ]
}
```

Внутренний mapping:

```text
keyframe_url = images[0].url
```

## Normalized keyframe result

```json
{
  "scene_id": "scene_01",
  "keyframe_status": "ready",
  "keyframe_request_id": "5185f35b-6849-492d-9343-61dadcc9f97f",
  "keyframe_url": "https://d3u0tzju9qauci.cloudfront.net/...",
  "source": "higgsfield-soul",
  "completed_at": "2026-05-27T14:36:49.979Z"
}
```

## Confirmed facts

Подтверждено реальным n8n-тестом:

- env headers работают через `hf-api-key` и `hf-secret`;
- `POST /v1/text2image/soul` создаёт request;
- `GET /requests/{request_id}/status` возвращает completed status;
- keyframe image URL лежит в `images[0].url`;
- этот URL можно передать дальше в DoP Turbo.

## n8n notes

В n8n UI может отображаться preview-ошибка:

```text
[ERROR: access to env vars denied]
```

При этом реальное выполнение node работает, если контейнер запущен с:

```env
N8N_BLOCK_ENV_ACCESS_IN_NODE=false
```

## Next steps

1. Сохранить export тестового n8n workflow в `workflows/`.
2. Добавить normalized result samples в `data/outputs`.
3. Масштабировать Soul generation с `scene_01` на все сцены.
