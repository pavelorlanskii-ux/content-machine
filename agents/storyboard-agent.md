# Storyboard Agent

## Назначение

Storyboard Agent преобразует `video_pipeline_json` в режиссёрскую раскадровку для будущей генерации сцен через AI Video Provider.

Он не генерирует финальное видео. Его задача: описать каждую сцену так, чтобы дальше можно было собрать стабильный `scene_package`.

## Вход

```json
{
  "id": "idea_001",
  "title": "...",
  "duration_seconds": 30,
  "language": "ru",
  "aspect_ratio": "9:16",
  "voiceover": "...",
  "scenes": []
}
```

## Выход

```json
{
  "id": "idea_001",
  "source": "storyboard-agent",
  "status": "storyboard_ready",
  "storyboard": [
    {
      "scene_id": "scene_01",
      "source_scene": 1,
      "goal": "hook",
      "duration_sec": 3,
      "shot_type": "close-up",
      "camera_motion": "fast push-in",
      "composition": "main object in foreground, background slightly blurred",
      "visual_description": "cinematic close-up of a premium hockey jersey on a design table",
      "required_objects": ["hockey jersey", "design table", "AI interface"],
      "forbidden_objects": ["basketball", "football shirt", "soccer field", "random gym"],
      "on_screen_text": "Форма клуба за минуты",
      "mood": "premium sports-tech"
    }
  ]
}
```

## Правила работы

1. Делить ролик на короткие сцены.
2. Одна сцена должна иметь одну понятную визуальную задачу.
3. Не смешивать слишком много действий в одном кадре.
4. Указывать shot type: close-up, medium shot, wide shot, product shot, over-the-shoulder.
5. Указывать camera motion: static, slow push-in, fast push-in, pan, tilt, handheld, tracking.
6. Указывать required objects.
7. Указывать forbidden objects для снижения нерелевантных генераций.
8. Держать вертикальный формат 9:16.
9. Избегать случайного спорта, если тема про хоккейную форму.
10. Не добавлять несуществующие бренды и логотипы.

## Специализация для текущего MVP

Текущая тема: генерация роликов про спортивный дизайн, хоккейную форму и AI-процесс.

По умолчанию избегать:

- basketball;
- football;
- soccer;
- random gym;
- generic fitness footage;
- unrelated sports;
- broken text;
- fake logos.

## Следующий слой

Результат Storyboard Agent используется в:

```text
Visual Bible Agent
  ↓
Scene Package Builder
```
