# design-pipeline v2 — оркестратор поверх существующих скиллов

**Дата:** 2026-06-04
**Статус:** направление подтверждено Анатолём — **оркестратор поверх impeccable**.
**Суть:** не переизобретать фазы 1–3, а оркестрировать `impeccable` + `emil-design-eng` + `ux-discovery`.

---

## Ключевой инсайт

Установленный скилл `impeccable` (24 команды, Apache 2.0, на базе Anthropic `frontend-design`)
уже закрывает фазы 1–3 исходной спеки — местами сильнее, чем спроектировано:

- **DESIGN.md как источник правды** (= главный тезис постов из Избранного) — ядро impeccable.
- **Двухуровневый AI slop test** (first/second-order category-reflex) — встроен, сильнее самодельного чеклиста.
- **extract-из-референса** — команда `impeccable extract` (то, что советовалось из постов).
- **Браузерная докрутка** — команда `impeccable live`.

Поэтому `design-pipeline` сжимается до тонкого оркестратора + слой зависимостей + интеграция в superpowers.

## Маппинг команд impeccable на фазы исходной спеки

| Фаза спеки | Команды impeccable | Кто добавляет |
|---|---|---|
| Ф0 UX | — (impeccable `teach` собирает PRODUCT.md поверхностно) | **`ux-discovery`** (глубокое интервью → PRODUCT.md) |
| Ф1 Design System | `teach` → `document` / `extract` | impeccable |
| Ф2 Build | `shape` → `craft` | impeccable (+ `emil` на motion) |
| Ф3 Visual Review | `critique` + `audit` + `polish` + `live` (+ встроенный slop test) | impeccable |

## Пересмотренные фазы (что реально вызывается)

### Ф0 — UX discovery
- **Инструмент:** `ux-discovery` (свой скилл).
- **Делает:** интервью по PRD → user journey, боли, happy path.
- **Артефакт:** `PRODUCT.md` в формате, который ест impeccable (Users, brand, tone, anti-references, strategic principles, `register`).
- **Связка:** ux-discovery — единственный источник, которого impeccable не имеет. Он кормит весь конвейер.

### Ф1 — Design System
- **Инструмент:** `impeccable`: `teach` (если PRODUCT.md ещё нет) → `document` (DESIGN.md из кода) или `extract <референс>` (токены из примера).
- **Артефакт:** `DESIGN.md` + токены (источник правды для всех экранов).

### Ф2 — Build
- **Инструмент:** `impeccable craft <feature>` (shape → build end-to-end на базе frontend-design).
- **Motion-углубление:** `impeccable animate` запускается с подключённым `emil-design-eng` —
  emil даёт decision-framework, кастомные easing-кривые, springs, perf-правила (transform/opacity), `prefers-reduced-motion`.
- **Артефакт:** код UI, соблюдающий DESIGN.md + emil-motion.

### Ф3 — Visual Review (цикл докрутки)
- **Инструмент:** `impeccable critique` (UX-скоринг) → `audit` (a11y/perf/responsive) → `polish` (финальный проход);
  встроенный AI slop test ловит generic. Браузерная итерация — `impeccable live`.
- **Опционально:** `superpowers-chrome` — только если нужны «честные» скриншоты сверх `live`. По умолчанию НЕ тянем (YAGNI): `live` уже даёт браузерную докрутку.
- **Артефакт:** отчёт + правки, лимит итераций фиксирован.

## Что переиспользуем vs что пишем сами

| Переиспользуем (НЕ писать) | Пишем сами (тонкий слой) |
|---|---|
| impeccable: design system, build, slop test, extract, live, audit | Оркестратор: последовательность ux-discovery → impeccable → emil |
| emil-design-eng: весь motion | dependency-манифест + preflight-проверка |
| ux-discovery: интервью | правило интеграции в superpowers (CLAUDE.md / триггер) |
| frontend-design: база (внутри impeccable — отдельно может не нужен) | маппинг ux-discovery → формат PRODUCT.md impeccable |

## Слой зависимостей (требование Анатоля: «скилл понимает зависимости и предлагает установку»)

Манифест `dependencies.yaml` в корне скилла:

```yaml
# design-pipeline зависимости
skills:
  - name: ux-discovery
    source: local           # claude-config/skills/ux-discovery
    required: true
    phase: 0
  - name: impeccable
    source: "npx impeccable@^2.3.2" # Apache 2.0; npx-управляемый мульти-harness шаблон
    version_strategy: "caret ^2.3.2 — авто патчи/миноры внутри 2.x, мажор (3.0) только вручную после changelog"
    required: true
    phases: [1, 2, 3]
    note: "НЕ копировать папку (плейсхолдеры {{scripts_path}} + резолв путей в рантайме). Вызывать через npx. Обновление = правка этой строки в git. Миграция на полную установку (pnpm add -g) открыта в любой момент, без конфликтов (npx-кэш и global node_modules — разные папки)."
  - name: emil-design-eng
    source: "npx skills add github.com/emilkowalski/skill --skill emil-design-eng"  # официальный скилл Эмиля
    required: false          # усилитель motion
    phase: 2
    note: "НЕ вендорить (нет LICENSE). Ставить из источника автора. Локальная копия в claude-config — убрать."
plugins:
  - name: frontend-design
    enable: "frontend-design@claude-plugins-official"
    required: false          # уже внутри impeccable; включать только при прямом использовании
  - name: superpowers-chrome
    enable: "superpowers-chrome@superpowers-marketplace"
    required: false          # только если нужны скриншоты сверх impeccable live
```

### Preflight (псевдо-логика старта скилла)

1. Прочитать `dependencies.yaml`.
2. Для каждого `required: true` — проверить наличие:
   - skill `local` → есть ли папка в `~/.claude/skills/` или `claude-config/skills/`;
   - skill `npx ...` → доступна ли команда;
   - plugin → включён ли в `settings.json → enabledPlugins`.
3. Чего нет — показать список + команды установки, спросить разрешение. Не падать на `required: false`.

## Интеграция в superpowers

- **Триггер 1 (явный):** «сделай дизайн» / «design» / «прокачай UI» → запуск оркестратора.
- **Триггер 2 (правило в CLAUDE.md):** когда superpowers-агенты доходят до генерации UI — обязаны
  вызвать `design-pipeline`, а не лепить вслепую. design-pipeline сам разворачивает ux-discovery → impeccable → emil.
- design-pipeline не заменяет superpowers, а подключается на UI-этапе его флоу.

## Distribution (унифицированная npx-модель)

Все внешние зависимости ставятся официальными `npx`-установщиками из источника — НЕ вендорятся:

| Зависимость | Тип | Команда установки (preflight ставит, если нет) |
|---|---|---|
| `impeccable` | публичный npm-пакет (CLI `cli/bin/cli.js`) | `npx impeccable@^2.3.2` |
| `emil-design-eng` | официальный скилл Эмиля Ковальски (`github.com/emilkowalski/skill`) | `npx skills add github.com/emilkowalski/skill --skill emil-design-eng` |
| `ux-discovery` | свой скилл | едет в репо `design-pipeline` (vendored, свой контент) |
| `design-pipeline` | свой скилл — ЭТО раздаваемое | `npx skills add github.com/ahtlv/design-pipeline` |

**Раздача студентам:**
1. Студент ставит `superpowers` (плагин) — база процессов.
2. `npx skills add github.com/ahtlv/design-pipeline` — твоя UI-надстройка (одна команда).
3. design-pipeline при первом запуске (preflight) сам доустанавливает impeccable + emil их npx-командами.

**preflight по типу зависимости:**
- `impeccable` → доступна ли `npx impeccable`? нет → предложить `npx impeccable@^2.3.2`.
- `emil-design-eng` → есть ли скилл? нет → предложить `npx skills add github.com/emilkowalski/skill --skill emil-design-eng`.

⚠️ emil НЕ вендорить копией: у репо нет явного LICENSE — копирование чужого контента в публичный
учебный репо = серая зона. Установка из источника автора санкционирована (он сам выложил для CC/Codex).
Локальная копия в `claude-config/skills/emil-design-eng` (сделана ранее) — избыточна, заменяется preflight-установкой.

## Интеграция в superpowers (НЕ «поверх» — плоская модель)

Иерархии «модуль поверх superpowers» в Claude Code нет. Скиллы плоские, Claude выбирает их по `description`
(model-invoked). superpowers сам design-pipeline не зовёт — его зовёт Claude по релевантности описания,
в т.ч. внутри superpowers-флоу (на execute-этапе, где подтягиваются implementation-скиллы).

Связка достигается тремя вещами:
1. design-pipeline установлен как скилл.
2. Сильное триггер-`description` («генерируешь лендинг/дашборд/UI...») — Claude сам подхватывает.
3. Правило в `CLAUDE.md`: «при генерации UI обязательно вызвать design-pipeline» — страховка на execute-этапе.

Активация на UI-задаче — авто (по описанию). Гарантию даёт правило в CLAUDE.md. Для студентов это
прописывается явно в README (одна настройка). design-pipeline и superpowers остаются независимыми
(pipeline работает и без superpowers — по явному «сделай дизайн»).

## Черновик SKILL.md (каркас для writing-plans)

```markdown
---
name: design-pipeline
description: "Дизайн-конвейер для superpowers: ux-discovery → impeccable → emil. Использовать когда: сделай дизайн, прокачай UI, design, сверстай лендинг/дашборд, нужен не-generic интерфейс. Оркестрирует существующие скиллы, не дублирует их."
---

# Design Pipeline

Тонкий оркестратор. Сам дизайн не реализует — связывает ux-discovery (UX) →
impeccable (система + сборка + ревью) → emil-design-eng (motion), встраивается
в superpowers на UI-этапе.

## Preflight (зависимости)
1. Прочитать dependencies.yaml.
2. Проверить required (ux-discovery, impeccable). Чего нет — показать команды
   установки, спросить разрешение. На required: false не падать.

## Фазы

### Ф0 — UX
Вызвать ux-discovery → получить/обновить PRODUCT.md в формате impeccable
(Users, brand, tone, anti-references, strategic principles, register).

### Ф1 — Design System
impeccable: `teach` (если PRODUCT.md нет) → `document`/`extract` → DESIGN.md + токены.

### Ф2 — Build
`impeccable craft <feature>`. На motion-шаге подключить emil-design-eng —
ВАЖНО: звать emil с конкретным запросом, не вхолостую (у него gate на пустой вызов).

### Ф3 — Visual Review (цикл, лимит N итераций)
`impeccable critique` → `audit` → `polish` (slop test встроен) → `live` для
браузерной докрутки. superpowers-chrome — только если нужны скриншоты сверх live.

## Граничные случаи
- Только UI/UX, не backend.
- Мелкий фикс — можно вызвать одну фазу/команду, не весь конвейер.
- Лимит итераций Ф3 фиксирован, чтобы не жечь токены.
```

## Открытые вопросы

1. **Глобализация зависимостей** — ✅ решено.
   - `emil-design-eng` вынесен в `claude-config/skills/` (глобально через симлинк `~/.claude/skills`).
   - `impeccable` — НЕ копией, а on-demand через `npx impeccable@^2.3.2` (caret: фиксы 2.x сами,
     мажор вручную). preflight проверяет доступность и предлагает установку, если нет.
   - Миграция на полную установку (`pnpm add -g impeccable`) остаётся открытой и безопасной —
     версии в разных папках (npx-кэш vs global node_modules), не конфликтуют.
2. **superpowers-chrome vs impeccable live** — решить, нужен ли отдельный скриншот-инструмент,
   или `live` достаточно. По умолчанию — `live`.
3. **frontend-design отдельно** — вероятно не нужен (внутри impeccable). Подтвердить.
4. **GitHub-distribution** — design-pipeline как папка в `ahtlv/claude-config` или отдельное репо.
