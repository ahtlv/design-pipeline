# design-pipeline

Дизайн-конвейер для **Claude Code** и **Codex**. Превращает идею или PRD в целостный UI без
generic-эстетики: оркеструет три инструмента и встраивает их в конвейер
[Matt Pocock skills](https://github.com/mattpocock/skills) на UI-этапах.

```
grill-me / wayfinder ─┬─ Ф0 ux-discovery → impeccable init      → PRODUCT.md
                      └─ Ф1 prototype (N вариантов) → document   → DESIGN.md
to-spec, to-tickets  ─── Ф2 тикет «Дизайн-система» блокирует UI-тикеты
implement            ─── Ф3 craft по DESIGN.md + emil (motion), логика через tdd
code-review          ─── Ф4 ось Design: detect → critique → audit → polish → live
```

## Что лечит

Четыре боли, когда агент генерит UI вслепую:

1. **Слабый UX**: навигация и флоу не продуманы. Лечит Ф0.
2. **Generic AI-slop**: дёшево и безлико. Лечат сравнение вариантов в Ф1 и детектор impeccable в Ф4.
3. **Разъезжающийся стиль**: каждый экран свой. Лечат `DESIGN.md` и тикет дизайн-системы,
   который блокирует все UI-тикеты.
4. **Сдача без докрутки**: агент не смотрит на результат. Лечит цикл ревью в Ф4.

## Установка

### 1. Конвейер Покока (рекомендуется)

Фазы привязаны к его стадиям. Без него design-pipeline тоже работает, но уже как отдельная труба.

```bash
# Claude Code
claude plugin install mattpocock-skills@claude-plugins-official
# Codex
codex plugin marketplace add mattpocock/skills
codex plugin add mattpocock-skills@mattpocock
```

Потом в проекте один раз запусти `setup-matt-pocock-skills`: он настроит трекер задач.

### 2. design-pipeline + ux-discovery

Одна команда ставит в оба харнеса:

```bash
npx skills add github.com/ahtlv/design-pipeline --skill design-pipeline ux-discovery -g -a claude-code codex
```

### 3. Правило интеграции

Pocock-скиллы `implement` и `code-review` зовут другие скиллы явно, по имени. Сами по себе
они design-pipeline не позовут, поэтому нужно правило. Добавь его в `~/.claude/CLAUDE.md`
и в `~/.codex/AGENTS.md` (или в оба файла проекта):

```markdown
## UI
Тикет или задача, которая создаёт или меняет интерфейс, идёт через скилл design-pipeline:
он задаёт, что делать на стадиях grill, to-spec, to-tickets, implement и code-review.
```

### 4. impeccable + emil

Доустановятся **автоматически** при первом запуске (preflight спросит разрешение). Или вручную:

```bash
npx impeccable@^4.8.0 install -y --providers=claude,codex --scope=global
npx skills add github.com/emilkowalski/skill --skill emil-design-eng -g -a claude-code codex
```

В Codex после установки impeccable открой `/hooks` и одобри его проектный хук.

## Вызов: Claude Code и Codex

| | Claude Code | Codex |
|---|---|---|
| команда impeccable | `/impeccable craft hero` | `$impeccable craft hero` |
| скилл | `/design-pipeline` | `$design-pipeline` |
| правило | `CLAUDE.md` | `AGENTS.md` |

## Использование

- **Явно:** «сделай дизайн», «сверстай лендинг», «прокачай UI», «анимация элемента».
- **В конвейере Покока:** на UI-тикете в `implement` и после `code-review` (через правило из шага 3).
- **Точечно:** одна фаза, например только Ф4 на уже существующем UI.

## Фазы

| Фаза | Стадия Покока | Инструмент | Артефакт |
|---|---|---|---|
| 0: UX | grill-me / wayfinder | ux-discovery → `impeccable init` | `docs/discovery.md`, `PRODUCT.md` |
| 1: Направление | до to-spec | prototype (UI) + `impeccable shape` → `document` | `DESIGN.md` + токены |
| 2: Спека и тикеты | to-spec, to-tickets | тикет «Дизайн-система» первым | спека, граф тикетов |
| 3: Сборка | implement | `impeccable craft` + emil, tdd для логики | код UI |
| 4: Дизайн-ревью | после code-review | `detect` → `critique` → `audit` → `polish` → `live` | правки, лимит 3 итерации |

## Зависимости

| Зависимость | Роль | Источник |
|---|---|---|
| `mattpocock-skills` | конвейер процессов (optional) | [mattpocock/skills](https://github.com/mattpocock/skills) |
| `impeccable` `^4.8.0` | PRODUCT/DESIGN.md, сборка, ревью, детектор | [impeccable.style](https://impeccable.style) (pbakaus, Apache 2.0) |
| `emil-design-eng` | motion (optional) | [emilkowalski/skill](https://github.com/emilkowalski/skill) |
| `ux-discovery` | UX-интервью | в этом репо |

## Версии и обновление

Semver через git-теги. История в [CHANGELOG](CHANGELOG.md), текущая версия в поле `version`
файла `skills/design-pipeline/SKILL.md`.

```bash
npx skills update design-pipeline        # до последней
npx impeccable update                    # impeccable отдельно
```

Пин impeccable живёт в `dependencies.yaml`: версия пайплайна и версии зависимостей
развиваются независимо.

## Атрибуция

`impeccable`, `emil-design-eng` и `mattpocock-skills` сторонние и **не включены в репозиторий**:
они ставятся из источников авторов. Спасибо [pbakaus](https://impeccable.style),
[Emil Kowalski](https://emilkowal.ski/skill) и [Matt Pocock](https://github.com/mattpocock/skills).

## Лицензия

MIT, см. [LICENSE](LICENSE). Распространяется на код этого репозитория (оркестратор + ux-discovery).
Сторонние зависимости под лицензиями их авторов.
