---
name: pve-bestpractices
description: Linux home lab best practices — security hardening, cleanup, optimization, proactive maintenance
version: 1.0
---

# Home Lab Best Practices

Этот скилл описывает лучшие практики для домашней лаборатории на Proxmox/Debian.
Применяй эти проверки проактивно — не жди пока сломается.

## Аудит безопасности

### Проверка открытых портов
```bash
ssh root@192.168.1.1 "ss -tlnp | grep LISTEN"
```
Ожидаемые порты: 22 (SSH), 8006 (Proxmox), 3128 (pveproxy).
Если видишь 0.0.0.0:* — это порт открыт наружу. Проверь зачем.

### Проверка неудачных логинов
```bash
ssh root@192.168.1.1 "faillock --user root"
ssh root@192.168.1.1 "grep 'Failed password' /var/log/auth.log | tail -20"
```

### Проверка sudo-доступа
```bash
ssh root@192.168.1.1 "grep -v '^#' /etc/sudoers | grep -v '^$'"
```

### Неиспользуемые пакеты
```bash
ssh root@192.168.1.1 "apt autoremove --dry-run"
ssh root@192.168.1.1 "deborphan 2>/dev/null || echo 'deborphan not installed'"
```

### Срок действия сертификатов
```bash
ssh root@192.168.1.1 "find /etc/pve -name '*.pem' -exec openssl x509 -enddate -noout -in {} \; 2>/dev/null"
```

## Очистка системы

### Логи
```bash
# Очистить journal старше 7 дней
ssh root@192.168.1.1 "journalctl --vacuum-time=7d"
# Размер journal
ssh root@192.168.1.1 "journalctl --disk-usage"
```

### Пакеты
```bash
ssh root@192.168.1.1 "apt autoremove -y"
ssh root@192.168.1.1 "apt autoclean"
```

### Кэш apt
```bash
ssh root@192.168.1.1 "du -sh /var/cache/apt/archives/"
```

### Старые ядра (на Proxmox)
```bash
ssh root@192.168.1.1 "dpkg -l | grep -E 'linux-image|linux-headers' | grep -v $(uname -r)"
```

### Временные файлы
```bash
ssh root@192.168.1.1 "find /tmp -type f -atime +7 -delete 2>/dev/null"
ssh root@192.168.1.1 "find /var/tmp -type f -atime +30 -delete 2>/dev/null"
```

## Оптимизация

### Автообновления безопасности
```bash
# Проверить установлен ли unattended-upgrades
ssh root@192.168.1.1 "dpkg -l unattended-upgrades | grep -E '^ii'"
# Если нет — предложить установить (только security updates)
```

### Проверка SWAP использования
```bash
ssh root@192.168.1.1 "free -h | grep Swap"
# Если swap > 0 при достаточном RAM — проверить swappiness
ssh root@192.168.1.1 "cat /proc/sys/vm/swappiness"
```

### ZFS ARC кэш
```bash
ssh root@192.168.1.1 "arc_summary | grep -E 'ARC size|hit ratio' | head -5"
```

### Smartctl короткий тест (ежемесячно)
```bash
ssh root@192.168.1.1 "smartctl -t short /dev/sda"
ssh root@192.168.1.1 "smartctl -t short /dev/sdb"
ssh root@192.168.1.1 "smartctl -t short /dev/sdc"
```

## SSH hardening (проверить, не применять без согласования)

```bash
# Проверить текущие настройки:
ssh root@192.168.1.1 "grep -E '^PermitRootLogin|^PasswordAuthentication|^Port|^PubkeyAuthentication' /etc/ssh/sshd_config"
```

Рекомендации (предложить, не применять без одобрения):
- PermitRootLogin prohibit-password (или no)
- PasswordAuthentication no (если используются ключи)
- Port != 22 (сменить на нестандартный)

## Мониторинг трендов

### Рост диска
```bash
# Сравнить с прошлой проверкой (из full-time-memory.md)
ssh root@192.168.1.1 "zfs list -o name,used,avail -H hdd16tb/samba"
```

### Рост нагрузки
```bash
ssh root@192.168.1.1 "uptime"
ssh root@192.168.1.1 "sar -u 1 3 2>/dev/null || top -b -n1 | head -5"
```

## План еженедельного обслуживания

Каждое воскресенье (или по команде «проведи обслуживание»):

1. **Безопасность:** неудачные логины, открытые порты, CVE в установленных пакетах
2. **Очистка:** journal vacuum, apt autoremove, /tmp
3. **Диски:** SMART короткий тест, ZFS scrub status
4. **Обновления:** проверить apt list --upgradable, предложить критичные
5. **Бэкапы:** проверить последний vzdump, предложить если старше недели
6. **Снапшоты:** удалить старше 2 недель
7. **Сводка в Telegram:** что сделано, что требует внимания

## Что НЕ делать без явного разрешения

- Не менять SSH конфиг (можно заблокировать себе доступ)
- Не удалять старые ядра без проверки что текущее работает > недели
- Не отключать сервисы без понимания зависимостей
- Не править sysctl без тестирования
- Не устанавливать unattended-upgrades без обсуждения (может сломать Proxmox)
