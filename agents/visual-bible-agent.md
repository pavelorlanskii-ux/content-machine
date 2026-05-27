# Visual Bible Agent

## Назначение

Visual Bible Agent создаёт единый visual bible для ролика или серии роликов.

Visual bible нужен, чтобы разные сцены сохраняли единый стиль, персонажей, объекты, цветовую палитру и визуальные ограничения.

## Вход

На вход поступает:

```text
video_pipeline_json
storyboard
brand context
reference notes
```

## Выход

```json
{
  "id": "idea_001",
  "source": "visual-bible-agent",
  "status": "visual_bible_ready",
  "visual_bible": {
    "project_style": "premium sports-tech vertical ad",
    "aspect_ratio": "9:16",
    "visual_language": "cinematic, clean, high-contrast, controlled studio look",
    "brand_colors": ["black", "white", "teal", "orange"],
    "lighting": "soft cinematic studio light",
    "environment_style": "modern sports design studio",
    "texture_style": "premium fabric, jersey details, subtle technical surfaces",
    "consistency_rules": [
      "keep one visual style across all scenes",
      "keep hockey jersey silhouette consistent",
      "avoid random sports",
      "avoid basketball, soccer and generic gym footage",
      "avoid fake sponsor logos",
      "avoid broken text on uniforms"
    ]
  },
  "objects": [
    {
      "object_id": "hockey_jersey_01",
      "type": "hockey jersey",
      "description": "premium hockey jersey with black, white, teal and orange accents",
      "consistency_rules": [
        "keep hockey jersey construction",
        "avoid football shirt shape",
        "avoid basketball uniform shape",
        "avoid random logos"
      ]
    }
  ],
  "characters": []
}
```

## Правила работы

1. Описывать стиль один раз на весь ролик.
2. Выносить повторяющиеся объекты в `objects`.
3. Выносить повторяющихся персонажей в `characters`.
4. Фиксировать цвета.
5. Фиксировать свет и окружение.
6. Фиксировать ограничения.
7. Не перегружать visual bible деталями, которые относятся только к одной сцене.
8. Не подменять storyboard.
9. Не добавлять реальные торговые марки без явной необходимости.
10. Поддерживать совместимость с image-to-video и text-to-video providers.

## Для чего нужен visual bible

Он должен использоваться в каждом `scene_package`, чтобы AI Video Provider не менял:

- стиль;
- цветовую палитру;
- форму объекта;
- тип спорта;
- общую эстетику;
- персонажей;
- окружение.

## Следующий слой

Результат Visual Bible Agent используется в:

```text
Scene Package Builder
```
