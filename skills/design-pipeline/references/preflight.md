# Preflight: проверка и установка зависимостей

Один раз перед первым запуском конвейера в проекте. Цель: всё на месте, а отсутствующее
предложено к установке, без молчаливого падения.

## Алгоритм

1. Определи харнес: Claude Code или Codex. От него зависят папки и синтаксис вызова.
2. Прочитай `dependencies.yaml`.
3. Найди каждую зависимость в папках скиллов своего харнеса:

   ```bash
   # Claude Code
   ls -d ~/.claude/skills/{impeccable,ux-discovery,emil-design-eng} .claude/skills/impeccable 2>/dev/null
   # Codex
   ls -d ~/.codex/skills/{ux-discovery,emil-design-eng} ~/.agents/skills/impeccable .agents/skills/impeccable 2>/dev/null
   ```

   Pocock-скиллы: в Claude Code это плагин `mattpocock-skills` (`claude plugin list`), в Codex
   скиллы `to-spec`, `to-tickets`, `implement` в `~/.codex/skills/`.
4. Поколение impeccable: найди в папке скилла упоминание команды `init`
   (`grep -rl "init" <папка impeccable>`). Нет `init`, а есть `teach`: это 2.x, конвейер на ней
   не поедет, предложи `npx impeccable update`. (`npx impeccable --version` показывает версию
   пакета в npx-кэше, а не установленного скилла.)
5. Для каждой отсутствующей зависимости покажи, чего нет, и команду из `install`. Спроси
   разрешение и ставь только после «да».
   - `required: true` (impeccable, ux-discovery): без них конвейер стоит, так и скажи.
   - `required: false` (emil, mattpocock-skills): предложи и продолжай без них.
6. После установки перезапусти харнес, если скилл не появился, и перепроверь шаг 3.

Готово, когда каждая required-зависимость найдена, а по каждой optional пользователь сказал
«ставим» или «без неё».

## Команды установки

```bash
# impeccable (required): один скилл, 24 команды, ставит в оба харнеса
npx impeccable@^4.8.0 install -y --providers=claude,codex --scope=global

# ux-discovery (required): из этого репо
npx skills add github.com/ahtlv/design-pipeline --skill ux-discovery -g -a claude-code codex

# emil-design-eng (optional)
npx skills add github.com/emilkowalski/skill --skill emil-design-eng -g -a claude-code codex
```

Только Codex: impeccable кладёт проектный хук в `.codex/hooks.json`. Открой `/hooks` и одобри
его; после обновлений, меняющих хук, Codex попросит одобрить снова.

## Почему так

- **Внешнее не копируется в репо**, ставится из источника: легально и обновляемо.
- **impeccable пинуется на `^4.8.0`**: миноры 4.x прилетают сами, мажор только руками после
  changelog. Обновление пина: одна строка в `dependencies.yaml`.
- **Один флаг на два харнеса** (`--providers=claude,codex`, `-a claude-code codex`): конвейер
  одинаково работает в Claude Code и Codex, и пользователь не ставит всё дважды.

## Граничные случаи

- Нет интернета: npx не пройдёт; предложи поставить заранее.
- Пользователь отказался от required: останови конвейер и назови, что именно без неё не поедет.
- impeccable стоит только в одном харнесе: запусти установку с `--providers=` только для
  недостающего (`claude` или `codex`).
