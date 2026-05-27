# n8n Higgsfield Integration Plan

Документ описывает подтверждённую интеграцию Higgsfield Soul и Higgsfield DoP Turbo в n8n workflow проекта Content Machine.

## Назначение

Цель интеграции: заменить случайный Pexels visual layer на управляемую AI video цепочку.

Подтверждённая связка:

```text
keyframe_prompt
  ↓
Higgsfield Soul
  ↓
images[0].url
  ↓
Higgsfield DoP Turbo
  ↓
video.url
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
keyframe_url
  ↓
DoP Turbo
  ↓
scene_video_url
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

Для n8n Docker compose нужен доступ к root `.env`:

```yaml
env_file:
  - ../../.env
```

И доступ к env в node expressions:

```yaml
environment:
  - N8N_BLOCK_ENV_ACCESS_IN_NODE=false
```

Реальные ключи не коммитить.

## Тестовый workflow

В n8n создан тестовый workflow:

```text
Test Higgsfield Soul Scene 01
```

Подтверждённая цепочка:

```text
Manual Trigger
  ↓
Build Soul Payload
  ↓
Create Soul Image
  ↓
Wait for Soul Render
  ↓
Check Soul Status
  ↓
Normalize Keyframe Result
  ↓
Build DoP Payload
  ↓
Create DoP Video
  ↓
Wait for DoP Render
  ↓
Check DoP Status
  ↓
Normalize Scene Video Result
```

## Block 1. Build Soul Payload

Code node.

```javascript
return [
  {
    json: {
      scene_id: "scene_01",
      source: "keyframe-generation-layer",
      provider: "higgsfield",
      model: "soul",
      soul_endpoint: "/v1/text2image/soul",
      soul_payload: {
        params: {
          seed: null,
          prompt: "A premium cinematic keyframe of a hockey jersey close-up on a design table, modern sports-tech studio, subtle AI interface glow in the background, black white teal orange accents, high-end commercial product photography, vertical 9:16, clean composition, shallow depth of field",
          quality: "720p",
          batch_size: 1,
          enhance_prompt: true,
          style_strength: 1,
          width_and_height: "1152x2048"
        },
        webhook: null
      }
    }
  }
];
```

Не отправлять без значения:

```text
style_id
image_reference
custom_reference_id
custom_reference_strength
```

## Block 2. Create Soul Image

HTTP Request node.

```text
Method: POST
URL: {{$env.HIGGSFIELD_API_BASE_URL + $json.soul_endpoint}}
Authentication: None
Send Headers: ON
Send Body: ON
Body Content Type: Raw
Content Type: application/json
Body: {{ JSON.stringify($json.soul_payload) }}
```

Headers:

```http
hf-api-key: {{$env.HIGGSFIELD_API_KEY_ID}}
hf-secret: {{$env.HIGGSFIELD_API_KEY}}
Content-Type: application/json
```

Успешный response содержит верхний `id`, который используется как request id.

## Block 3. Wait for Soul Render

Wait node:

```text
Resume: After Time Interval
Wait Amount: 60
Wait Unit: Seconds
```

Позже заменить на polling loop.

## Block 4. Check Soul Status

HTTP Request node.

```text
Method: GET
URL: {{$env.HIGGSFIELD_API_BASE_URL + "/requests/" + $json.id + "/status"}}
Authentication: None
Send Headers: ON
Send Body: OFF
```

Headers:

```http
hf-api-key: {{$env.HIGGSFIELD_API_KEY_ID}}
hf-secret: {{$env.HIGGSFIELD_API_KEY}}
```

Completed response:

```json
{
  "status": "completed",
  "request_id": "5185f35b-6849-492d-9343-61dadcc9f97f",
  "images": [
    {
      "url": "https://d3u0tzju9qauci.cloudfront.net/..."
    }
  ]
}
```

Mapping:

```text
keyframe_url = images[0].url
```

## Block 5. Normalize Keyframe Result

Code node.

```javascript
const item = $input.first().json;

if (item.status !== "completed") {
  return [
    {
      json: {
        status: "not_ready",
        provider_status: item.status,
        request_id: item.request_id,
        status_url: item.status_url,
        source: "higgsfield-soul"
      }
    }
  ];
}

const keyframeUrl = item.images?.[0]?.url;

if (!keyframeUrl) {
  throw new Error("Soul completed, but images[0].url was not found");
}

return [
  {
    json: {
      scene_id: "scene_01",
      keyframe_status: "ready",
      keyframe_request_id: item.request_id,
      keyframe_url: keyframeUrl,
      source: "higgsfield-soul",
      completed_at: new Date().toISOString()
    }
  }
];
```

## Block 6. Build DoP Payload

Code node.

```javascript
const item = $input.first().json;

if (item.keyframe_status !== "ready") {
  throw new Error("Keyframe is not ready");
}

return [
  {
    json: {
      scene_id: item.scene_id,
      source: "higgsfield-dop-turbo",
      provider: "higgsfield",
      model: "dop-turbo",
      dop_endpoint: "/higgsfield-ai/dop/turbo",
      keyframe_url: item.keyframe_url,
      dop_payload: {
        seed: null,
        prompt: "A cinematic close-up of a premium hockey jersey on a design table, modern sports-tech studio, AI interface glow in the background, black white teal orange accents, vertical 9:16, premium commercial look",
        motions: null,
        image_url: item.keyframe_url,
        enhance_prompt: true
      }
    }
  }
];
```

Важное ограничение: `"motions": ["General"]` вызвал validation error. Для MVP используем:

```json
"motions": null
```

## Block 7. Create DoP Video

HTTP Request node.

```text
Method: POST
URL: {{$env.HIGGSFIELD_API_BASE_URL + $json.dop_endpoint}}
Authentication: None
Send Headers: ON
Send Body: ON
Body Content Type: Raw
Content Type: application/json
Body: {{ JSON.stringify($json.dop_payload) }}
```

Headers:

```http
hf-api-key: {{$env.HIGGSFIELD_API_KEY_ID}}
hf-secret: {{$env.HIGGSFIELD_API_KEY}}
Content-Type: application/json
```

Успешный response:

```json
{
  "status": "queued",
  "request_id": "5c5013c6-6fe8-415e-9cbf-cfca61812259",
  "status_url": "https://platform.higgsfield.ai/requests/5c5013c6-6fe8-415e-9cbf-cfca61812259/status",
  "cancel_url": "https://platform.higgsfield.ai/requests/5c5013c6-6fe8-415e-9cbf-cfca61812259/cancel"
}
```

## Block 8. Wait for DoP Render

Wait node:

```text
Resume: After Time Interval
Wait Amount: 60
Wait Unit: Seconds
```

Позже заменить на polling loop.

## Block 9. Check DoP Status

HTTP Request node.

```text
Method: GET
URL: {{$env.HIGGSFIELD_API_BASE_URL + "/requests/" + $json.request_id + "/status"}}
Authentication: None
Send Headers: ON
Send Body: OFF
```

Headers:

```http
hf-api-key: {{$env.HIGGSFIELD_API_KEY_ID}}
hf-secret: {{$env.HIGGSFIELD_API_KEY}}
```

Completed response:

```json
{
  "status": "completed",
  "request_id": "5c5013c6-6fe8-415e-9cbf-cfca61812259",
  "video": {
    "url": "https://cloud-cdn.higgsfield.ai/..."
  }
}
```

Mapping:

```text
scene_video_url = video.url
```

## Block 10. Normalize Scene Video Result

Code node.

```javascript
const item = $input.first().json;

if (item.status !== "completed") {
  return [
    {
      json: {
        status: "not_ready",
        provider_status: item.status,
        request_id: item.request_id,
        status_url: item.status_url,
        source: "higgsfield-dop-turbo"
      }
    }
  ];
}

const sceneVideoUrl = item.video?.url;

if (!sceneVideoUrl) {
  throw new Error("DoP completed, but video.url was not found");
}

return [
  {
    json: {
      scene_id: "scene_01",
      scene_video_status: "ready",
      video_request_id: item.request_id,
      scene_video_url: sceneVideoUrl,
      source: "higgsfield-dop-turbo",
      completed_at: new Date().toISOString()
    }
  }
];
```

## Confirmed result

Финальный normalized output:

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

## Current notes

В n8n UI может отображаться preview-ошибка:

```text
[ERROR: access to env vars denied]
```

При этом реальное выполнение node работает, если:

```env
N8N_BLOCK_ENV_ACCESS_IN_NODE=false
```

## Current blockers

Осталось сделать:

- экспортировать тестовый n8n workflow в `workflows/`;
- сохранить normalized result samples в `data/outputs`;
- заменить фиксированные `Wait 60 seconds` на polling loop;
- масштабировать pipeline с `scene_01` на все сцены;
- проверить схему `motions` через `GET /v1/motions`;
- интегрировать результат в Final Assembly Layer.
