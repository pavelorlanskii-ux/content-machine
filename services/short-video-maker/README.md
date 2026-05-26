# Short Video Maker Service

Этот сервис планируется использовать как слой сборки вертикальных видео в проекте Content Machine.

## Роль в Content Machine

Short Video Maker должен закрывать этап генерации простого вертикального видео после того, как Script Agent подготовил структурированный `video_pipeline_json`.

Планируемая цепочка:

```text
Idea Input
  ↓
Script Agent
  ↓
video_payload
  ↓
video_pipeline_json
  ↓
Short Video Maker Adapter
  ↓
Short Video Maker REST API / MCP
  ↓
generated vertical video
```

## Что сервис должен получать на вход

На вход сервису или адаптеру передаётся объект:

```json
{
  "id": "idea_001",
  "status": "ready_for_video_pipeline",
  "video_pipeline_json": {
    "id": "idea_001",
    "title": "Форма хоккейного клуба за минуты",
    "duration_seconds": 30,
    "language": "ru",
    "aspect_ratio": "9:16",
    "resolution": "1080x1920",
    "voiceover": "...",
    "captions": [
      "Форма клуба за минуты",
      "Берём референс",
      "ИИ генерирует варианты"
    ],
    "scenes": [
      {
        "scene": 1,
        "block": "Hook",
        "duration": "0-3 сек",
        "content": "Сильное заявление про скорость создания формы",
        "visual": "Быстрая смена кадров: пустая форма, референсы, готовый дизайн",
        "on_screen_text": "Форма клуба за минуты"
      }
    ],
    "cta": "Сохрани, если хочешь быстрее придумывать дизайн для спорта и брендов.",
    "source_script_format": "markdown"
  }
}
```

## Что сервис должен вернуть

На выходе ожидается объект со ссылкой или локальным путём к готовому видео:

```json
{
  "id": "idea_001",
  "status": "video_generated",
  "video": {
    "format": "mp4",
    "aspect_ratio": "9:16",
    "resolution": "1080x1920",
    "duration_seconds": 30,
    "local_path": "data/generated/idea_001.mp4",
    "public_url": null
  },
  "source": "short-video-maker"
}
```

## Возможности Short Video Maker

Short Video Maker планируется использовать как technical video assembly layer.

Потенциальные функции:

- создание short-form video из текстового ввода;
- text-to-speech;
- автоматические captions;
- подбор background videos;
- добавление background music;
- сборка видео через Remotion;
- работа через REST API;
- работа как MCP server.

## Ограничения

На текущем этапе важно учитывать:

- сервис не генерирует видео с нуля по image prompt;
- background videos могут браться из внешних источников, например Pexels;
- текущий TTS-слой может быть ориентирован на English voiceover;
- для русского production-пайплайна может понадобиться отдельный TTS;
- для финального качества могут понадобиться ElevenLabs, Yandex SpeechKit или другой TTS;
- сервис пока рассматривается как слой сборки, а не как финальное решение для визуально сложных AI-роликов;
- для роликов с персонажами, сценами и стабильным визуальным стилем позже может понадобиться ComfyUI или другой visual generation stack.

## Минимальные требования

Ориентировочные требования для локального запуска:

- Docker;
- internet connection;
- Pexels API key или другой источник background videos;
- минимум 3 GB RAM;
- желательно 4 GB RAM или больше;
- минимум 2 vCPU;
- минимум 5 GB disk space.

## Переменные окружения

Планируемые переменные:

```env
SHORT_VIDEO_MAKER_URL=http://localhost:3123
PEXELS_API_KEY=your_pexels_api_key_here
TTS_PROVIDER=default
VIDEO_OUTPUT_DIR=data/generated
```

## Место сервиса в структуре проекта

```text
content-machine/
  services/
    short-video-maker/
      README.md
      docker-compose.yml
```

## Будущий адаптер

Для подключения сервиса нужен отдельный адаптер:

```text
video_pipeline_json
  ↓
Short Video Maker Adapter
  ↓
short_video_maker_payload
```

Адаптер должен преобразовать:

```json
{
  "title": "...",
  "voiceover": "...",
  "captions": [],
  "scenes": []
}
```

в payload конкретного API Short Video Maker.

## Черновой формат `short_video_maker_payload`

Предварительный формат:

```json
{
  "id": "idea_001",
  "title": "Форма хоккейного клуба за минуты",
  "text": "Форму хоккейного клуба можно придумать не за недели, а за минуты...",
  "captions": [
    "Форма клуба за минуты",
    "Берём референс",
    "ИИ генерирует варианты"
  ],
  "duration_seconds": 30,
  "aspect_ratio": "9:16",
  "resolution": "1080x1920",
  "language": "ru",
  "background_video_query": "hockey design technology sports",
  "music_mood": "dynamic technology sport"
}
```

## Что нужно проверить перед запуском

Перед подключением в n8n нужно изучить:

1. как установить Short Video Maker локально;
2. есть ли официальный `docker-compose.yml`;
3. какой порт использует REST API;
4. какой endpoint отвечает за генерацию видео;
5. какой формат payload ожидает API;
6. где сохраняется готовое видео;
7. можно ли отключить или заменить TTS;
8. можно ли передавать свои captions;
9. можно ли задать вертикальный формат 9:16;
10. можно ли подключить русскую озвучку.

## План подключения

### Этап 1. Локальный запуск

- [ ] Изучить README оригинального проекта;
- [ ] Добавить рабочий `docker-compose.yml`;
- [ ] Запустить сервис локально;
- [ ] Проверить доступность API;
- [ ] Выполнить тестовую генерацию видео.

### Этап 2. Адаптер

- [ ] Создать документацию payload-адаптера;
- [ ] Создать n8n node `Build Short Video Maker Payload`;
- [ ] Преобразовать `video_pipeline_json` в payload сервиса;
- [ ] Проверить валидность payload.

### Этап 3. Интеграция с n8n

- [ ] Добавить HTTP Request node;
- [ ] Отправить payload в Short Video Maker;
- [ ] Получить ответ сервиса;
- [ ] Сохранить путь к видео;
- [ ] Передать результат дальше в publishing pipeline.

### Этап 4. Улучшения

- [ ] Подключить русский TTS;
- [ ] Добавить брендированные subtitles;
- [ ] Добавить шаблоны визуального оформления;
- [ ] Добавить сохранение видео в `data/generated`;
- [ ] Добавить обработку ошибок;
- [ ] Добавить повторную генерацию при ошибке.