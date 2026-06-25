---
name: pve-zfs
description: ZFS pool monitoring and management — status, health, capacity, scrub
version: 1.0
---

# ZFS Monitoring

Мониторинг и управление ZFS пулами на Proxmox хосте.

## Пулы
- **rpool** — NVMe mirror (238+238 GB), системный
- **hdd16tb** — HDD RAIDZ1 (3×7.3TB), данные

## Основные команды

### Статус всех пулов
```bash
ssh root@192.168.1.202 "zpool status"
```

### Сводка по использованию
```bash
ssh root@192.168.1.202 "zpool list"
ssh root@192.168.1.202 "zfs list -o name,used,avail,refer,mountpoint"
```

### Проверка ошибок
```bash
ssh root@192.168.1.202 "zpool status | grep -E 'errors:|state:'"
```

### История scrub
```bash
ssh root@192.168.1.202 "zpool status | grep -A2 'scan:'"
```

### SMART дисков
```bash
ssh root@192.168.1.202 "smartctl -A /dev/sda | grep -E 'Temperature|Reallocated|Pending|Uncorrectable'"
ssh root@192.168.1.202 "smartctl -A /dev/sdb | grep -E 'Temperature|Reallocated|Pending|Uncorrectable'"
ssh root@192.168.1.202 "smartctl -A /dev/sdc | grep -E 'Temperature|Reallocated|Pending|Uncorrectable'"
ssh root@192.168.1.202 "smartctl -A /dev/nvme0 | grep 'Temperature:'"
ssh root@192.168.1.202 "smartctl -A /dev/nvme1 | grep 'Temperature:'"
```

## Пороги алертов
- HDD температура: warn 40°C, crit 45°C
- NVMe температура: crit 65°C
- ZFS health: любое не ONLINE → немедленный алерт
- Диск: >85% занято → предупреждение, >90% → критический алерт

## Best Practices
- Scrub: минимум раз в месяц на обоих пулах
- SMART: проверять еженедельно, следить за Reallocated_Sector и Pending_Sector
- Снапшоты: перед любыми изменениями на уровне ZFS
- Не игнорировать "features not enabled" — запланировать zpool upgrade в окно обслуживания
