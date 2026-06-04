# design-pipeline

Дизайн-конвейер для Claude Code. Превращает PRD/идею в целостный, **не-generic** UI — оркеструя
три инструмента в один поток. Надстройка над [superpowers](https://github.com/obra/superpowers) для UI-этапа.

```
PRD ─▶ ux-discovery ─▶ impeccable (design system → build → review) ─▶ готовый UI
                                          ▲ emil-design-eng (motion)
```

## Что лечит

Четыре боли, которые болят, когда агент генерит UI вслепую:

1. **Слабый UX** — навигация и флоу не продуманы → решает фаза UX.
2. **Generic AI-slop** — дёшево, безлико → решает встроенный anti-slop test impeccable + extract из референса.
3. **Разъезжающийся стиль** — каждый экран свой → решает единая дизайн-система (источник правды).
4. **Сдача без докрутки** — агент не смотрит на результат → решает цикл visual review в браузере.

## Установка

Порядок важен.

### 1. superpowers (база процессов)
Установи плагин `superpowers` через меню `/plugin` в Claude Code.

### 2. design-pipeline + ux-discovery
```bash
npx skills add github.com/ahtlv/design-pipeline --skill design-pipeline
npx skills add github.com/ahtlv/design-pipeline --skill ux-discovery
```

### 3. Правило интеграции в superpowers
Добавь в свой `~/.claude/CLAUDE.md`:

```markdown
## Дизайн UI
При генерации фронтенда/UI обязательно использовать скилл design-pipeline,
а не верстать вслепую. Он сам разворачивает ux-discovery → impeccable → emil.
```

Без этого правила design-pipeline всё равно подхватится по описанию на UI-задаче,
но правило делает связку с superpowers надёжной.

### 4. impeccable + emil
Доустановятся **автоматически** при первом запуске (preflight спросит разрешение). Или вручную заранее:
```bash
npx impeccable@^2.3.2 skills install
npx skills add github.com/emilkowalski/skill --skill emil-design-eng
```

## Версии и обновление

Версионирование — semver через git-теги (`v0.1.0`, `v0.2.0`, ...). История — в [CHANGELOG](CHANGELOG.md),
текущая версия — в поле `version` файла `skills/design-pipeline/SKILL.md`.

```bash
# последняя версия (по умолчанию ставит из main)
npx skills add github.com/ahtlv/design-pipeline --skill design-pipeline

# пин на конкретную версию (через git-тег в пути URL)
npx skills add https://github.com/ahtlv/design-pipeline/tree/v0.1.0/skills/design-pipeline

# обновить установленный скилл до последней
npx skills update design-pipeline
```

Версии зависимостей пинуются отдельно: impeccable — `^2.3.2` в `dependencies.yaml` (фиксы 2.x сами,
мажор вручную). Так версия пайплайна и версии его зависимостей развиваются независимо.

## Использование

- **Явно:** «сделай дизайн», «прокачай UI», «сверстай лендинг», «design a frontend», «дизайн посадочной», «сделай анимацию», «анимация элемента»
- **Авто:** внутри superpowers-флоу на этапе реализации UI (если добавил правило из шага 3).
- **Точечно:** можно вызвать одну фазу, например только ревью существующего UI.

## Фазы

| Фаза | Инструмент | Артефакт |
|---|---|---|
| 0 — UX | ux-discovery | `PRODUCT.md` |
| 1 — Design System | impeccable `extract`/`teach`/`document` | `DESIGN.md` + токены |
| 2 — Build | impeccable `craft` + emil (motion) | код UI |
| 3 — Visual Review | impeccable `critique`/`audit`/`polish`/`live` | докрученный UI |

## Зависимости

| Зависимость | Роль | Источник |
|---|---|---|
| `superpowers` | оркестратор процессов | плагин (`/plugin`) |
| `impeccable` | дизайн-система, сборка, anti-slop | [impeccable.style](https://impeccable.style) (pbakaus, Apache 2.0) |
| `emil-design-eng` | motion / анимации | [github.com/emilkowalski/skill](https://github.com/emilkowalski/skill) (Emil Kowalski) |
| `ux-discovery` | UX-исследование | в этом репо |

## Атрибуция

`impeccable` и `emil-design-eng` — сторонние скиллы, они **не включены в этот репозиторий**, а ставятся
из источников их авторов (это законно и даёт автообновления). Спасибо [pbakaus](https://impeccable.style)
и [Emil Kowalski](https://emilkowal.ski/skill) за их работу.

## Лицензия

MIT — см. [LICENSE](LICENSE). Распространяется на код этого репозитория (оркестратор + ux-discovery).
Сторонние зависимости — под лицензиями их авторов.
