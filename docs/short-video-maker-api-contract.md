# Short Video Maker API Contract

Документ фиксирует REST API Short Video Maker, который используется в проекте Content Machine для генерации вертикальных видео.

## Назначение

Short Video Maker принимает подготовленный payload со сценами, текстом озвучки, параметрами музыки, субтитров и ориентации видео.

В рамках проекта сервис используется как слой сборки видео после этапа `video_pipeline_json`.

Поток:

```text
Script Agent
  ↓
video_pipeline_json
  ↓
Build Short Video Maker Payload
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

## Базовый URL

Локальный адрес сервиса:

```text
http://localhost:3123
```

Healthcheck:

```http
GET /health
```

Проверенный ответ:

```json
{
  "status": "ok"
}
```

## Рабочие endpoints

### Создание видео

```http
POST /api/short-video
```

Назначение: запускает генерацию короткого вертикального видео.

Пример payload:

```json
{
  "scenes": [
    {
      "text": "Форму хоккейного клуба можно придумать не за недели, а за минуты.",
      "searchTerms": [
        "hockey",
        "sports design",
        "technology"
      ]
    },
    {
      "text": "Сначала берём референс, цвета клуба и настроение команды.",
      "searchTerms": [
        "hockey jersey",
        "sports team",
        "design"
      ]
    },
    {
      "text": "ИИ быстро собирает несколько вариантов, а человек выбирает сильный.",
      "searchTerms": [
        "artificial intelligence",
        "creative process",
        "sports"
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
```

Успешный ответ:

```json
{
  "videoId": "cmpmspp1e00002xpdc1bhcpj9"
}
```

### Проверка статуса видео

```http
GET /api/short-video/{videoId}/status
```

Пример:

```http
GET /api/short-video/cmpmspp1e00002xpdc1bhcpj9/status
```

Проверенный ответ:

```json
{
  "status": "ready"
}
```

Возможные статусы нужно уточнить отдельно по коду `ShortCreator`, но для текущего MVP подтверждён статус:

```text
ready
```

### Получение готового mp4

```http
GET /api/short-video/{videoId}
```

Пример:

```http
GET /api/short-video/cmpmspp1e00002xpdc1bhcpj9
```

Проверенный ответ:

```http
HTTP/1.1 200 OK
Content-Type: video/mp4
Content-Disposition: inline; filename=cmpmspp1e00002xpdc1bhcpj9.mp4
Content-Length: 39686000
```

Важно: правильный endpoint для mp4 начинается с `/api`.

Неверные варианты, которые возвращают frontend fallback или 404:

```text
/short-video/{videoId}
/videos/{videoId}.mp4
/data/videos/{videoId}.mp4
/api/videos/{videoId}.mp4
/api/short-video/{videoId}.mp4
```

### Список голосов

```http
GET /api/voices
```

Проверенный пример ответа:

```json
[
  "af_heart",
  "af_alloy",
  "af_aoede",
  "af_bella",
  "af_jessica",
  "af_kore",
  "af_nicole",
  "af_nova",
  "af_river",
  "af_sarah",
  "af_sky",
  "am_adam",
  "am_echo",
  "am_eric",
  "am_fenrir",
  "am_liam",
  "am_michael",
  "am_onyx",
  "am_puck",
  "am_santa"
]
```

Для первого MVP выбран голос:

```text
af_heart
```

### Список музыкальных тегов

```http
GET /api/music-tags
```

Проверенный пример ответа:

```json
[
  "melancholic",
  "chill",
  "uneasy",
  "excited",
  "euphoric/high",
  "dark",
  "sad",
  "happy",
  "angry",
  "hopeful",
  "contemplative",
  "funny/quirky"
]
```

Для первого MVP выбран тег:

```text
excited
```

### Список видео

```http
GET /api/short-videos
```

Назначение: возвращает список созданных видео.

Формат ответа нужно дополнительно проверить в n8n-интеграции.

### Удаление видео

```http
DELETE /api/short-video/{videoId}
```

Назначение: удаляет видео по `videoId`.

Пример:

```http
DELETE /api/short-video/cmpmspp1e00002xpdc1bhcpj9
```

Ожидаемый успешный ответ:

```json
{
  "success": true
}
```

## Проверенный тестовый videoId

```text
cmpmspp1e00002xpdc1bhcpj9
```

Локальный mp4-файл был найден внутри контейнера:

```text
/app/data/videos/cmpmspp1e00002xpdc1bhcpj9.mp4
```

Также файл был скопирован локально в проект:

```text
data/generated/cmpmspp1e00002xpdc1bhcpj9.mp4
```

Папка `data/generated/` находится в `.gitignore`, поэтому сгенерированные видео не попадают в Git.

## Требования к payload

### Поле `scenes`

Тип:

```text
array
```

Каждая сцена должна содержать:

| Поле | Тип | Описание |
|---|---|---|
| `text` | string | Текст сцены для озвучки и/или субтитров |
| `searchTerms` | array | Поисковые запросы для background video |

Пример сцены:

```json
{
  "text": "Форму хоккейного клуба можно придумать не за недели, а за минуты.",
  "searchTerms": [
    "hockey",
    "sports design",
    "technology"
  ]
}
```

### Поле `config`

Тип:

```text
object
```

Проверенные поля:

| Поле | Тип | Пример | Описание |
|---|---|---|---|
| `paddingBack` | number | `1500` | Дополнительный отступ в конце видео |
| `music` | string | `excited` | Музыкальный тег |
| `captionPosition` | string | `bottom` | Позиция субтитров |
| `captionBackgroundColor` | string | `blue` | Цвет фона субтитров |
| `voice` | string | `af_heart` | Голос TTS |
| `orientation` | string | `portrait` | Вертикальная ориентация |
| `musicVolume` | string | `medium` | Громкость музыки |

## Адаптация из `video_pipeline_json`

Входной объект Content Machine:

```json
{
  "id": "idea_001",
  "video_pipeline_json": {
    "id": "idea_001",
    "title": "Форма хоккейного клуба за минуты",
    "duration_seconds": 30,
    "voiceover": "...",
    "captions": [],
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
    "cta": "..."
  }
}
```

Целевой объект для Short Video Maker:

```json
{
  "scenes": [
    {
      "text": "Сильное заявление про скорость создания формы",
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
```

## Правила преобразования

### `scenes[].text`

На первом этапе брать из поля:

```text
video_pipeline_json.scenes[].content
```

Если `content` пустой, fallback:

```text
video_pipeline_json.scenes[].on_screen_text
```

Если и он пустой, fallback:

```text
video_pipeline_json.voiceover
```

### `scenes[].searchTerms`

На первом этапе использовать простой словарь по теме ролика.

Для тестового ролика про хоккейную форму:

```json
[
  "hockey",
  "sports design",
  "technology"
]
```

В будущей версии `searchTerms` должен генерировать отдельный Visual/Search Agent.

### `config.music`

На первом этапе:

```text
excited
```

### `config.voice`

На первом этапе:

```text
af_heart
```

### `config.orientation`

Для Reels, TikTok и YouTube Shorts:

```text
portrait
```

## n8n-интеграция

Следующие node в n8n:

```text
Build Video Pipeline JSON
  ↓
Build Short Video Maker Payload
  ↓
HTTP Request: Create Short Video
  ↓
Normalize Short Video Response
  ↓
Check Short Video Status
  ↓
Save Video URL
```

## HTTP Request node для создания видео

Метод:

```text
POST
```

URL:

```text
http://host.docker.internal:3123/api/short-video
```

Важно: если n8n работает в Docker, внутри контейнера `localhost` будет означать сам контейнер n8n, а не Mac. Поэтому для обращения к Short Video Maker с Mac-хоста нужно использовать:

```text
host.docker.internal
```

Если n8n и Short Video Maker будут в одной Docker-сети, можно будет использовать имя контейнера или service name.

Headers:

```json
{
  "Content-Type": "application/json"
}
```

Body:

```json
{
  "scenes": [],
  "config": {}
}
```

## HTTP Request node для проверки статуса

Метод:

```text
GET
```

URL:

```text
http://host.docker.internal:3123/api/short-video/{{$json.videoId}}/status
```

Ожидаемый ответ:

```json
{
  "status": "ready"
}
```

## HTTP Request node для получения mp4

Метод:

```text
GET
```

URL:

```text
http://host.docker.internal:3123/api/short-video/{{$json.videoId}}
```

Ожидаемые headers:

```http
Content-Type: video/mp4
Content-Disposition: inline; filename={videoId}.mp4
```

## Ограничения текущего MVP

- Видеоряд зависит от `searchTerms` и внешнего источника background videos.
- Русский текст озвучивается неидеально, так как доступные voices ориентированы на Kokoro TTS.
- Для production-уровня нужно отдельно подключать русский TTS.
- Визуальная релевантность пока ограничена поисковыми запросами.
- Для роликов с фирменным стилем, персонажами и контролируемыми сценами потребуется отдельный visual generation stack.

## Следующие улучшения

- Добавить node `Build Short Video Maker Payload` в n8n.
- Передавать `video_pipeline_json` в payload сервиса.
- Автоматически подбирать `searchTerms`.
- Автоматически выбирать `music` по тону ролика.
- Автоматически выбирать `voice`.
- Сохранять `videoId`.
- Проверять статус до `ready`.
- Сохранять прямой mp4 URL.
- Добавить отдельный слой русского TTS.
- Добавить Brand/Subtitles template.
