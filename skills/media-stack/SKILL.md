---
name: media-stack
description: Media server stack monitoring — jellyfin, sonarr, radarr, qbittorrent, jackett health checks
version: 1.0
---

# Media Stack Monitoring

Проверка здоровья всего медиастека: jellyfin + arr-ы + qbittorrent + jackett.

## Проверка сервисов

### Jellyfin (CT 100, 192.168.1.2021)
```bash
# Статус сервиса
ssh root@192.168.1.202 "pct exec 100 -- systemctl is-active jellyfin"
# Health endpoint
ssh root@192.168.1.202 "pct exec 100 -- curl -s -o /dev/null -w '%{http_code}' http://localhost:8096/health"
# Версия
ssh root@192.168.1.202 "pct exec 100 -- dpkg -l jellyfin-server 2>/dev/null | grep jellyfin | awk '{print \$3}'"
# Нагрузка
ssh root@192.168.1.202 "pct exec 100 -- top -b -n1 | head -5"
# Транскодинг (активен ли)
ssh root@192.168.1.202 "pct exec 100 -- ps aux | grep ffmpeg | grep -v grep | wc -l"
```

### qBittorrent (CT 102, 192.168.1.2022)
```bash
ssh root@192.168.1.202 "pct exec 102 -- systemctl is-active qbittorrent-nox"
curl -s http://192.168.1.2022:8090 -o /dev/null -w '%{http_code}'
```

### Sonarr (CT 103, 192.168.1.2023)
```bash
ssh root@192.168.1.202 "pct exec 103 -- systemctl is-active sonarr"
curl -s http://192.168.1.2023:8989 -o /dev/null -w '%{http_code}'
```

### Radarr (CT 105, 192.168.1.2024)
```bash
ssh root@192.168.1.202 "pct exec 105 -- systemctl is-active radarr"
curl -s http://192.168.1.2024:7878 -o /dev/null -w '%{http_code}'
```

### Jackett (CT 104, DHCP)
```bash
# Получить текущий IP
JACKETT_IP=$(ssh root@192.168.1.202 "pct exec 104 -- ip -4 addr show eth0 | grep -oP 'inet \K[\d.]+'")
ssh root@192.168.1.202 "pct exec 104 -- systemctl is-active jackett 2>/dev/null || pct exec 104 -- systemctl is-active Jackett"
```

### Samba (на хосте)
```bash
ssh root@192.168.1.202 "systemctl is-active smbd"
```

## Сводная проверка (одной командой)
```bash
for ct in 100 102 103 104 105; do
    name=$(ssh root@192.168.1.202 "pct config $ct 2>/dev/null | awk -F': ' '/^hostname/{print \$2}'")
    status=$(ssh root@192.168.1.202 "pct status $ct 2>/dev/null | awk '{print \$2}'")
    echo "$ct $name: $status"
done
echo "samba: $(ssh root@192.168.1.202 'systemctl is-active smbd')"
```

## Порядок восстановления при сбое
1. Samba (если упала — каскадный сбой mount points)
2. Jellyfin (самый важный)
3. qBittorrent + Jackett (конвейер загрузок)
4. Sonarr + Radarr (поиск контента)

## Best Practices
- Проверять весь стек каждые 4 часа
- При падении jellyfin — сначала диагностика, потом действие
- Jackett IP проверять при каждой проверке (DHCP!)
- Если несколько сервисов упали одновременно — искать общую причину (сеть, диск, хост)
