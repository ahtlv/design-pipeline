---
name: design-pipeline
version: 0.1.0
description: "Дизайн-конвейер: превращает PRD/идею в целостный, не-generic UI через оркестрацию ux-discovery → impeccable → emil. Использовать когда пользователь говорит: сделай дизайн, прокачай UI, сверстай лендинг/дашборд, нужен красивый интерфейс без AI-slop, design a frontend, build a landing page, дизайн посадочной, сделай анимацию, анимация элемента. Надстройка над superpowers для UI-этапа."
---

# Design Pipeline

Тонкий оркестратор дизайн-процесса. Сам дизайн не реализует — связывает три уже существующих
инструмента в один конвейер и встраивается в superpowers на UI-этапе:

- **ux-discovery** — UX-исследование → `PRODUCT.md` (контекст для impeccable)
- **impeccable** — дизайн-система, сборка, анти-slop ревью (23 команды)
- **emil-design-eng** — углубление motion (анимации, easing, springs)

Конвейер бьёт по четырём болям: слабый UX, generic-эстетика, разъезжающийся стиль, сдача вслепую без докрутки.

## Preflight — проверка зависимостей

Перед запуском конвейера убедись, что зависимости установлены. Детали и команды —
в [references/preflight.md](references/preflight.md). Кратко:

| Зависимость | Проверка | Установка, если нет |
|---|---|---|
| `impeccable` | доступна команда `/impeccable`? | `npx impeccable@^2.3.2 skills install` |
| `ux-discovery` | скилл установлен? | `npx skills add github.com/ahtlv/design-pipeline --skill ux-discovery` |
| `emil-design-eng` | скилл установлен? (опционально, для motion) | `npx skills add github.com/emilkowalski/skill --skill emil-design-eng` |

Чего нет — покажи команду установки и спроси разрешение. На опциональный `emil` не падай —
без него Ф2 работает, просто motion будет на уровне impeccable, а не углублённый.

## Фазы

Для мелкой задачи можно вызвать одну фазу (например только Ф3 — ревью), не гоняя весь конвейер.

### Ф0 — UX
1. Вызови скилл **ux-discovery** с PRD/идеей пользователя.
2. Оформи результат как **`PRODUCT.md`** в формате impeccable: Users, brand, tone, anti-references,
   strategic principles, поле `register` (brand или product).
3. Если PRD нет — ux-discovery ведёт интервью и собирает картину сам.

→ Артефакт: `PRODUCT.md`. Лечит: UX/структуру.

### Ф1 — Design System
Источник дизайн-системы выбирается по приоритету:
1. **EXTRACT** (приоритет) — есть референс, который нравится: `/impeccable extract <url|screenshot>` →
   вытащить токены из живого примера. Лучший анти-slop.
2. **SEED** — референса нет, но есть вектор: взять готовую систему и адаптировать.
3. **FROM-SCRATCH** (fallback) — `/impeccable teach` (если нет PRODUCT.md) → `/impeccable document`.

→ Артефакт: `DESIGN.md` + токены = источник правды для всех экранов. Лечит: единый стиль.

### Ф2 — Build
1. `/impeccable craft <feature>` — shape → build end-to-end, опираясь на `DESIGN.md`.
2. **Motion-углубление:** на этапе анимаций вызови скилл **emil-design-eng** с КОНКРЕТНЫМ запросом
   (не вхолостую — у него gate на пустой вызов). Например: «применить motion-принципы Эмиля к этим
   компонентам: easing-кривые, тайминги <300ms, spring для drag, prefers-reduced-motion».

→ Артефакт: код UI, соблюдающий `DESIGN.md` + motion Эмиля. Лечит: визуал/эстетику.

### Ф3 — Visual Review (цикл докрутки)
Лимит итераций — **3** (чтобы не жечь токены бесконечно).
1. `/impeccable critique` — UX-ревью с эвристическим скорингом.
2. `/impeccable audit` — технические проверки (a11y, perf, responsive).
3. `/impeccable polish` — финальный проход; встроенный AI slop test ловит generic.
4. `/impeccable live` — браузерная докрутка элементов (нужен запущенный dev-server).
5. Применить правки → повторить, пока не сойдётся или не упрётся в лимит.

→ Артефакт: отчёт + применённые правки. Лечит: докрутку (агент впервые СМОТРИТ на результат).

## Граничные случаи
- **Только UI/UX** — backend/логику конвейер не трогает.
- **Мелкий фикс** — вызови одну фазу/команду, не весь конвейер.
- **Нет dev-server** — Ф3 `live` требует запущенный сервер; для статики impeccable работает с файлом.
- **emil не установлен** — Ф2 не падает, motion остаётся на уровне impeccable.

## Интеграция в superpowers

design-pipeline — обычный скилл, Claude вызывает его по описанию (model-invoked) на UI-задаче,
в том числе внутри superpowers-флоу (на execute-этапе, где подтягиваются implementation-скиллы).

Для надёжной связки добавь правило в свой `CLAUDE.md` (см. [README](../../README.md)):
> При генерации фронтенда/UI обязательно использовать design-pipeline, а не верстать вслепую.

design-pipeline и superpowers независимы: конвейер работает и без superpowers (по явному «сделай дизайн»).
