# Changelog

Все заметные изменения design-pipeline.
Формат: [Keep a Changelog](https://keepachangelog.com), версии: [semver](https://semver.org).

## [0.2.0] — unreleased
### Added
- Поддержка **Codex**: `agents/openai.yaml` для обоих скиллов, таблица синтаксиса вызова
  (`/impeccable` и `$impeccable`), установка одной командой в оба харнеса, проверка папок
  `~/.codex/skills` и `~/.agents/skills` в preflight.
- Встраивание в конвейер **Matt Pocock skills**: фазы привязаны к стадиям grill / to-spec /
  to-tickets / implement / code-review. Тикет «Дизайн-система» блокирует UI-тикеты.
- Ф1 через Pocock `prototype` (UI-ветка): несколько вариантов, референс как вводная в `shape`.
- Ф4 «Дизайн-ревью» как третья ось к `code-review`, первым шагом `npx impeccable detect`.
- Критерии готовности на каждой фазе.

### Changed
- impeccable `^2.3.2` → `^4.8.0`: `teach` заменён на `init`, установка через `npx impeccable install`.
- Фаз стало пять (0–4): «Спека и тикеты» выделена в отдельную фазу.
- Спеки перенесены из `docs/superpowers/specs/` в `docs/specs/`.

### Removed
- Зависимость и интеграция с **superpowers**.
- EXTRACT-из-референса по URL: в impeccable 4.x `extract` собирает компоненты из своего кода.

## [0.1.0] — unreleased
### Added
- Первый релиз. Оркестратор `ux-discovery → impeccable → emil`.
- 4 фазы: UX → Design System → Build → Visual Review (цикл докрутки, лимит 3 итерации).
- `preflight`: проверка и установка зависимостей (impeccable `^2.3.2`, emil-design-eng, ux-discovery).
- Vendored `ux-discovery` (свой скилл, едет в комплекте).
- README с порядком установки и правилом интеграции в superpowers.
- Унифицированная npx-модель зависимостей (ничего не вендорится, кроме своего ux-discovery).
