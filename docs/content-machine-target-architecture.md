# Content Machine Target Architecture

Документ фиксирует целевую архитектуру проекта Content Machine.

## Главная цель

Content Machine должна быть системой, которая не просто генерирует видео по готовой идее, а проходит полный цикл:

```text
найти трендовую тему
  ↓
оценить потенциал идеи
  ↓
собрать контент-план
  ↓
написать сценарий
  ↓
сделать раскадровку
  ↓
создать визуальную концепцию
  ↓
сгенерировать ключевые кадры
  ↓
сгенерировать видео
  ↓
собрать финальный ролик
  ↓
подготовить публикацию
  ↓
опубликовать
  ↓
собрать аналитику
  ↓
улучшить следующие ролики
```

## Принцип

Проект должен быть универсальным.

Темы могут быть разными:

- спорт;
- бизнес;
- образование;
- технологии;
- lifestyle;
- бренды;
- продукты;
- новости;
- тренды соцсетей.

Хоккейная форма и спортивный дизайн используются только как sample-кейс `idea_001`, а не как ограничение всей системы.

## Целевая цепочка агентов

```text
Trend Research Agent
  ↓
Idea Scoring Agent
  ↓
Content Plan Agent
  ↓
Script Agent
  ↓
Storyboard Agent
  ↓
Visual Bible Agent
  ↓
Keyframe Generation Layer
  ↓
AI Video Provider
  ↓
Scene QA Agent
  ↓
Final Assembly Layer
  ↓
Publishing Agent
  ↓
Analytics Agent
```

## 1. Trend Research Agent

Ищет потенциально сильные темы для коротких вертикальных видео.

Источники могут быть:

- TikTok trends;
- YouTube Shorts;
- Instagram Reels;
- X / Twitter;
- Google Trends;
- Reddit;
- Telegram;
- новостные источники;
- внутренний список ниш и тем;
- ручные идеи пользователя.

Выход:

```json
{
  "trend_id": "trend_001",
  "topic": "AI tools for sports design",
  "source": "manual",
  "trend_signal": "high",
  "notes": "topic has visual demonstration potential"
}
```

## 2. Idea Scoring Agent

Оценивает найденные темы.

Критерии:

- новизна;
- вирусный потенциал;
- визуальный потенциал;
- понятность за 30 секунд;
- применимость к выбранной нише;
- сложность производства;
- ожидаемая вовлечённость.

Выход:

```json
{
  "idea_id": "idea_001",
  "topic": "AI tools for sports design",
  "score": 82,
  "priority": "high",
  "reason": "clear visual contrast and strong hook potential"
}
```

## 3. Content Plan Agent

Собирает идеи в контент-план.

Выход:

```json
{
  "week": "2026-W22",
  "items": [
    {
      "idea_id": "idea_001",
      "platform": "Reels",
      "status": "ready_for_script"
    }
  ]
}
```

## 4. Script Agent

Пишет сценарий ролика.

Отвечает за:

- hook;
- структуру;
- voiceover;
- экранный текст;
- CTA;
- длительность;
- платформу.

## 5. Storyboard Agent

Делает раскадровку.

Важно: Storyboard Agent должен быть универсальным. Он не должен быть привязан к спорту или хоккею.

Он описывает:

- сцену;
- кадр;
- движение камеры;
- объекты;
- настроение;
- композицию;
- визуальные ограничения.

## 6. Visual Bible Agent

Фиксирует визуальный стиль конкретного ролика или серии.

Visual Bible зависит от темы.

Примеры:

- для спортивного ролика: premium sports-tech style;
- для образовательного ролика: clean explainer style;
- для lifestyle ролика: cinematic creator style;
- для продукта: commercial product demo style;
- для новости: fast editorial social-video style.

## 7. Keyframe Generation Layer

Создаёт ключевые кадры сцен.

Целевая логика:

```text
scene_package
  ↓
keyframe_prompt
  ↓
image generator
  ↓
keyframe image
  ↓
public image_url
```

Первый кандидат для image generator:

```text
Higgsfield Soul
```

## 8. AI Video Provider

Генерирует видео на основе keyframe image и prompt.

Первый кандидат:

```text
Higgsfield DoP Turbo
```

Архитектура должна позволять заменить провайдера:

- Higgsfield;
- Runway;
- Luma;
- Kling;
- Pika;
- другой API.

## 9. Scene QA Agent

Проверяет каждую сцену.

Критерии:

- соответствует ли сцена storyboard;
- соответствует ли visual bible;
- нет ли артефактов;
- нет ли нерелевантных объектов;
- подходит ли сцена для финального ролика;
- нет ли проблем с текстом, лицами, логотипами или визуальной логикой.

## 10. Final Assembly Layer

Собирает финальное видео:

- сцены;
- voiceover;
- captions;
- music;
- transitions;
- final mp4.

На раннем этапе эту роль может частично выполнять Short Video Maker.

В дальнейшем сборку лучше выделить в отдельный ffmpeg / Remotion layer.

## 11. Publishing Agent

Готовит публикацию:

- title;
- caption;
- hashtags;
- platform-specific text;
- posting time;
- publishing payload.

Потенциальный сервис:

```text
Postiz
```

## 12. Analytics Agent

Собирает результаты публикаций:

- views;
- retention;
- likes;
- comments;
- shares;
- saves;
- CTR;
- follower growth.

Задача Analytics Agent:

```text
результаты публикаций
  ↓
выводы
  ↓
улучшение следующих идей
```

## MVP stages

### MVP 0.1

Уже собрано:

- идея;
- script;
- video payload;
- Short Video Maker;
- mp4.

### MVP 0.2

Добавляется:

- Storyboard Agent;
- Visual Bible Agent;
- Scene Package Builder;
- Higgsfield как AI Video Provider;
- Keyframe Generation Layer.

### MVP 0.3

Добавить верхний слой:

- Trend Research Agent;
- Idea Scoring Agent;
- Content Plan Agent.

### MVP 0.4

Добавить нижний слой:

- Publishing Agent;
- Analytics Agent;
- feedback loop.

## Важное ограничение

Все текущие файлы с `idea_001` являются тестовым sample-кейсом.

Они не должны ограничивать архитектуру проекта только спортом, хоккеем или дизайном формы.

## Практический вывод

Higgsfield Soul и Higgsfield DoP Turbo являются частью visual/video layer, а не центром всей системы.

Центр системы:

```text
trend → idea → script → video → publish → analytics → improve
```
