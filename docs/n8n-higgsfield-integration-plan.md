# n8n Higgsfield Integration Plan

Документ описывает план интеграции Higgsfield Soul и Higgsfield DoP Turbo в n8n workflow проекта Content Machine.

## Назначение

Цель интеграции: заменить случайный Pexels visual layer на управляемую AI video цепочку.

Целевая связка:

```text
keyframe_prompts
  ↓
Higgsfield Soul
  ↓
keyframe_url
  ↓
scene_packages
  ↓
Higgsfield DoP Turbo
  ↓
scene_video_url
  ↓
Final Assembly Layer
```

## Общая логика

Higgsfield используется в двух ролях:

1. `Higgsfield Soul` генерирует keyframe image.
2. `Higgsfield DoP Turbo` превращает keyframe image в video scene.

```text
text prompt
  ↓
Soul
  ↓
image URL
  ↓
DoP Turbo
  ↓
video URL
```

## Требования к env

В `.env` должны быть реальные значения:

```env
AI_IMAGE_PROVIDER=higgsfield
AI_VIDEO_PROVIDER=higgsfield

HIGGSFIELD_API_BASE_URL=https://platform.higgsfield.ai
HIGGSFIELD_API_KEY_ID=real_api_key_id_here
HIGGSFIELD_API_KEY=real_api_key_secret_here

HIGGSFIELD_IMAGE_MODEL=soul
HIGGSFIELD_VIDEO_MODEL=dop-turbo
```

Важно: реальные ключи не коммитить.

## Входные файлы

Для тестового `idea_001` используются:

```text
data/outputs/idea_001-keyframe-prompts.json
data/outputs/idea_001-scene-packages.json
```

## Выходные файлы

Планируемые результаты:

```text
data/outputs/idea_001-keyframe-results.json
data/outputs/idea_001-higgsfield-scene-results.json
```

Runtime-файлы, если понадобится локальное скачивание:

```text
data/generated/keyframes/
data/generated/scenes/
```

## n8n workflow blocks

Целевой workflow лучше собирать блоками.

```text
Load Keyframe Prompts
  ↓
Build Soul Payload
  ↓
Create Soul Image
  ↓
Normalize Soul Response
  ↓
Wait for Soul Render
  ↓
Check Soul Status
  ↓
Normalize Keyframe Result
  ↓
Merge Keyframe URL with Scene Package
  ↓
Build DoP Payload
  ↓
Create DoP Video
  ↓
Normalize DoP Response
  ↓
Wait for DoP Render
  ↓
Check DoP Status
  ↓
Normalize Scene Video Result
```

## Block 1. Load Keyframe Prompts

Источник:

```text
/files/data/outputs/idea_001-keyframe-prompts.json
```

В n8n Docker локальный путь проекта должен быть смонтирован как:

```text
content-machine/data
  ↓
/files/data
```

Output должен содержать массив:

```json
{
  "keyframe_prompts": []
}
```

## Block 2. Split Keyframe Prompts

Каждый keyframe prompt должен стать отдельным item.

Ожидаемый item:

```json
{
  "scene_id": "scene_01",
  "prompt": "...",
  "width_and_height": "1152x2048",
  "quality": "720p",
  "batch_size": 1,
  "enhance_prompt": true,
  "style_id": null,
  "style_strength": 1,
  "image_reference": null,
  "seed": null
}
```

## Block 3. Build Soul Payload

Создаёт body для `POST /v1/text2image/soul`.

Target endpoint:

```text
https://platform.higgsfield.ai/v1/text2image/soul
```

Headers:

```http
hf-api-key: {{$env.HIGGSFIELD_API_KEY_ID}}
hf-secret: {{$env.HIGGSFIELD_API_KEY}}
Content-Type: application/json
```

Body:

```json
{
  "params": {
    "seed": null,
    "prompt": "{{$json.prompt}}",
    "quality": "{{$json.quality}}",
    "style_id": null,
    "batch_size": 1,
    "enhance_prompt": true,
    "style_strength": 1,
    "image_reference": null,
    "width_and_height": "{{$json.width_and_height}}",
    "custom_reference_id": "",
    "custom_reference_strength": 1
  },
  "webhook": null
}
```

## Block 4. Create Soul Image

HTTP Request node:

```text
POST {{$env.HIGGSFIELD_API_BASE_URL}}/v1/text2image/soul
```

Expected response:

```json
{
  "status": "queued",
  "request_id": "...",
  "status_url": "...",
  "cancel_url": "..."
}
```

## Block 5. Normalize Soul Response

Output:

```json
{
  "scene_id": "scene_01",
  "keyframe_request_id": "...",
  "keyframe_status": "queued",
  "keyframe_status_url": "...",
  "source": "higgsfield-soul"
}
```

## Block 6. Wait for Soul Render

Для MVP можно использовать фиксированное ожидание:

```text
Wait 60 seconds
```

Дальше заменить на цикл:

```text
check status
if queued / in_progress, wait and repeat
if completed, continue
if failed / nsfw / canceled, return error
```

## Block 7. Check Soul Status

HTTP Request node:

```text
GET {{$env.HIGGSFIELD_API_BASE_URL}}/requests/{{$json.keyframe_request_id}}/status
```

Headers:

```http
hf-api-key: {{$env.HIGGSFIELD_API_KEY_ID}}
hf-secret: {{$env.HIGGSFIELD_API_KEY}}
```

Possible statuses:

```text
queued
in_progress
nsfw
failed
completed
canceled
```

## Block 8. Normalize Keyframe Result

Цель: привести ответ Higgsfield к внутреннему формату.

Target normalized output:

```json
{
  "scene_id": "scene_01",
  "keyframe_status": "ready",
  "keyframe_request_id": "...",
  "keyframe_url": "https://...",
  "source": "higgsfield-soul"
}
```

Open question: точное поле `keyframe_url` будет известно только после реального completed-response.

## Block 9. Load Scene Packages

Источник:

```text
/files/data/outputs/idea_001-scene-packages.json
```

Каждый scene package должен быть объединён с соответствующим `keyframe_url` по `scene_id`.

## Block 10. Build DoP Payload

Target endpoint:

```text
https://platform.higgsfield.ai/higgsfield-ai/dop/turbo
```

Headers:

```http
hf-api-key: {{$env.HIGGSFIELD_API_KEY_ID}}
hf-secret: {{$env.HIGGSFIELD_API_KEY}}
Content-Type: application/json
```

Body:

```json
{
  "seed": null,
  "prompt": "{{$json.visual_prompt}}",
  "motions": "{{$json.motions}}",
  "image_url": "{{$json.keyframe_url}}",
  "enhance_prompt": true
}
```

## Block 11. Create DoP Video

HTTP Request node:

```text
POST {{$env.HIGGSFIELD_API_BASE_URL}}/higgsfield-ai/dop/turbo
```

Expected response:

```json
{
  "status": "queued",
  "request_id": "...",
  "status_url": "...",
  "cancel_url": "..."
}
```

## Block 12. Normalize DoP Response

Output:

```json
{
  "scene_id": "scene_01",
  "video_request_id": "...",
  "video_status": "queued",
  "video_status_url": "...",
  "source": "higgsfield-dop-turbo"
}
```

## Block 13. Wait for DoP Render

Для MVP:

```text
Wait 60 seconds
```

Для production:

```text
poll until completed / failed / nsfw / canceled
```

## Block 14. Check DoP Status

HTTP Request node:

```text
GET {{$env.HIGGSFIELD_API_BASE_URL}}/requests/{{$json.video_request_id}}/status
```

Headers:

```http
hf-api-key: {{$env.HIGGSFIELD_API_KEY_ID}}
hf-secret: {{$env.HIGGSFIELD_API_KEY}}
```

## Block 15. Normalize Scene Video Result

Target normalized output:

```json
{
  "scene_id": "scene_01",
  "scene_video_status": "ready",
  "video_request_id": "...",
  "scene_video_url": "https://...",
  "source": "higgsfield-dop-turbo"
}
```

Open question: точное поле `scene_video_url` будет известно только после реального completed-response.

## Error handling

Статусы, которые нужно считать ошибкой:

```text
failed
nsfw
canceled
```

Normalized error:

```json
{
  "scene_id": "scene_01",
  "status": "failed",
  "source": "higgsfield",
  "error_stage": "soul_or_dop",
  "provider_status": "failed",
  "retry_required": true
}
```

## Retry logic

Для MVP retry можно делать вручную.

Для production:

```text
if keyframe failed → regenerate keyframe
if video failed → regenerate video from same keyframe
if nsfw → adjust prompt / negative prompt
if style mismatch → adjust visual bible / keyframe prompt
```

## Current blockers

Пока не подтверждено:

- exact completed response schema for Soul;
- exact completed response schema for DoP Turbo;
- где лежит `keyframe_url`;
- где лежит `scene_video_url`;
- публичны ли URL результата;
- сколько длится generation;
- сколько credits требуется на полный 3-scene pipeline.

## Manual test required

Перед полной автоматизацией нужен один тест:

```text
Soul prompt
  ↓
completed response
  ↓
keyframe_url
  ↓
DoP Turbo
  ↓
completed response
  ↓
scene_video_url
```

После этого можно финализировать n8n nodes.

## MVP implementation order

1. Создать n8n branch workflow copy.
2. Load `idea_001-keyframe-prompts.json`.
3. Build Soul Payload.
4. Create Soul Image.
5. Check Soul Status.
6. Посмотреть completed-response.
7. Обновить `docs/higgsfield-soul-api-contract.md`.
8. Build DoP Payload.
9. Create DoP Video.
10. Check DoP Status.
11. Посмотреть completed-response.
12. Обновить `docs/higgsfield-api-contract.md`.
13. Сохранить нормализованные results в `data/outputs`.
