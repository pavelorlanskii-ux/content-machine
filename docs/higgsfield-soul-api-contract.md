# Higgsfield Soul API Contract

Документ фиксирует текущий контракт Higgsfield Soul API для генерации keyframe images в проекте Content Machine.

## Назначение

Higgsfield Soul рассматривается как первый image generation layer для создания ключевых кадров перед Higgsfield DoP Turbo.

Целевая связка:

```text
scene_package
  ↓
keyframe_prompt
  ↓
Higgsfield Soul
  ↓
keyframe image
  ↓
public image_url
  ↓
Higgsfield DoP Turbo
  ↓
scene mp4
```

## Base URL

```text
https://platform.higgsfield.ai
```

Переменная окружения:

```env
HIGGSFIELD_API_BASE_URL=https://platform.higgsfield.ai
```

## Авторизация

В API Reference для `POST /v1/text2image/soul` видны обязательные headers:

```http
hf-api-key: <api_key_id>
hf-secret: <api_key_secret>
Content-Type: application/json
```

Значит для локального `.env` нужны оба значения:

```env
HIGGSFIELD_API_KEY_ID=your_higgsfield_api_key_id_here
HIGGSFIELD_API_KEY=your_higgsfield_api_key_secret_here
```

Важное правило: реальные значения не коммитить и не отправлять в чат.

## Endpoints

В API Reference Higgsfield Soul видны endpoints:

```http
GET  /requests/{request_id}/status
POST /requests/{request_id}/cancel
POST /v1/text2image/soul
GET  /v1/text2image/soul-styles
POST /soul
```

Для MVP используем основной endpoint:

```http
POST /v1/text2image/soul
```

## Create Soul image generation

Endpoint:

```http
POST /v1/text2image/soul
```

Полный URL:

```text
https://platform.higgsfield.ai/v1/text2image/soul
```

## Request body

По API Reference body содержит объект `params` и `webhook`:

```json
{
  "params": {
    "seed": null,
    "prompt": "",
    "quality": "720p",
    "style_id": null,
    "batch_size": 1,
    "enhance_prompt": true,
    "style_strength": 1,
    "image_reference": {
      "type": "image_url",
      "image_url": ""
    },
    "width_and_height": "1536x1536",
    "custom_reference_id": "",
    "custom_reference_strength": 1
  },
  "webhook": null
}
```

## Required fields

По API Reference обязательные поля:

```text
params
prompt
width_and_height
```

## Field descriptions

### `params.prompt`

Основной prompt для генерации keyframe.

Берётся из:

```text
keyframe_prompt.prompt
```

### `params.width_and_height`

Размер изображения.

Доступные значения из API Reference:

```text
1152x2048
2048x1152
2048x1536
1536x2048
1344x2016
```

Для вертикального видео 9:16 оптимально:

```text
1152x2048
```

### `params.quality`

Доступные значения:

```text
720p
1080p
```

Для MVP:

```text
720p
```

Для production-тестов можно использовать:

```text
1080p
```

### `params.batch_size`

Количество изображений.

Доступные значения:

```text
1
4
```

Для MVP:

```json
"batch_size": 1
```

### `params.enhance_prompt`

Флаг автоматического улучшения prompt.

Для MVP:

```json
"enhance_prompt": true
```

### `params.seed`

Seed для повторяемости генерации.

Для MVP:

```json
"seed": null
```

### `params.style_id`

Soul Style ID.

Для MVP:

```json
"style_id": null
```

Список стилей можно получить через:

```http
GET /v1/text2image/soul-styles
```

### `params.style_strength`

Сила стиля.

Диапазон из API Reference:

```text
0..1
```

Для MVP:

```json
"style_strength": 1
```

### `params.image_reference`

Опциональный image reference.

Для text-to-image без reference можно передавать:

```json
"image_reference": null
```

или не использовать reference, если API это допускает.

Если нужен reference image:

```json
"image_reference": {
  "type": "image_url",
  "image_url": "https://..."
}
```

### `webhook`

Для MVP:

```json
"webhook": null
```

## Example request body for Content Machine

```json
{
  "params": {
    "seed": null,
    "prompt": "A premium cinematic keyframe for a vertical social video, clean high-end commercial look, controlled studio lighting, strong composition, no random logos, no broken text",
    "quality": "720p",
    "style_id": null,
    "batch_size": 1,
    "enhance_prompt": true,
    "style_strength": 1,
    "image_reference": null,
    "width_and_height": "1152x2048",
    "custom_reference_id": "",
    "custom_reference_strength": 1
  },
  "webhook": null
}
```

## Example request body for `idea_001`

```json
{
  "params": {
    "seed": null,
    "prompt": "A premium cinematic keyframe of a hockey jersey close-up on a design table, modern sports-tech studio, subtle AI interface glow in the background, black white teal orange accents, high-end commercial product photography, vertical 9:16, clean composition, shallow depth of field",
    "quality": "720p",
    "style_id": null,
    "batch_size": 1,
    "enhance_prompt": true,
    "style_strength": 1,
    "image_reference": null,
    "width_and_height": "1152x2048",
    "custom_reference_id": "",
    "custom_reference_strength": 1
  },
  "webhook": null
}
```

## Create response

По API Reference успешный ответ похож на общий request-response:

```json
{
  "status": "queued",
  "request_id": "123e4567-e89b-12d3-a456-4266141740",
  "status_url": "https://example.com",
  "cancel_url": "https://example.com"
}
```

## Status endpoint

Endpoint:

```http
GET /requests/{request_id}/status
```

Полный URL:

```text
https://platform.higgsfield.ai/requests/{request_id}/status
```

## Status values

Возможные статусы:

```text
queued
in_progress
nsfw
failed
completed
canceled
```

## Completed response

Схема completed-response пока не подтверждена реальным request.

Нужно выполнить генерацию и проверить, где возвращается URL результата.

Ожидаемые варианты:

```json
{
  "status": "completed",
  "result_url": "https://..."
}
```

или:

```json
{
  "status": "completed",
  "image_url": "https://..."
}
```

или:

```json
{
  "status": "completed",
  "output": {
    "url": "https://..."
  }
}
```

## Soul styles

Endpoint:

```http
GET /v1/text2image/soul-styles
```

Используется для получения списка `style_id`.

В MVP можно использовать:

```json
"style_id": null
```

## n8n target flow

```text
Build Keyframe Prompts
  ↓
Create Higgsfield Soul Image
  ↓
Normalize Higgsfield Soul Create Response
  ↓
Wait for Soul Render
  ↓
Check Higgsfield Request Status
  ↓
If queued / in_progress → wait and repeat
  ↓
If completed → save keyframe_url
  ↓
If failed / nsfw / canceled → return error
```

## Connection to DoP Turbo

После получения `keyframe_url` он используется как `image_url` в DoP Turbo:

```json
{
  "image_url": "https://...",
  "prompt": "...",
  "motions": ["General"],
  "seed": null,
  "enhance_prompt": true
}
```

## Open questions

Нужно подтвердить через реальный request:

- exact completed response schema;
- поле, где лежит image URL;
- публичный ли URL результата;
- можно ли сразу использовать этот URL в DoP Turbo;
- нужен ли дополнительный storage layer;
- поддерживается ли `image_reference: null`;
- нужен ли endpoint `/soul` или достаточно `/v1/text2image/soul`.

## Next steps

1. Добавить `HIGGSFIELD_API_KEY_ID` в `.env.example`.
2. Создать `data/outputs/idea_001-keyframe-prompts.json`.
3. После пополнения credits сделать тестовый Soul request.
4. Проверить completed response.
5. Зафиксировать поле `keyframe_url`.
6. После этого строить n8n-цепочку Soul → DoP Turbo.
