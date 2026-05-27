# Scene Package Builder

## Назначение

Scene Package Builder объединяет данные из `video_pipeline_json`, `storyboard` и `visual_bible` в отдельные пакеты для генерации сцен через AI Video Provider.

Главный принцип:

```text
1 scene = 1 scene package = 1 AI video generation task
```

## Вход

```text
video_pipeline_json
storyboard
visual_bible
character / object bible
```

## Выход

```json
{
  "id": "idea_001",
  "source": "scene-package-builder",
  "status": "scene_packages_ready",
  "provider": "higgsfield",
  "scene_packages": [
    {
      "scene_id": "scene_01",
      "duration_sec": 3,
      "aspect_ratio": "9:16",
      "output_format": "mp4",
      "voiceover_text": "Форму хоккейного клуба можно придумать не за недели, а за минуты.",
      "on_screen_text": "Форма клуба за минуты",
      "visual_prompt": "A cinematic close-up of a premium hockey jersey on a design table, modern sports-tech studio, AI interface glow in the background, vertical 9:16, premium commercial look",
      "negative_prompt": "basketball, football, soccer, random gym, generic fitness, low quality, distorted text, fake logos, wrong jersey shape",
      "camera_prompt": "fast push-in, shallow depth of field, product-focused close-up",
      "style_reference": "visual_bible_001",
      "character_reference": null,
      "object_reference": "hockey_jersey_01",
      "qa_rules": [
        "must show hockey jersey or hockey design context",
        "must not show basketball or soccer",
        "must match premium sports-tech style",
        "must be usable in vertical social video"
      ]
    }
  ]
}
```

## Правила сборки `visual_prompt`

Каждый `visual_prompt` должен содержать:

1. основной объект сцены;
2. действие или состояние;
3. окружение;
4. стиль;
5. формат;
6. качество;
7. важные визуальные ограничения.

Пример:

```text
A cinematic close-up of a premium hockey jersey on a design table, modern sports-tech studio, AI interface glow in the background, vertical 9:16, premium commercial look
```

## Правила сборки `negative_prompt`

`negative_prompt` должен отсеивать нерелевантные визуальные результаты.

Для текущего MVP по умолчанию добавлять:

```text
basketball, football, soccer, random gym, generic fitness, low quality, distorted text, fake logos, wrong jersey shape
```

## Правила сборки `camera_prompt`

`camera_prompt` должен описывать только движение камеры и тип кадра.

Примеры:

```text
fast push-in, shallow depth of field, product-focused close-up
slow cinematic pan, medium shot, premium studio lighting
static product shot, clean background, vertical 9:16
```

## Совместимость с провайдерами

Scene Package Builder не должен быть жёстко привязан к Higgsfield.

Поле `provider` может принимать:

```text
higgsfield
runway
luma
kling
pika
manual
```

## Следующий слой

Результат используется в:

```text
AI Video Provider Adapter
  ↓
Scene QA Agent
  ↓
Final Assembly Layer
```
