# Higgsfield API Contract

Документ фиксирует текущий контракт Higgsfield Cloud API для интеграции AI Video Provider в проект Content Machine.

## Назначение

Higgsfield рассматривается как первый AI Video Provider для замены Pexels stock footage.

Текущая целевая роль Higgsfield:

```text
scene_package
  ↓
keyframe image
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

Это значит, что для генерации сцены через API нужен не только текстовый prompt, но и доступная по URL картинка первого кадра.

```text
image_url + prompt + motions → video generation request
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

В кабинете Higgsfield Cloud создаётся API key.

Локальная переменная:

```env
HIGGSFIELD_API_KEY=your_higgsfield_api_key_here
```

`API Key ID` пока не используется в контракте, так как в API Reference не найдено отдельного обязательного поля или header для `key_id`.

Если в дальнейшем Higgsfield потребует отдельный key id, добавим:

```env
HIGGSFIELD_API_KEY_ID=your_higgsfield_api_key_id_here
```

## Create DoP Turbo generation

Endpoint:

```http
POST /higgsfield-ai/dop/turbo
```

Полный URL:

```text
https://platform.higgsfield.ai/higgsfield-ai/dop/turbo
```

## Request body

По API Reference доступны поля:

```json
{
  "seed": null,
  "prompt": "",
  "motions": null,
  "image_url": "",
  "enhance_prompt": true
}
```

## Required fields

Обязательные поля:

```text
image_url
prompt
```

## Field descriptions

### `image_url`

Публично доступный URL первого кадра.

Это может быть:

- URL изображения из внешнего storage;
- URL keyframe image, сгенерированного отдельной моделью;
- временный публичный URL из storage layer.

Для локальных файлов это не сработает напрямую:

```text
data/assets/references/image.png
```

Нужно сначала получить публичный URL.

### `prompt`

Основное описание движения и сцены.

Берётся из `scene_package.visual_prompt`.

### `motions`

Список motion presets Higgsfield.

Пример:

```json
"motions": ["General"]
```

Для текущего `idea_001` используется:

```text
scene_01 → General
scene_02 → Handheld
scene_03 → General
```

### `seed`

Число для повторяемости генерации.

Для MVP можно оставлять:

```json
"seed": null
```

### `enhance_prompt`

Флаг автоматического улучшения prompt на стороне Higgsfield.

Для MVP:

```json
"enhance_prompt": true
```

## Example request body

```json
{
  "seed": null,
  "prompt": "A cinematic close-up of a premium hockey jersey on a design table, modern sports-tech studio, AI interface glow in the background, black white teal orange accents, vertical 9:16, premium commercial look",
  "motions": ["General"],
  "image_url": "https://example.com/keyframes/idea_001-scene_01.png",
  "enhance_prompt": true
}
```

## Example create response

По API Reference успешный ответ имеет вид:

```json
{
  "status": "queued",
  "request_id": "123e4567-e89b-12d3-a456-4266141740",
  "status_url": "https://example.com",
  "cancel_url": "https://example.com"
}
```

## Generation statuses

Возможные статусы:

```text
queued
in_progress
nsfw
failed
completed
canceled
```

## Check generation status

Endpoint:

```http
GET /requests/{request_id}/status
```

Полный URL:

```text
https://platform.higgsfield.ai/requests/{request_id}/status
```

## Example status response

Текущий пример из API Reference показывает queued-ответ:

```json
{
  "status": "queued",
  "request_id": "123e4567-e89b-12d3-a456-4266141740",
  "status_url": "https://example.com",
  "cancel_url": "https://example.com"
}
```

## Completed response

Схема completed-response пока не подтверждена.

Нужно выполнить реальный request и проверить, где возвращается ссылка на готовое видео.

Ожидаемые возможные варианты:

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
  "video_url": "https://..."
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

До реального теста это остаётся open question.

## Cancel generation

Endpoint:

```http
POST /requests/{request_id}/cancel
```

Используется для отмены генерации.

## List motions

Endpoint:

```http
GET /v1/motions
```

Используется для получения списка доступных motion presets.

В Playground были видны примеры:

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

Для текущего MVP безопасные presets:

```text
General
Handheld
```

## n8n target flow

Целевая цепочка n8n для Higgsfield:

```text
Build Scene Package
  ↓
Prepare Keyframe Image URL
  ↓
Create Higgsfield DoP Request
  ↓
Normalize Higgsfield Create Response
  ↓
Wait for Higgsfield Render
  ↓
Check Higgsfield Status
  ↓
If queued / in_progress → wait and repeat
  ↓
If completed → save result URL
  ↓
If failed / nsfw / canceled → return error
```

## Storage requirement

Так как DoP Turbo требует `image_url`, нужен storage layer для keyframes.

Варианты:

1. ручной upload в Higgsfield Playground для тестов;
2. внешний image hosting;
3. S3-compatible storage;
4. Cloudinary;
5. Supabase Storage;
6. GitHub raw для тестовых reference images;
7. отдельный локальный сервис с публичным URL через tunnel.

Для production лучше использовать S3-compatible storage или Cloudinary.

## Current MVP limitation

Пока не подтверждено:

- exact authentication header;
- completed response schema;
- direct mp4 URL field;
- upload endpoint for local images;
- pricing and rate limits;
- practical quality of DoP Turbo for hockey jersey scenes.

## Next steps

1. Сделать ручной test generation в Playground после пополнения credits.
2. Проверить вкладку `Requests` после completed generation.
3. Зафиксировать completed response schema.
4. Выбрать keyframe storage layer.
5. Добавить n8n node `Create Higgsfield DoP Request`.
6. Добавить n8n node `Check Higgsfield Status`.
7. Добавить нормализацию результата в `higgsfield_scene_result`.
