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

Подтверждено:

n8n работает локально в Docker;
OpenAI node генерирует сценарий;
workflow собирает video_payload;
workflow собирает video_pipeline_json;
workflow собирает short_video_maker_payload;
Short Video Maker поднят в Docker;
Pexels API подключён как источник фоновых stock videos;
n8n отправляет payload в Short Video Maker;
Short Video Maker возвращает videoId;
n8n ждёт 60 секунд перед проверкой статуса;
n8n проверяет статус видео;
результат сохраняется в data/outputs;
готовый mp4 скачивается в data/generated.
Основной workflow

Файл workflow:

workflows/script-agent-prompt-builder.json

Текущая логика workflow:

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
Ключевые артефакты

Сценарий:

data/outputs/idea_001-script-output.md

Промежуточный video payload:

data/outputs/idea_001-video-payload.json

Структурированный video pipeline JSON:

data/outputs/idea_001-video-pipeline.json

Результат генерации видео:

data/outputs/idea_001-short-video-result.json

Готовый mp4:

data/generated/idea_001-short-video.mp4

Папка data/generated/ не попадает в Git.

Структура проекта
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
Локальные сервисы
n8n

Адрес:

http://localhost:5678

Docker Compose:

services/n8n/docker-compose.yml

В контейнер n8n смонтирована локальная папка проекта:

content-machine/data
  ↓
/files/data

Это нужно, чтобы n8n видел локальные данные проекта.

Short Video Maker

Адрес:

http://localhost:3123

Docker Compose:

services/short-video-maker/docker-compose.yml

Сервис использует:

Pexels API для фоновых stock videos;
Kokoro TTS для озвучки;
Remotion и ffmpeg для сборки mp4.
Переменные окружения

Пример переменных окружения:

.env.example

Локальный .env не коммитится.

Нужные переменные:

OPENAI_API_KEY=your_openai_api_key_here
OPENAI_MODEL=gpt-4.1-mini
PEXELS_API_KEY=your_pexels_api_key_here
SHORT_VIDEO_MAKER_URL=http://localhost:3123
POSTIZ_URL=http://localhost:5000

PEXELS_API_KEY нужен, потому что Short Video Maker берёт фоновые видео из Pexels.

Short Video Maker API

Healthcheck:

GET /health

Создание видео:

POST /api/short-video

Проверка статуса:

GET /api/short-video/{videoId}/status

Получение mp4:

GET /api/short-video/{videoId}

Список голосов:

GET /api/voices

Список музыкальных тегов:

GET /api/music-tags

Документация:

docs/short-video-maker-api-contract.md
services/short-video-maker/README.md
Проверочные команды

Проверить контейнеры:

docker ps

Проверить Short Video Maker:

curl -s http://localhost:3123/health

Ожидаемый ответ:

{"status":"ok"}

Проверить статус видео:

curl -s http://localhost:3123/api/short-video/{videoId}/status

Ожидаемый ответ после рендера:

{"status":"ready"}

Скачать mp4:

curl -L "http://localhost:3123/api/short-video/{videoId}" \
  -o data/generated/idea_001-short-video.mp4

Проверить тип файла:

file data/generated/idea_001-short-video.mp4

Ожидаемый тип:

ISO Media, MP4
Текущий пример результата

Файл:

data/outputs/idea_001-short-video-result.json

Пример структуры:

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
Ограничения MVP
Видеоряд

Short Video Maker не генерирует видео с нуля. Он берёт stock videos из Pexels по searchTerms.

Из-за этого визуальная релевантность может быть слабой: запрос про хоккейную форму может вернуть спортзал, баскетбол или абстрактный спортивный футаж.

Озвучка

Текущий TTS слой ориентирован на Kokoro voices. Русская озвучка пока не production-quality.

Для production-версии нужно отдельно подключить русский TTS, например Yandex SpeechKit, ElevenLabs или другой TTS-сервис.

Сохранение результата

Read/Write Files from Disk в текущей сборке n8n блокирует прямую запись файла.

Поэтому result JSON сейчас сохраняется через:

Convert Result JSON to File
  ↓
Download
  ↓
ручная замена файла в data/outputs
Ожидание рендера

После POST /api/short-video видео создаётся не мгновенно. Поэтому в workflow добавлен node:

Wait for Short Video Render

Текущая задержка:

60 seconds

Если ролик длиннее или сервис загружен, может потребоваться 120 секунд или цикл проверки статуса.

Следующие этапы
Пакет O. Улучшение качества payload
улучшить генерацию searchTerms;
добавить Visual/Search Agent;
выбирать music по тону ролика;
выбирать voice по языку и стилю;
добавить контроль длительности сцен.
Пакет P. Автоматическое ожидание статуса
заменить фиксированный Wait 60 seconds на цикл проверки:
check status;
if processing, wait;
repeat;
if ready, finalize;
if failed, return error.
Пакет Q. Локальная библиотека футажей
добавить data/assets/backgrounds;
использовать локальные видео вместо случайного Pexels;
сделать controlled stock layer.
Пакет R. Production video stack
подключить русский TTS;
добавить брендированные субтитры;
добавить шаблоны оформления;
рассмотреть ComfyUI / Runway / Kling / Pika или другой visual generation stack.
Пакет S. Publishing layer
подготовить payload для Postiz;
сформировать title, caption, hashtags;
подключить календарь публикаций;
добавить аналитику.
