# Changelog

Все заметные изменения design-pipeline.
Формат — [Keep a Changelog](https://keepachangelog.com), версии — [semver](https://semver.org).

## [0.1.0] — unreleased
### Added
- Первый релиз. Оркестратор `ux-discovery → impeccable → emil`.
- 4 фазы: UX → Design System → Build → Visual Review (цикл докрутки, лимит 3 итерации).
- `preflight`: проверка и установка зависимостей (impeccable `^2.3.2`, emil-design-eng, ux-discovery).
- Vendored `ux-discovery` (свой скилл, едет в комплекте).
- README с порядком установки и правилом интеграции в superpowers.
- Унифицированная npx-модель зависимостей (ничего не вендорится, кроме своего ux-discovery).
