# Short Video Maker Service

Short Video Maker используется в проекте Content Machine как технический слой сборки вертикальных видео.

## Роль в Content Machine

Сервис получает подготовленный payload со сценами, текстом, поисковыми запросами для фонов, настройками музыки, субтитров и голоса.

Цепочка:

```text
video_pipeline_json
  ↓
Build Short Video Maker Payload
  ↓
short_video_maker_payload
  ↓
POST /api/short-video
  ↓
videoId
  ↓
GET /api/short-video/{videoId}/status
  ↓
GET /api/short-video/{videoId}
  ↓
generated mp4
```

## Локальный адрес

```text
http://localhost:3123
```

Healthcheck:

```bash
curl -s http://localhost:3123/health
```

Ожидаемый ответ:

```json
{"status":"ok"}
```

## Docker Compose

Файл запуска:

```text
services/short-video-maker/docker-compose.yml
```

Контейнер:

```text
content-machine-short-video-maker
```

Проверка:

```bash
docker ps
```

## Переменные окружения

Для работы нужен Pexels API key:

```env
PEXELS_API_KEY=your_pexels_api_key_here
```

Ключ хранится только в локальном `.env` и не коммитится.

Pexels нужен не для генерации видео, а как источник фоновых stock videos по `searchTerms`.

## Основные API endpoints

### Создание видео

```text
POST /api/short-video
```

Пример payload:

```json
{
  "scenes": [
    {
      "text": "Форму хоккейного клуба можно придумать не за недели, а за минуты.",
      "searchTerms": ["hockey", "sports design", "technology"]
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
```

Успешный ответ:

```json
{
  "videoId": "cmpmziptu000635s6d4pd1j3h"
}
```

### Проверка статуса

```text
GET /api/short-video/{videoId}/status
```

Ожидаемый ответ после рендера:

```json
{
  "status": "ready"
}
```

### Получение mp4

```text
GET /api/short-video/{videoId}
```

Ожидаемые headers:

```http
Content-Type: video/mp4
Content-Disposition: inline; filename={videoId}.mp4
```

### Список голосов

```text
GET /api/voices
```

### Список музыкальных тегов

```text
GET /api/music-tags
```

## Проверенный пример

Проверенный `videoId`:

```text
cmpmziptu000635s6d4pd1j3h
```

Проверка статуса:

```bash
curl -s http://localhost:3123/api/short-video/cmpmziptu000635s6d4pd1j3h/status
```

Получение mp4:

```bash
curl -L "http://localhost:3123/api/short-video/cmpmziptu000635s6d4pd1j3h" \
  -o data/generated/idea_001-short-video.mp4
```

## Подключение в n8n

Если n8n работает в Docker, внутри n8n нельзя обращаться к Short Video Maker через `localhost`.

Правильный адрес из n8n:

```text
http://host.docker.internal:3123
```

Создание видео из n8n:

```text
POST http://host.docker.internal:3123/api/short-video
```

Проверка статуса из n8n:

```text
GET http://host.docker.internal:3123/api/short-video/{{$json.videoId}}/status
```

Получение mp4 из n8n:

```text
GET http://host.docker.internal:3123/api/short-video/{{$json.videoId}}
```

## Текущая интеграция

В основном workflow используются node:

```text
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
```

Результат сохраняется в:

```text
data/outputs/idea_001-short-video-result.json
```

Готовый mp4 скачивается в:

```text
data/generated/idea_001-short-video.mp4
```

## Ограничения

- сервис не генерирует видео с нуля;
- фоновые видео берутся из Pexels;
- Pexels может вернуть нерелевантный stock footage;
- текущие voices ориентированы на Kokoro TTS;
- русская озвучка пока не production-quality;
- для production-версии нужен отдельный русский TTS;
- для фирменных роликов потребуется отдельный visual generation stack или локальная библиотека футажей.

## Следующие улучшения

1. Улучшить генерацию `searchTerms`.
2. Добавить Visual/Search Agent.
3. Подключить production TTS для русского языка.
4. Добавить branded captions template.
5. Добавить локальную библиотеку футажей.
6. Добавить цикл проверки статуса вместо фиксированного ожидания.
7. Добавить обработку статуса `failed`.
8. Подготовить publishing payload для Postiz.
