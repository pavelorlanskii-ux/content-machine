# Keyframe Generation Layer

Документ описывает слой генерации ключевых кадров для AI video pipeline.

## Назначение

Higgsfield DoP Turbo работает как image-to-video. Это значит, что для генерации видео сцены нужен не только prompt, но и `image_url`.

Поэтому перед DoP Turbo нужен отдельный слой генерации keyframe image.

## Место в общей архитектуре

Keyframe Generation Layer является частью visual/video layer, а не всей Content Machine.

Общая система остаётся универсальной:

```text
trend
  ↓
idea
  ↓
script
  ↓
storyboard
  ↓
visual bible
  ↓
keyframe generation
  ↓
AI video generation
  ↓
assembly
  ↓
publishing
  ↓
analytics
```

## Целевая цепочка

```text
scene_package
  ↓
keyframe_prompt
  ↓
Higgsfield Soul
  ↓
keyframe image
  ↓
storage layer
  ↓
public image_url
  ↓
Higgsfield DoP Turbo
  ↓
scene mp4
```

## Первый кандидат

Для генерации keyframe image используем Higgsfield Soul.

Роль Soul:

```text
text prompt → keyframe image
```

Роль DoP Turbo:

```text
keyframe image + prompt + motions → scene video
```

## Почему не отправлять текст сразу в видео

Генерация видео напрямую из текста менее стабильна.

Более устойчивый подход:

1. сначала получить управляемый первый кадр;
2. проверить, что объект и стиль верные;
3. оживить кадр через image-to-video;
4. при ошибке перегенерировать только keyframe или только video scene.

## Keyframe prompt

Keyframe prompt должен описывать:

- главный объект сцены;
- композицию;
- стиль;
- свет;
- цвета;
- формат;
- визуальные ограничения;
- отсутствие случайных логотипов;
- отсутствие нерелевантных объектов.

Prompt должен строиться из:

```text
scene_package.visual_prompt
  +
visual_bible
  +
scene-specific constraints
```

## Универсальность

Keyframe prompt не должен быть жёстко привязан к спорту или хоккею.

Пример спортивного sample-кейса `idea_001` может содержать хоккейную форму, но для других тем keyframe layer должен работать так же:

- образовательный ролик;
- продуктовый ролик;
- бизнес-ролик;
- lifestyle-сцена;
- новостной формат;
- AI-инструмент;
- брендовая история.

## Output

Ожидаемый результат keyframe generation:

```json
{
  "scene_id": "scene_01",
  "keyframe_status": "ready",
  "keyframe_url": "https://...",
  "source": "higgsfield-soul"
}
```

## Storage requirement

DoP Turbo требует публичный `image_url`.

Локальный путь не подходит:

```text
data/assets/references/scene_01.png
```

Нужен публичный URL:

```text
https://...
```

Варианты storage:

1. Higgsfield internal output URL, если Soul API возвращает публичную ссылку.
2. Cloudinary.
3. S3-compatible storage.
4. Supabase Storage.
5. GitHub raw для ручных тестов.
6. Временный tunnel/local file server только для разработки.

Для production лучше использовать S3-compatible storage, Cloudinary или Supabase Storage.

## n8n target flow

```text
Build Scene Packages
  ↓
Build Keyframe Prompts
  ↓
Create Higgsfield Soul Image
  ↓
Normalize Higgsfield Soul Response
  ↓
Wait for Keyframe Render
  ↓
Check Higgsfield Image Status
  ↓
Save Keyframe URL
  ↓
Prepare DoP Turbo Payload
  ↓
Create Higgsfield DoP Request
```

## Data files

Планируемые файлы для `idea_001`:

```text
data/outputs/idea_001-keyframe-prompts.json
data/outputs/idea_001-keyframe-results.json
```

Runtime-файлы, которые не обязательно коммитить:

```text
data/generated/keyframes/
data/generated/scenes/
```

## Environment

Переменные окружения:

```env
AI_IMAGE_PROVIDER=higgsfield
HIGGSFIELD_IMAGE_MODEL=soul
HIGGSFIELD_VIDEO_MODEL=dop-turbo
```

Связанные переменные:

```env
AI_VIDEO_PROVIDER=higgsfield
HIGGSFIELD_API_KEY=your_higgsfield_api_key_here
HIGGSFIELD_API_BASE_URL=https://platform.higgsfield.ai
```

## Open questions

Нужно проверить Higgsfield Soul API Reference:

- endpoint генерации изображения;
- обязательные поля body;
- response schema;
- status endpoint;
- где возвращается image URL;
- нужен ли отдельный upload/storage layer;
- exact authentication header;
- поддерживаемые aspect ratios;
- есть ли negative prompt;
- есть ли seed;
- есть ли enhance prompt;
- есть ли direct public image URL.

## Next steps

1. Открыть Higgsfield Soul в API Reference.
2. Зафиксировать endpoint и body schema.
3. Добавить `docs/higgsfield-soul-api-contract.md`.
4. Создать `data/outputs/idea_001-keyframe-prompts.json`.
5. Проверить, возвращает ли Soul публичный image URL.
6. Если URL не публичный, выбрать storage layer.
7. После этого собирать n8n-цепочку Soul → DoP Turbo.
