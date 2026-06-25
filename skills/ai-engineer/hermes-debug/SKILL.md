---
name: hermes-debug
description: Diagnose DevOps Hermes — read configs, analyze SOUL.md, check logs, find root cause
version: 1.0
---

# Debug DevOps Hermes

Диагностика проблем DevOps Hermes (default профиль). Используй когда владелец
сообщает что DevOps Hermes тупит, зацикливается, не выполняет команды.

## Шаг 1: Быстрая диагностика

```bash
# Проверить что DevOps Hermes вообще жив
hermes --profile default -z "ping" 2>&1 | head -5

# Проверить toolsets (должны быть terminal,file,web,skills)
grep -E "^toolsets:|platform_toolsets" /root/.hermes/config.yaml

# Проверить модель
grep "^model:" /root/.hermes/config.yaml

# Проверить что web не в disabled
grep -A10 "disabled_toolsets" /root/.hermes/config.yaml
```

## Шаг 2: Анализ конфигов

```bash
# Размер SOUL.md (слишком большой = проблема)
wc -l /root/.hermes/SOUL.md

# Противоречия в SOUL.md
grep -n "не чини\|чини\|исправляй\|делай\|предлагай\|спрашивай\|одобр" /root/.hermes/SOUL.md

# Проверить SSH адрес
grep "ssh root@" /root/.hermes/SOUL.md
grep "ssh root@" /root/.hermes/AGENTS.md

# Проверить IP хоста
grep "192.168.1" /root/.hermes/AGENTS.md | head -5
```

## Шаг 3: Анализ логов

```bash
# Последние сессии (зацикливание?)
ls -lt /root/.hermes/sessions/ | head -5

# Размер full-time-memory
wc -l /root/.hermes/full-time-memory.md

# Последние записи (что он делал?)
tail -30 /root/.hermes/full-time-memory.md

# Рабочая память
cat /root/.hermes/memories/MEMORY.md
```

## Шаг 4: Тестовый прогон

```bash
# Простой тест
hermes --profile default -z "назови три любых фильма" 2>&1 | head -20

# Тест с доступом к хосту
hermes --profile default -z "сколько контейнеров на хосте? выполни pct list" 2>&1 | head -20

# Тест с web search
hermes --profile default -z "какая последняя версия nginx?" 2>&1 | head -20
```

## Шаг 5: Формирование диагноза

После сбора данных сформулируй:
- **Симптом:** что именно идёт не так
- **Гипотеза:** почему (со ссылкой на конкретную строку/файл)
- **Исправление:** что править
- **Проверка:** как убедиться что исправлено
