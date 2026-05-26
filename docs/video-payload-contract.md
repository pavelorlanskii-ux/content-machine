# Video Payload Contract

Документ описывает структуру данных, которую Script Agent workflow передаёт на следующий этап видеогенерации.

## Назначение

`video_payload` нужен как промежуточный формат между генерацией сценария и сервисом сборки вертикального видео.

Поток:

```text
Idea Input
  ↓
Build Script Prompt
  ↓
LLM Script Generator
  ↓
Normalize Script Output
  ↓
Build Video Payload
  ↓
Final Output
  ↓
Video Generation Service
```

## Текущий формат

```json
{
  "id": "idea_001",
  "status": "ready_for_video_generation",
  "video_payload": {
    "id": "idea_001",
    "format": "vertical_video",
    "aspect_ratio": "9:16",
    "platform": "reels_tiktok_shorts",
    "language": "ru",
    "duration_seconds": 30,
    "source": "openai",
    "generated_at": "2026-05-26T14:10:12.143Z",
    "production_status": "script_ready",
    "next_step": "video_generation",
    "video_requirements": {
      "resolution": "1080x1920",
      "captions": true,
      "voiceover": true,
      "background_music": true,
      "safe_zones": true
    },
    "script_markdown": "..."
  },
  "completed_at": "2026-05-26T14:10:12.154Z"
}
```

## Поля верхнего уровня

| Поле | Тип | Описание |
|---|---|---|
| `id` | string | ID идеи или ролика |
| `status` | string | Текущий статус обработки |
| `video_payload` | object | Основной объект для видеогенерации |
| `completed_at` | string | Время завершения подготовки payload |

## Поля `video_payload`

| Поле | Тип | Описание |
|---|---|---|
| `id` | string | ID ролика |
| `format` | string | Формат видео, сейчас `vertical_video` |
| `aspect_ratio` | string | Соотношение сторон, сейчас `9:16` |
| `platform` | string | Целевая группа платформ |
| `language` | string | Язык ролика |
| `duration_seconds` | number | Плановая длительность ролика |
| `source` | string | Источник сценария, например `openai` |
| `generated_at` | string | Время генерации сценария |
| `production_status` | string | Производственный статус |
| `next_step` | string | Следующий этап пайплайна |
| `video_requirements` | object | Технические требования к видео |
| `script_markdown` | string | Сценарий в Markdown |

## Поля `video_requirements`

| Поле | Тип | Описание |
|---|---|---|
| `resolution` | string | Целевое разрешение видео |
| `captions` | boolean | Нужны ли субтитры |
| `voiceover` | boolean | Нужна ли озвучка |
| `background_music` | boolean | Нужна ли фоновая музыка |
| `safe_zones` | boolean | Нужно ли учитывать безопасные зоны интерфейсов Reels, TikTok и Shorts |

## Требования к видеогенератору

На следующем этапе видеогенератор должен уметь принять `video_payload` и выделить из `script_markdown`:

1. voiceover;
2. on-screen text;
3. scene descriptions;
4. duration;
5. CTA.

## Ближайшая задача

Следующий шаг: создать адаптер, который преобразует `script_markdown` в более строгий JSON для video pipeline.

Целевой формат следующего этапа:

```json
{
  "id": "idea_001",
  "title": "...",
  "duration_seconds": 30,
  "voiceover": "...",
  "captions": [
    "..."
  ],
  "scenes": [
    {
      "scene": 1,
      "duration": "0-3 сек",
      "visual": "...",
      "on_screen_text": "..."
    }
  ],
  "cta": "..."
}
```

## Роль этого контракта в проекте

Этот документ нужен, чтобы не смешивать сценарий, видео-структуру и публикационный слой.

Правильная логика такая:

```text
script_markdown
  сырой сценарий от Script Agent

video_payload
  производственный объект для видео-пайплайна

video_pipeline_json
  строгая структура для конкретного видеогенератора

publishing_payload
  данные для постинга: caption, hashtags, title, thumbnail
```

## Следующие улучшения

В следующих версиях контракта нужно добавить:

- отдельное поле `title`;
- отдельное поле `voiceover`;
- массив `captions`;
- массив `scenes`;
- поле `music_mood`;
- поле `visual_style`;
- поле `thumbnail_prompt`;
- поле `posting_caption`;
- поле `hashtags`;
- поле `platform_adaptations`.
