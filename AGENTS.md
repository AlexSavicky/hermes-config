# Инфраструктура домашней лаборатории

## Proxmox Host
- Адрес: 192.168.1.1 (шлюз vmbr0)
- Доступ из CT 106 (hermes): `ssh root@192.168.1.1` — настроить ключ
- Версия: Proxmox VE (ZFS)
- CPU: с CPU pinning на jellyfin (ядра 0,2,4,6), остальные — общие

## Дисковая подсистема

### rpool (NVMe mirror — система)
- Диски: /dev/nvme0n1 (238.5G) + /dev/nvme1n1 (238.5G)
- Файловая: ZFS mirror
- Размер: ~199G
- Использование: 19G / 181G free (10%)
- Содержит: системный root (pve-1), CT/VM диски (subvol-*)

### hdd16tb (HDD RAIDZ — данные)
- Диски: sda, sdb, sdc — 3×7.3TB в RAIDZ
- Файловая: ZFS RAIDZ
- Размер: 15TB (samba) + 8.9TB (backup)
- Использование: 5.7T / 8.8T free (40% — samba)
- Содержит: все медиаданные, торренты, ISO, софт

### Samba (на хосте PVE)
- Сервис: smbd (systemd)
- Шары: Samba (общая), Video, torrents, ISO, soft, photos, Games, temp, else, Cert
- Пользователи: Kan, Arsenal, kpsync, vm, guest
- Конфиг: /etc/samba/smb.conf
- Критично: при падении samba — падает доступ ко всем медиа

## LXC контейнеры

### CT 100 — jellyfin (Медиасервер)
- Статус: CRITICAL
- IP: 192.168.1.11
- OS: Ubuntu
- CPU: 8 ядер (pinned: 0,2,4,6)
- RAM: 12 GB
- Диск: 8G (subvol-100-disk-1, 41% used)
- GPU: проброшен /dev/dri/renderD128
- Транскодинг: tmpfs 4G в /mnt/transcode
- Маунты: /mnt/serial (сериалы), /mnt/films (фильмы)
- Сервис: jellyfin

### CT 102 — qbittorrent (Торрент-клиент)
- Статус: HIGH
- IP: 192.168.1.12
- OS: Debian
- CPU: 2 ядра
- RAM: 2 GB
- Диск: 8G (subvol-102-disk-0, 6% used)
- Маунты: ISO, torrents, Video, temp, soft, else, Films, Games
- Сервис: qbittorrent-nox (порт 8090)
- Web UI: http://192.168.1.12:8090
- API: http://192.168.1.12:8090/api/v2 (admin / пароль в /root/.qbit_api_pass)

### CT 103 — sonarr (Сериалы)
- Статус: HIGH
- IP: 192.168.1.13
- OS: Debian
- CPU: 2 ядра
- RAM: 1 GB
- Диск: 4G (subvol-103-disk-0, 18% used)
- Маунты: /mnt/serial, /mnt/films, /mnt/temp, /mnt/torrents
- Сервис: sonarr

### CT 104 — jackett (Торрент-индексатор)
- Статус: MEDIUM
- IP: DHCP (динамический)
- OS: Debian
- CPU: 1 ядро
- RAM: 512 MB
- Диск: 2G (subvol-104-disk-0, 28% used)
- Сервис: jackett (или Jackett)
- Важно: после перезагрузки может сменить IP — проверять `pct exec 104 -- ip a`

### CT 105 — radarr (Фильмы)
- Статус: HIGH
- IP: 192.168.1.14
- OS: Debian
- CPU: 2 ядра
- RAM: 1 GB
- Диск: 4G (subvol-105-disk-0, 17% used)
- Маунты: /mnt/films, /mnt/temp, /mnt/torrents
- Сервис: radarr
- Особенность: unprivileged=0, apparmor=unconfined

### CT 106 — hermes (ТЫ ЗДЕСЬ)
- Статус: CRITICAL (ты админишь всё)
- IP: 192.168.1.15
- OS: Debian
- CPU: 2 ядра
- RAM: 4 GB
- Диск: 30G (subvol-106-disk-0, 12% used)

## Существующая автоматизация (pve-tgbot)

На хосте PVE уже работает Telegram-бот:
- Скрипт: /root/pve-tgbot.sh
- Конфиг: /etc/pve-tgbot.conf
- Сервис: systemd pve-tgbot
- Пользователи: /etc/pve-tgbot/users/
- Лог: /var/log/pve-tgbot.log

### Команды бота (уже работают):
- /status — uptime, load, RAM, список CT/VM
- /temps — температуры HDD, NVMe, CPU, датчики MB
- /df — ZFS использование
- /zfs — zpool status
- /ct — список CT и VM
- /logs — ошибки journald за 24ч
- /restart <id> — перезапуск CT/VM
- /ctoff <id> / /cton <id> — стоп/старт
- /run <cmd> — выполнить whitelist-команду
- /backup <id> / /backupall — vzdump бэкапы
- /snapshot <id> / /snaplist / /snaprestore — снапшоты
- /ctlog <id> — логи контейнера
- /ctexec <id> <cmd> — команда внутри CT
- /ping <host> — проверка доступности
- /smart <disk> — SMART отчёт
- /torrents / /deltorrent — управление .torrent
- /adduser / /deluser / /userlist — пользователи
- /reboot / /shutdown — только владелец

### Автоматические алерты (фоном):
- Температуры: HDD warn=40°C crit=45°C, NVMe crit=65°C, CPU crit=80°C
- ZFS pool: деградация
- CT/VM: упал / поднялся
- RAM: >90%
- Диски: >85%

### Крон-задачи:
- pve-daily-report.sh — ежедневный отчёт в 12:00
- pve-startup-alert — уведомление при загрузке хоста

## SSH-доступ для Hermes

Чтобы Hermes мог управлять LXC, ему нужен SSH на хост PVE:

```bash
# На хосте PVE (из-под root):
# Уже должен быть ключ, проверь:
cat ~/.ssh/authorized_keys

# Если нет — добавь свой pubkey
```

Или альтернатива: Hermes может вызывать `pct` команды прямо из CT 106 если дать права. Но SSH на хост — самый надёжный способ.

## Порядок восстановления при сбое

1. Проверить `pct list` — все ли CT running
2. При падении медиастека: jellyfin → sonarr/radarr → qbittorrent → jackett
3. При падении Samba: `systemctl restart smbd` на хосте
4. При проблемах с диском: `zpool status hdd16tb`, SMART через `/smart`
5. Критический сбой: снапшоты CT + бэкапы в /hdd16tb/backup/

## ЧТО НЕ ТРОГАТЬ
- Не менять конфиги LXC (CPU pinning на jellyfin критичен для транскодинга)
- Не трогать /etc/pve-tgbot.conf (бот)
- Не менять /etc/samba/smb.conf без снапшота
- Не останавливать pve-tgbot сервис
- Не удалять снапшоты без явной команды
- Не делать `pct destroy` никогда
