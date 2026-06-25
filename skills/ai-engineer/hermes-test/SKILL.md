---
name: hermes-test
description: Test DevOps Hermes — run validation prompts, check responses, verify fixes
version: 1.0
---

# Test DevOps Hermes

Валидация что DevOps Hermes работает правильно после исправлений.

## Тестовый набор

### Test 1: Базовый ответ
```bash
hermes --profile default -z "привет, расскажи кто ты и за что отвечаешь"
```
**Ожидается:** Осмысленный ответ про DevOps роль, без зацикливания, без галлюцинаций.

### Test 2: Доступ к хосту
```bash
hermes --profile default -z "выполни pct list и скажи сколько контейнеров"
```
**Ожидается:** Команда выполняется через SSH на 192.168.1.202, возвращает список из 6 контейнеров.

### Test 3: Web search
```bash
hermes --profile default -z "какая последняя версия nginx?"
```
**Ожидается:** Использует web_search, возвращает актуальную информацию.

### Test 4: Skills usage
```bash
hermes --profile default -z "используй скилл pve-system и проверь uptime хоста"
```
**Ожидается:** Загружает скилл, выполняет `ssh root@192.168.1.202 "uptime"`, возвращает результат.

### Test 5: Memory write
```bash
hermes --profile default -z "запиши в память что сегодня 25 июня и идёт тестирование"
```
**Ожидается:** Обновляет MEMORY.md, не зацикливается.

### Test 6: Full-time memory
```bash
hermes --profile default -z "запиши в full-time-memory что проведён тест 25 июня"
```
**Ожидается:** Добавляет запись в `/root/.hermes/full-time-memory.md`.

### Test 7: Контекст (два запроса подряд)
```bash
# Первый:
hermes --profile default -z "меня зовут Алексей"
# Второй:
hermes --profile default -z "как меня зовут?"
```
**Ожидается (при наличии буфера диалога):** Помнит имя из предыдущего сообщения.

## Критерии PASS/FAIL

| Критерий | PASS | FAIL |
|---|---|---|
| Ответ осмысленный | Отвечает по теме, без повторов | Зациклился, галлюцинирует, пустой ответ |
| SSH работает | Команды выполняются на хосте | Connection refused, wrong host |
| Web search | Возвращает внешнюю информацию | Не может найти, нет доступа |
| Skills | Загружает и использует скилл | "Skills toolset disabled" |
| Memory | Пишет в память без ошибок | Переполнение, ошибка записи |
| No loops | Завершается за < 60 сек | Бесконечный цикл |

## Отчёт о тестировании

После прогона тестов сформируй отчёт:
```
📊 Тестирование DevOps Hermes — [дата]

| Тест | Результат | Время | Примечание |
|------|-----------|-------|------------|
| 1. Базовый ответ | PASS | 3s | |
| 2. Доступ к хосту | FAIL | 15s | wrong IP in SOUL.md |
...
```
