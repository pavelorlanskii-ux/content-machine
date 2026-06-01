# Content Machine

Проект для создания контент-машины вертикального видео под Reels, TikTok и YouTube Shorts.

## Цель проекта

Создать модульную систему, которая помогает регулярно производить вертикальные видео:

1. искать идеи;
2. собирать контент-план;
3. писать сценарии;
4. готовить визуальную концепцию;
5. собирать видео;
6. готовить публикации;
7. анализировать результаты;
8. улучшать следующие ролики на основе данных.

## Текущий статус

Собран рабочий технический MVP-контур:

```text
идея
  ↓
n8n workflow
  ↓
OpenAI / LLM Script Generator
  ↓
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
videoId
  ↓
Wait 60 seconds
  ↓
status ready
  ↓
result JSON
  ↓
готовый mp4
```

## Подтверждено

- n8n работает локально в Docker;
- OpenAI node генерирует сценарий;
- workflow собирает `video_payload`;
- workflow собирает `video_pipeline_json`;
- workflow собирает `short_video_maker_payload`;
- Short Video Maker поднят в Docker;
- Pexels API подключён как источник фоновых stock videos;
- n8n отправляет payload в Short Video Maker;
- Short Video Maker возвращает `videoId`;
- n8n ждёт 60 секунд перед проверкой статуса;
- n8n проверяет статус видео;
- результат сохраняется в `data/outputs`;
- готовый mp4 скачивается в `data/generated`.

## Основной workflow

Файл workflow:

```text
workflows/script-agent-prompt-builder.json
```

Текущая логика workflow:

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
```

## Ключевые артефакты

Сценарий:

```text
data/outputs/idea_001-script-output.md
```

Промежуточный video payload:

```text
data/outputs/idea_001-video-payload.json
```

Структурированный video pipeline JSON:

```text
data/outputs/idea_001-video-pipeline.json
```

Результат генерации видео:

```text
data/outputs/idea_001-short-video-result.json
```

Готовый mp4:

```text
data/generated/idea_001-short-video.mp4
```

Папка `data/generated/` не попадает в Git.

## Структура проекта

```text
content-machine/
  agents/
    промты и логика ИИ-агентов

  data/
    тестовые данные, идеи, результаты и локальные артефакты

  data/generated/
    сгенерированные видео
    не коммитится в Git

  data/outputs/
    markdown, json и другие результаты выполнения workflow

  data/prompts/
    собранные prompt-файлы

  docs/
    документация, контракты данных, технические заметки

  scripts/
    вспомогательные скрипты

  services/
    docker-compose файлы и настройки внешних сервисов

  workflows/
    n8n workflow и автоматизации
```

## Локальные сервисы

### n8n

Адрес:

```text
http://localhost:5678
```

Docker Compose:

```text
services/n8n/docker-compose.yml
```

В контейнер n8n смонтирована локальная папка проекта:

```text
content-machine/data
  ↓
/files/data
```

Это нужно, чтобы n8n видел локальные данные проекта.

### Short Video Maker

Адрес:

```text
http://localhost:3123
```

Docker Compose:

```text
services/short-video-maker/docker-compose.yml
```

Сервис использует:

- Pexels API для фоновых stock videos;
- Kokoro TTS для озвучки;
- Remotion и ffmpeg для сборки mp4.

### Public storyboard frames for n8n

Higgsfield DoP needs a public URL for storyboard frames that n8n saves in `data/storyboard-frames`.
For local runs, serve the `data` folder and expose it with a temporary localtunnel URL:

```sh
python3 -m http.server 8080 --bind 127.0.0.1 --directory data
```

In another terminal:

```sh
npx localtunnel --port 8080 --local-host 127.0.0.1
```

Put the generated URL in local `.env`:

```env
PUBLIC_FILES_BASE_URL=https://your-current-url.loca.lt
```

If n8n is already running, restart or recreate the container so it receives the new environment value:

```sh
docker compose -f services/n8n/docker-compose.yml up -d --force-recreate
```

Then import `workflows/short-video-pipeline.json` into n8n. Temporary tunnel URLs should only live in local `.env` and must not be committed.

### Higgsfield DoP status polling

Higgsfield DoP scene renders are polled until all scenes are completed before the workflow normalizes scene video results. The default polling window is 10 attempts with a 60 second wait between attempts. If any scene is still queued or in progress after the max attempts, the workflow fails with pending scene diagnostics that include scene ID, request ID, and status.

### Creative control contracts

The creative pipeline now selects one trend format, then carries character, object, and world bibles through the storyboard and DoP stages to preserve continuity. Each scene includes a validated scene contract before DoP calls, and DoP prompts are written as short motion contracts to reduce object morphing, object substitution, and identity drift.

## Переменные окружения

Пример переменных окружения:

```text
.env.example
```

Локальный `.env` не коммитится.

Нужные переменные:

```env
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_MODEL=gpt-4.1-mini
PEXELS_API_KEY=your_pexels_api_key_here
SHORT_VIDEO_MAKER_URL=http://localhost:3123
POSTIZ_URL=http://localhost:5000
PUBLIC_FILES_BASE_URL=https://your-current-url.loca.lt
```

`PEXELS_API_KEY` нужен, потому что Short Video Maker берёт фоновые видео из Pexels.

## Short Video Maker API

Healthcheck:

```text
GET /health
```

Создание видео:

```text
POST /api/short-video
```

Проверка статуса:

```text
GET /api/short-video/{videoId}/status
```

Получение mp4:

```text
GET /api/short-video/{videoId}
```

Список голосов:

```text
GET /api/voices
```

Список музыкальных тегов:

```text
GET /api/music-tags
```

Документация:

```text
docs/short-video-maker-api-contract.md
services/short-video-maker/README.md
```

## Проверочные команды

Проверить контейнеры:

```bash
docker ps
```

Проверить Short Video Maker:

```bash
curl -s http://localhost:3123/health
```

Ожидаемый ответ:

```json
{"status":"ok"}
```

Проверить статус видео:

```bash
curl -s http://localhost:3123/api/short-video/{videoId}/status
```

Ожидаемый ответ после рендера:

```json
{"status":"ready"}
```

Скачать mp4:

```bash
curl -L "http://localhost:3123/api/short-video/{videoId}" \
  -o data/generated/idea_001-short-video.mp4
```

Проверить тип файла:

```bash
file data/generated/idea_001-short-video.mp4
```

Ожидаемый тип:

```text
ISO Media, MP4
```

## Текущий пример результата

Файл:

```text
data/outputs/idea_001-short-video-result.json
```

Пример структуры:

```json
[
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
]
```

## Ограничения MVP

### Видеоряд

Short Video Maker не генерирует видео с нуля. Он берёт stock videos из Pexels по `searchTerms`.

Из-за этого визуальная релевантность может быть слабой: запрос про хоккейную форму может вернуть спортзал, баскетбол или абстрактный спортивный футаж.

### Озвучка

Текущий TTS слой ориентирован на Kokoro voices. Русская озвучка пока не production-quality.

Для production-версии нужно отдельно подключить русский TTS, например Yandex SpeechKit, ElevenLabs или другой TTS-сервис.

### Сохранение результата

`Read/Write Files from Disk` в текущей сборке n8n блокирует прямую запись файла.

Поэтому result JSON сейчас сохраняется через:

```text
Convert Result JSON to File
  ↓
Download
  ↓
ручная замена файла в data/outputs
```

### Ожидание рендера

После `POST /api/short-video` видео создаётся не мгновенно. Поэтому в workflow добавлен node:

```text
Wait for Short Video Render
```

Текущая задержка:

```text
60 seconds
```

Если ролик длиннее или сервис загружен, может потребоваться 120 секунд или цикл проверки статуса.

## Следующие этапы

### Пакет O. AI Video Provider вместо Pexels

- заменить Pexels как основной visual layer;
- добавить Storyboard Agent;
- добавить Visual Bible Agent;
- добавить Scene Package Builder;
- рассмотреть Higgsfield как первого AI Video Provider;
- оставить Pexels только как fallback.

### Пакет P. Автоматическое ожидание статуса

- заменить фиксированный `Wait 60 seconds` на цикл проверки:
  - check status;
  - if processing, wait;
  - repeat;
  - if ready, finalize;
  - if failed, return error.

### Пакет Q. Локальная библиотека футажей

- добавить `data/assets/backgrounds`;
- использовать локальные видео вместо случайного Pexels;
- сделать controlled stock layer.

### Пакет R. Production video stack

- подключить русский TTS;
- добавить брендированные субтитры;
- добавить шаблоны оформления;
- рассмотреть ComfyUI / Runway / Kling / Pika или другой visual generation stack.

### Пакет S. Publishing layer

- подготовить payload для Postiz;
- сформировать title, caption, hashtags;
- подключить календарь публикаций;
- добавить аналитику.
