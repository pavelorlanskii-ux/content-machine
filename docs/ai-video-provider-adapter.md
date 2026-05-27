# AI Video Provider Adapter

Документ описывает целевую архитектуру замены Pexels stock footage на AI video generation layer.

## Назначение

Текущий MVP использует Short Video Maker и Pexels:

```text
video_pipeline_json
  ↓
short_video_maker_payload
  ↓
Pexels stock footage
  ↓
TTS + captions + Remotion
  ↓
final mp4
```

Это работает технически, но не даёт стабильного визуального качества. Pexels подбирает случайные stock videos по `searchTerms`, поэтому сцены могут быть нерелевантными.

Целевая архитектура должна заменить Pexels-слой на AI Video Provider:

```text
video_pipeline_json
  ↓
Storyboard Agent
  ↓
Visual Bible Agent
  ↓
Scene Package Builder
  ↓
AI Video Provider
  ↓
scene videos
  ↓
Final Assembly Layer
  ↓
final mp4
```

## Основной принцип

Не генерировать всё видео одним длинным промтом.

Правильный подход:

```text
1 сцена = 1 scene package = 1 AI video generation task
```

Это нужно для стабильности:

- легче контролировать качество;
- можно перегенерировать только плохую сцену;
- проще удерживать стиль;
- проще проверять результат;
- дешевле исправлять ошибки;
- легче подключать разные AI video providers.

## Целевая цепочка

```text
Idea Input
  ↓
Script Agent
  ↓
Video Pipeline JSON
  ↓
Storyboard Agent
  ↓
Visual Bible Agent
  ↓
Scene Package Builder
  ↓
AI Video Provider Adapter
  ↓
AI Scene Generation Tasks
  ↓
Scene QA Agent
  ↓
Final Assembly Layer
  ↓
Final Vertical Video
```

## Роль агентов

### 1. Script Agent

Отвечает за смысловую структуру ролика:

- хук;
- основная мысль;
- сцены;
- voiceover;
- экранный текст;
- CTA;
- длительность.

### 2. Storyboard Agent

Отвечает за режиссёрскую структуру.

Он преобразует текстовые сцены в раскадровку:

- что происходит в кадре;
- какой план;
- какое движение камеры;
- какая композиция;
- какие объекты должны быть в сцене;
- какой эмоциональный тон;
- какой on-screen text.

### 3. Visual Bible Agent

Отвечает за визуальную консистентность проекта.

Он формирует единый visual bible:

- стиль ролика;
- персонажи;
- одежда;
- цвета;
- свет;
- окружение;
- визуальные ограничения;
- что нельзя менять между сценами.

### 4. Character / Object Bible

Если в ролике есть персонажи, маскоты, форма, продукт или объект, они должны быть описаны отдельно.

Пример:

```json
{
  "characters": [
    {
      "character_id": "creative_director_01",
      "role": "sports creative director",
      "appearance": "30-35 years old, focused, modern sports marketing look",
      "clothing": "dark hoodie, minimal sports branding",
      "consistency_rules": [
        "same face across scenes",
        "same clothing",
        "same proportions"
      ]
    }
  ],
  "objects": [
    {
      "object_id": "hockey_jersey_01",
      "type": "hockey jersey",
      "description": "premium black and white hockey jersey with teal and orange club accents",
      "consistency_rules": [
        "keep hockey jersey silhouette",
        "avoid football shirt shape",
        "avoid basketball uniform shape"
      ]
    }
  ]
}
```

### 5. Scene Package Builder

Объединяет данные в пакет для генерации каждой сцены.

Каждая сцена должна стать отдельным JSON-объектом:

```json
{
  "scene_id": "scene_01",
  "duration_sec": 3,
  "voiceover_text": "Форму хоккейного клуба можно придумать не за недели, а за минуты.",
  "on_screen_text": "Форма клуба за минуты",
  "visual_prompt": "A cinematic close-up of a premium hockey jersey on a design table, sports-tech studio, AI interface glow in the background",
  "negative_prompt": "basketball, football, soccer, random gym, low quality, distorted text, extra fingers, wrong jersey shape",
  "camera_prompt": "fast push-in, shallow depth of field, vertical 9:16",
  "style_reference": "visual_bible_001",
  "character_reference": null,
  "object_reference": "hockey_jersey_01",
  "output_format": "mp4",
  "aspect_ratio": "9:16"
}
```

### 6. AI Video Provider Adapter

Единый слой для подключения разных AI video providers.

Провайдеры могут быть:

- Higgsfield;
- Runway;
- Luma;
- Kling;
- Pika;
- другой API.

Адаптер должен скрывать различия между API.

Универсальный flow:

```text
create generation task
  ↓
receive task_id
  ↓
wait
  ↓
check status
  ↓
receive result_url
  ↓
download scene video
```

### 7. Scene QA Agent

Проверяет каждую сгенерированную сцену.

Критерии:

- сцена соответствует storyboard;
- стиль соответствует visual bible;
- нет нерелевантного спорта;
- нет грубых артефактов;
- нет случайных логотипов;
- нет сломанного текста;
- объект похож на хоккейную форму;
- длительность примерно соответствует задаче.

Выход:

```json
{
  "scene_id": "scene_01",
  "qa_status": "approved",
  "issues": [],
  "retry_required": false
}
```

Если сцена плохая:

```json
{
  "scene_id": "scene_01",
  "qa_status": "rejected",
  "issues": [
    "scene shows basketball court instead of hockey jersey",
    "visual style does not match sports-tech premium look"
  ],
  "retry_required": true,
  "retry_prompt_adjustments": [
    "explicitly mention hockey jersey",
    "exclude basketball court",
    "use close-up product shot"
  ]
}
```

### 8. Final Assembly Layer

Собирает финальный ролик из отдельных сцен:

```text
scene_01.mp4
scene_02.mp4
scene_03.mp4
  ↓
voiceover
  ↓
captions
  ↓
music
  ↓
transitions
  ↓
final mp4
```

На текущем этапе сборку может выполнять Short Video Maker или отдельный ffmpeg/Remotion layer.

## Почему Pexels заменяется

Pexels подходит для MVP, но не подходит для production-контента, где важны:

- конкретный визуальный стиль;
- управляемые сцены;
- персонажи;
- фирменные объекты;
- спортивная айдентика;
- повторяемость результата;
- стабильность между сценами.

Pexels остаётся fallback-вариантом:

```text
AI video provider unavailable
  ↓
fallback to Pexels stock footage
```

## Higgsfield как первый кандидат

Higgsfield рассматривается как первый кандидат для AI Video Provider.

Причины:

- ориентирован на AI video;
- может быть полезен для social-first вертикальных роликов;
- потенциально подходит для image-to-video и stylized ad scenes;
- может заменить случайные stock videos управляемыми AI-generated сценами.

Технически нужно проверить:

- есть ли доступ к API key;
- какой API base URL;
- какие режимы доступны: text-to-video, image-to-video, video-to-video;
- есть ли async task flow;
- как получать `task_id`;
- как проверять status;
- как получать direct mp4 URL;
- есть ли лимиты и стоимость генерации.

## Переменные окружения

Целевые переменные:

```env
AI_VIDEO_PROVIDER=higgsfield
HIGGSFIELD_API_KEY=your_higgsfield_api_key_here
HIGGSFIELD_API_BASE_URL=your_higgsfield_api_base_url_here
```

В будущем можно добавить:

```env
RUNWAY_API_KEY=your_runway_api_key_here
LUMA_API_KEY=your_luma_api_key_here
KLING_API_KEY=your_kling_api_key_here
```

## Новые workflow node

Целевой n8n pipeline:

```text
Build Video Pipeline JSON
  ↓
Build Storyboard
  ↓
Build Visual Bible
  ↓
Build Scene Packages
  ↓
Create AI Video Scene
  ↓
Wait for AI Scene Render
  ↓
Check AI Scene Status
  ↓
Normalize AI Scene Result
  ↓
Scene QA
  ↓
Assemble Final Video
```

## MVP 2.0 scope

Минимальный следующий этап:

1. создать `Storyboard Agent`;
2. создать `Visual Bible Agent`;
3. создать `Scene Package Builder`;
4. сделать 1 тестовый `scene_package`;
5. вручную отправить scene prompt в Higgsfield;
6. получить scene video;
7. сохранить результат в `data/generated/scenes`;
8. описать дальнейшую автоматизацию.

## Выходные файлы будущего этапа

```text
data/outputs/idea_001-storyboard.json
data/outputs/idea_001-visual-bible.json
data/outputs/idea_001-scene-packages.json
data/generated/scenes/idea_001-scene_01.mp4
data/generated/idea_001-final-ai-video.mp4
```

## Ограничения

- API Higgsfield нужно проверить отдельно;
- качество генерации зависит от модели;
- image-to-video может быть стабильнее, чем text-to-video;
- для персонажей и формы нужны reference images;
- для стабильного результата потребуется Scene QA;
- стоимость генерации может быть высокой;
- генерация по сценам увеличивает количество API calls.

## Следующие шаги

1. Обновить `.env.example`.
2. Добавить папку `data/generated/scenes`.
3. Добавить папку `data/assets/references`.
4. Создать `agents/storyboard-agent.md`.
5. Создать `agents/visual-bible-agent.md`.
6. Создать `agents/scene-package-builder.md`.
7. Подготовить тестовый storyboard для `idea_001`.
8. Проверить Higgsfield API вручную.
9. После проверки API добавить n8n-интеграцию.
