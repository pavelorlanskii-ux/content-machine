# Script to Video Pipeline Adapter

Документ описывает адаптер, который преобразует `script_markdown` в строгий JSON для видеопайплайна.

## Назначение

`script_markdown` удобен для человека, но видеогенератору нужна более строгая структура данных.

Адаптер нужен как промежуточный слой между Script Agent и Video Generation Service.

Поток:

```text
Script Agent
  ↓
script_markdown
  ↓
Script to Video Pipeline Adapter
  ↓
video_pipeline_json
  ↓
Video Generation Service
```

## Входные данные

Адаптер принимает объект из финального выхода Script Agent workflow:

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

## Выходные данные

Адаптер должен вернуть объект:

```json
{
  "id": "idea_001",
  "status": "ready_for_video_pipeline",
  "video_pipeline_json": {
    "id": "idea_001",
    "title": "ИИ-дизайн формы за 30 секунд",
    "duration_seconds": 30,
    "language": "ru",
    "aspect_ratio": "9:16",
    "resolution": "1080x1920",
    "voiceover": "...",
    "captions": [
      "Форма клуба за 30 секунд",
      "Старт: референс + стиль клуба",
      "1 идея → 10 вариантов"
    ],
    "scenes": [
      {
        "scene": 1,
        "block": "Hook",
        "duration": "0-3 сек",
        "content": "Сильное заявление про скорость создания формы",
        "visual": "Быстрая смена кадров: пустая форма → референсы → готовый дизайн",
        "on_screen_text": "Форма клуба за 30 секунд"
      }
    ],
    "cta": "Сохрани, если хочешь быстрее придумывать дизайн для спорта и брендов.",
    "source_script_format": "markdown"
  }
}
```

## Целевые поля `video_pipeline_json`

| Поле | Тип | Описание |
|---|---|---|
| `id` | string | ID ролика |
| `title` | string | Название ролика |
| `duration_seconds` | number | Плановая длительность |
| `language` | string | Язык ролика |
| `aspect_ratio` | string | Соотношение сторон |
| `resolution` | string | Целевое разрешение |
| `voiceover` | string | Полный текст озвучки |
| `captions` | array | Список экранных надписей |
| `scenes` | array | Массив сцен |
| `cta` | string | Финальный призыв к действию |
| `source_script_format` | string | Исходный формат сценария |

## Структура объекта сцены

Каждая сцена в массиве `scenes` должна иметь структуру:

```json
{
  "scene": 1,
  "block": "Hook",
  "duration": "0-3 сек",
  "content": "Содержание сцены",
  "visual": "Описание визуала",
  "on_screen_text": "Текст на экране"
}
```

## Правила парсинга

### `title`

Берётся из блока Markdown:

```markdown
## 1. Название ролика
```

Первый непустой абзац после заголовка считается названием.

### `duration_seconds`

Берётся из:

```json
video_payload.duration_seconds
```

Если поле отсутствует, использовать значение по умолчанию:

```json
30
```

### `language`

Берётся из:

```json
video_payload.language
```

Если поле отсутствует, использовать:

```json
"ru"
```

### `aspect_ratio`

Берётся из:

```json
video_payload.aspect_ratio
```

Если поле отсутствует, использовать:

```json
"9:16"
```

### `resolution`

Берётся из:

```json
video_payload.video_requirements.resolution
```

Если поле отсутствует, использовать:

```json
"1080x1920"
```

### `voiceover`

Берётся из блока Markdown:

```markdown
## 5. Voiceover
```

Текст извлекается до следующего заголовка второго уровня:

```markdown
##
```

### `captions`

На первом этапе captions можно собирать двумя способами.

Приоритет 1: из блока:

```markdown
## 6. On-screen text
```

Приоритет 2: из колонки таблицы:

```markdown
Текст на экране
```

Если оба способа не сработали, вернуть пустой массив:

```json
[]
```

### `scenes`

На первом этапе сцены берутся из таблицы блока:

```markdown
## 4. Структура ролика
```

Ожидаемый формат таблицы:

```markdown
| Блок | Время | Содержание | Визуал | Текст на экране |
|---|---:|---|---|---|
| Hook | 0-3 сек | ... | ... | ... |
```

Каждая строка таблицы превращается в объект:

```json
{
  "scene": 1,
  "block": "Hook",
  "duration": "0-3 сек",
  "content": "...",
  "visual": "...",
  "on_screen_text": "..."
}
```

### `cta`

Берётся из блока Markdown:

```markdown
## 8. CTA
```

Первый непустой абзац после заголовка считается CTA.

Если блок не найден, использовать пустую строку:

```json
""
```

## Ограничения первой версии

Первая версия адаптера может быть простой.

Допустимо:

- использовать регулярные выражения;
- парсить только базовые Markdown-блоки;
- возвращать пустые массивы, если блоки не найдены;
- оставлять пустую строку для `cta`, если блок отсутствует;
- не обрабатывать сложные вложенные Markdown-структуры.

Недопустимо:

- терять `id`;
- возвращать невалидный JSON;
- ломать workflow, если часть Markdown-блоков отсутствует;
- смешивать `script_markdown` и `video_pipeline_json` в одном поле;
- удалять исходный сценарий до завершения video pipeline.

## Ошибки и fallback

Если `script_markdown` отсутствует, адаптер должен вернуть валидный объект с пустыми полями:

```json
{
  "id": "idea_001",
  "status": "ready_for_video_pipeline",
  "video_pipeline_json": {
    "id": "idea_001",
    "title": "",
    "duration_seconds": 30,
    "language": "ru",
    "aspect_ratio": "9:16",
    "resolution": "1080x1920",
    "voiceover": "",
    "captions": [],
    "scenes": [],
    "cta": "",
    "source_script_format": "markdown"
  }
}
```

## Будущая цепочка n8n

После реализации адаптера workflow должен выглядеть так:

```text
Manual Trigger
  ↓
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
Build Video Pipeline JSON
```

## Следующий технический шаг

Реализовать адаптер в n8n через `Code` node:

```text
Build Video Pipeline JSON
```

Этот node должен:

1. принять `video_payload`;
2. извлечь `script_markdown`;
3. распарсить title, voiceover, captions, scenes и CTA;
4. вернуть `video_pipeline_json`;
5. сохранить исходный `id`.

## Следующие улучшения

В следующих версиях адаптера нужно добавить:

- более устойчивый Markdown parser;
- отдельное поле `music_mood`;
- отдельное поле `visual_style`;
- отдельное поле `thumbnail_prompt`;
- отдельное поле `posting_caption`;
- отдельное поле `hashtags`;
- адаптацию под конкретные платформы: Reels, TikTok, YouTube Shorts;
- проверку длительности сцен;
- проверку количества caption-строк;
- проверку безопасных зон для интерфейсов платформ.
