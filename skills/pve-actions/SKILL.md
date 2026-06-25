---
name: pve-actions
description: Proxmox VM/CT actions — snapshots, backups, rollback, resource changes
version: 1.0
---

# PVE Actions — снапшоты, бэкапы, ресурсы

Деструктивные и административные действия с контейнерами.

## Снапшоты

### Создать снапшот
```bash
ssh root@192.168.1.202 "pct snapshot <vmid> <snapname> --description 'hermes $(date +%Y-%m-%d_%H:%M)'"
```

### Список снапшотов
```bash
ssh root@192.168.1.202 "pct listsnapshot <vmid>"
```

### Откатить снапшот
```bash
ssh root@192.168.1.202 "pct rollback <vmid> <snapname>"
```

### Удалить старые снапшоты
```bash
# Оставить последние 7, удалить остальные
ssh root@192.168.1.202 "pct listsnapshot <vmid> | tail -n +2 | head -n -7 | awk '{print \$1}' | xargs -I{} pct delsnapshot <vmid> {}"
```

## Бэкапы (vzdump)

### Бэкап одного контейнера
```bash
ssh root@192.168.1.202 "vzdump <vmid> --compress zstd --storage hdd16tb-backup --mode snapshot"
```

### Бэкап всех контейнеров
```bash
ssh root@192.168.1.202 "vzdump --all --compress zstd --storage hdd16tb-backup --mode snapshot"
```

### Проверить наличие бэкапов
```bash
ssh root@192.168.1.202 "pvesm list hdd16tb-backup --content backup | grep tar.zst | tail -10"
```

## Изменение ресурсов

### Изменить RAM
```bash
ssh root@192.168.1.202 "pct set <vmid> --memory <MB>"
```

### Изменить CPU cores
```bash
ssh root@192.168.1.202 "pct set <vmid> --cores <N>"
```

### Изменить размер диска
```bash
ssh root@192.168.1.202 "pct resize <vmid> rootfs <size>G"
```

## Best Practices
- Снапшот ПЕРЕД любым изменением конфига
- Снапшот ПЕРЕД apt upgrade внутри контейнера
- Бэкап раз в неделю, хранить последние 4
- Снапшоты старше 2 недель — удалять (экономия места)
- НЕ менять CPU pinning на jellyfin (ядра 0,2,4,6)
- НЕ трогать GPU passthrough
- НЕ менять mount points без снапшота и согласования
- После изменения RAM — проверить что контейнер запустился и сервис active
