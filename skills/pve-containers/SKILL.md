---
name: pve-containers
description: Proxmox LXC container management — list, start, stop, restart, exec, status, config
version: 1.0
---

# PVE Container Management

Управление LXC-контейнерами на Proxmox хосте. Все команды выполняются через SSH.

## Основные операции

### Список контейнеров
```bash
ssh root@192.168.1.202 "pct list"
```

### Статус контейнера
```bash
ssh root@192.168.1.202 "pct status <vmid>"
```

### Запуск / остановка / перезапуск
```bash
ssh root@192.168.1.202 "pct start <vmid>"
ssh root@192.168.1.202 "pct stop <vmid>"
ssh root@192.168.1.202 "pct reboot <vmid>"
```

### Выполнить команду внутри контейнера
```bash
ssh root@192.168.1.202 "pct exec <vmid> -- <command>"
```

### Просмотр конфигурации
```bash
ssh root@192.168.1.202 "pct config <vmid>"
```

### Просмотр логов контейнера
```bash
ssh root@192.168.1.202 "pct exec <vmid> -- journalctl -n 50 --no-pager"
```

## Известные контейнеры
- CT 100 jellyfin (192.168.1.2021) — CRITICAL, не перезагружать без предупреждения
- CT 102 qbittorrent (192.168.1.2022)
- CT 103 sonarr (192.168.1.2023)
- CT 104 jackett (DHCP)
- CT 105 radarr (192.168.1.2024)
- CT 106 hermes (192.168.1.2025) — я сам

## Best Practices
- Перед остановкой критичного контейнера — предупредить владельца
- Перед любым деструктивным действием — снапшот
- После перезапуска контейнера — проверить что сервис внутри active
- Для jellyfin: не перезагружать если идёт транскодинг (18:00–01:00 MSK)
