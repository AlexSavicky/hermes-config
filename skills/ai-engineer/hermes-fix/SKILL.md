---
name: hermes-fix
description: Fix DevOps Hermes — edit SOUL.md, config.yaml, AGENTS.md, update skills
version: 1.0
---

# Fix DevOps Hermes

Исправление конфигов DevOps Hermes (default профиль).

## Основные файлы для правки

| Файл | Что править | Инструмент |
|---|---|---|
| `/root/.hermes/SOUL.md` | Противоречивые инструкции, неверные IP, логика | `file: write_file` |
| `/root/.hermes/config.yaml` | toolsets, model, memory, platform_toolsets | `hermes config set` или `file: write_file` |
| `/root/.hermes/AGENTS.md` | Устаревшая карта инфраструктуры | `file: write_file` |
| `/root/.hermes/skills/*/SKILL.md` | Неверные команды в скиллах | `file: write_file` |
| `/root/.hermes/.env` | Креды, токены | `file: write_file` |

## Типовые исправления

### 1. Добавить toolset
```bash
hermes config set toolsets terminal,file,web,skills
```

### 2. Добавить platform_toolsets для CLI
В `/root/.hermes/config.yaml`, секция `platform_toolsets`, должно быть:
```yaml
platform_toolsets:
  cli: terminal,file,web,skills
```

### 3. Сменить модель
```bash
hermes config set model ollama-cloud/deepseek-v4-pro
```

### 4. Убрать web из disabled_toolsets
В `/root/.hermes/config.yaml` удалить `web` из списка `disabled_toolsets`.

### 5. Исправить SSH адрес
Везде заменить `192.168.1.1` на `192.168.1.202`.

### 6. Исправить противоречивые инструкции
В `/root/.hermes/SOUL.md` найти конфликтующие правила и переписать.

## Процедура исправления

1. Определи что править (из hermes-debug)
2. Сделай бэкап: `cp /root/.hermes/SOUL.md /root/.hermes/SOUL.md.bak`
3. Внеси минимальное изменение
4. Проверь: `hermes --profile default -z "тестовый запрос"`
5. Если не помогло — откати: `cp /root/.hermes/SOUL.md.bak /root/.hermes/SOUL.md`
6. Повтори с другой гипотезой

## Что НЕ делать

- Не менять всё сразу — одно изменение за раз
- Не удалять бэкап пока не подтверждено что исправление работает
- Не менять модель на ту что не протестирована
- Не удалять скиллы без явной команды
