# Script to Video Pipeline Adapter

Документ описывает преобразование сценария и `video_payload` в структуру, пригодную для видеогенерации.

## Назначение

Адаптер нужен, чтобы перевести результат Script Agent в формат, который дальше может использоваться видеосборщиком.

Текущая цепочка:

```text
script_markdown
  ↓
video_payload
  ↓
video_pipeline_json
  ↓
short_video_maker_payload
  ↓
Short Video Maker API
  ↓
generated mp4
Входные данные

На вход адаптер получает video_payload.

Пример:

{
  "id": "idea_001",
  "status": "ready_for_video_pipeline",
  "video_payload": {
    "id": "idea_001",
    "source": "script-agent",
    "script_format": "markdown",
    "script_markdown": "...",
    "platform": "Reels",
    "duration_seconds": 30,
    "language": "ru"
  }
}
Первый уровень преобразования

Из video_payload.script_markdown формируется video_pipeline_json.

Цель video_pipeline_json: выделить из сценария структурированные блоки, которые можно использовать для сборки видео.

Пример структуры:

{
  "id": "idea_001",
  "title": "Форма хоккейного клуба за минуты",
  "duration_seconds": 30,
  "language": "ru",
  "aspect_ratio": "9:16",
  "resolution": "1080x1920",
  "voiceover": "...",
  "captions": [],
  "scenes": [
    {
      "scene": 1,
      "block": "Hook",
      "duration": "0-3 сек",
      "content": "Быстро заявляем идею: ИИ помогает стартовать дизайн формы без долгих поисков",
      "visual": "Быстрая смена кадров: референс, цвета, форма",
      "on_screen_text": "Форма клуба за минуты"
    }
  ],
  "cta": "Сохрани, если хочешь быстрее придумывать дизайн для спорта и брендов.",
  "source_script_format": "markdown"
}
Второй уровень преобразования

Из video_pipeline_json формируется short_video_maker_payload.

Этот payload отправляется в Short Video Maker через endpoint:

POST /api/short-video

Целевая структура:

{
  "scenes": [
    {
      "text": "Быстро заявляем идею: ИИ помогает стартовать дизайн формы без долгих поисков",
      "searchTerms": [
        "hockey",
        "sports design",
        "technology"
      ]
    }
  ],
  "config": {
    "paddingBack": 1500,
    "music": "excited",
    "captionPosition": "bottom",
    "captionBackgroundColor": "blue",
    "voice": "af_heart",
    "orientation": "portrait",
    "musicVolume": "medium"
  }
}
Правила преобразования в video_pipeline_json
id

Берётся из исходной идеи или video_payload.id.

idea_001
title

Берётся из заголовка сценария или генерируется из темы идеи.

duration_seconds

Берётся из исходной идеи.

Для тестового MVP:

30
language

Для текущего MVP:

ru
aspect_ratio

Для Reels, TikTok и YouTube Shorts:

9:16
resolution

Для вертикального видео:

1080x1920
voiceover

Берётся из текстовой части сценария.

captions

На текущем этапе может быть пустым массивом или набором коротких экранных фраз.

scenes

Сцены формируются из смысловых блоков сценария.

Каждая сцена должна содержать:

Поле	Назначение
scene	номер сцены
block	тип блока: Hook, Context, Process, CTA
duration	примерная длительность
content	основной текст сцены
visual	описание визуального ряда
on_screen_text	экранный текст
cta

Финальный призыв к действию.

Правила преобразования в short_video_maker_payload
scenes[].text

Основное правило:

video_pipeline_json.scenes[].content

Fallback:

video_pipeline_json.scenes[].on_screen_text

Если оба поля пустые:

video_pipeline_json.voiceover
scenes[].searchTerms

На текущем этапе используются простые правила:

Если сцена связана с хоккеем, формой или спортивным дизайном:

[
  "hockey",
  "sports design",
  "technology"
]

Если сцена связана с ИИ или генерацией:

[
  "artificial intelligence",
  "creative process",
  "technology"
]

Fallback:

[
  "sports",
  "technology",
  "creative process"
]

В будущей версии searchTerms должен формировать отдельный Visual/Search Agent.

config.paddingBack

Текущее значение:

1500
config.music

Текущее значение:

excited
config.captionPosition

Текущее значение:

bottom
config.captionBackgroundColor

Текущее значение:

blue
config.voice

Текущее значение:

af_heart
config.orientation

Для вертикальных платформ:

portrait
config.musicVolume

Текущее значение:

medium
n8n workflow

Текущие node:

Build Video Payload
  ↓
Build Video Pipeline JSON
  ↓
Build Short Video Maker Payload
  ↓
Create Short Video
  ↓
Normalize Short Video Response
  ↓
Wait for Short Video Render
  ↓
Check Short Video Status
  ↓
Finalize Short Video Result
  ↓
Convert Result JSON to File
Результат Short Video Maker

После POST /api/short-video сервис возвращает:

{
  "videoId": "cmpmziptu000635s6d4pd1j3h"
}

После ожидания workflow проверяет статус:

GET /api/short-video/{videoId}/status

Ожидаемый ответ:

{
  "status": "ready"
}

После этого workflow формирует итоговый объект:

{
  "id": "idea_001",
  "status": "short_video_ready",
  "render_status": "ready",
  "videoId": "cmpmziptu000635s6d4pd1j3h",
  "status_url": "http://host.docker.internal:3123/api/short-video/cmpmziptu000635s6d4pd1j3h/status",
  "video_url": "http://host.docker.internal:3123/api/short-video/cmpmziptu000635s6d4pd1j3h",
  "browser_url": "http://localhost:3123/video/cmpmziptu000635s6d4pd1j3h",
  "source": "short-video-maker",
  "completed_at": "2026-05-27T07:26:04.054Z"
}
Выходные файлы

Текущие sample-файлы:

data/outputs/idea_001-video-payload.json
data/outputs/idea_001-video-pipeline.json
data/outputs/idea_001-short-video-result.json

Готовый mp4 скачивается отдельно:

data/generated/idea_001-short-video.mp4

Папка data/generated/ не коммитится.

## Ограничения текущего адаптера

- `searchTerms` пока формируются простыми правилами;
- визуальный ряд зависит от Pexels;
- Pexels может вернуть нерелевантный stock footage;
- русский TTS пока не production-quality;
- длительность сцен пока не рассчитывается точно;
- ожидание рендера реализовано фиксированной задержкой 60 секунд;
- нет цикла повторной проверки статуса;
- нет обработки `failed` кроме финального статуса.

## Следующие улучшения

1. Добавить Visual/Search Agent для генерации точных `searchTerms`.
2. Добавить контроль длительности сцен.
3. Добавить цикл проверки статуса видео.
4. Добавить обработку ошибок `failed`.
5. Добавить локальную библиотеку футажей.
6. Добавить production TTS для русского языка.
7. Добавить branded captions template.
8. Добавить publishing payload для Postiz.
