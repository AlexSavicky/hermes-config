---
name: pve-system
description: System monitoring and maintenance — disk, RAM, CPU, services, journalctl, apt updates
version: 1.0
---

# System Monitoring & Maintenance

Системный мониторинг и обслуживание Proxmox хоста и контейнеров.

## Мониторинг

### Нагрузка и память
```bash
ssh root@192.168.1.202 "uptime"
ssh root@192.168.1.202 "free -h"
ssh root@192.168.1.202 "df -h | grep -v tmpfs"
```

### Ошибки в журнале
```bash
ssh root@192.168.1.202 "journalctl -p err --since '24 hours ago' --no-pager | tail -50"
```

### Сервисы на хосте
```bash
ssh root@192.168.1.202 "systemctl is-active smbd pve-tgbot pvedaemon pveproxy"
```

### Температуры CPU
```bash
ssh root@192.168.1.202 "sensors | grep -E 'Core|Package|temp'"
```

## Обслуживание

### Проверка обновлений (dry-run)
```bash
# На хосте:
ssh root@192.168.1.202 "apt update && apt list --upgradable 2>/dev/null | grep -v 'Listing'"

# В контейнере:
ssh root@192.168.1.202 "pct exec <vmid> -- apt update && pct exec <vmid> -- apt list --upgradable 2>/dev/null"
```

### Очистка пакетов
```bash
ssh root@192.168.1.202 "apt autoremove --dry-run"
ssh root@192.168.1.202 "apt autoclean"
```

### Ротация логов
```bash
ssh root@192.168.1.202 "journalctl --vacuum-time=7d"
```

### Проверка перезагрузки после обновлений
```bash
ssh root@192.168.1.202 "test -f /var/run/reboot-required && echo 'REBOOT REQUIRED' || echo 'no reboot needed'"
```

## Best Practices
- НЕ делать apt upgrade без плана и одобрения владельца
- НЕ перезагружать хост без предупреждения
- Очистку пакетов и логов делать автоматически раз в неделю
- Перед apt upgrade: снапшот всех CT
- journalctl vacuum раз в 2 недели
