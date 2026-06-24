# Hermes DevOps — конфигурация для домашней лаборатории

Конфигурационные файлы для [Hermes Agent](https://hermes-agent.nousresearch.com/) в роли DevOps-администратора PVE-хоста с медиасервером.

## Файлы

| Файл | Назначение | Куда |
|---|---|---|
| `AGENTS.md` | Полная карта инфраструктуры: 6 LXC, ZFS, Samba, Telegram-бот | `~/.hermes/AGENTS.md` |
| `SOUL.md` | DevOps-личность Hermes: мониторинг, предложения, общение, эскалация | `~/.hermes/SOUL.md` |
| `MEMORY.md` | Шаблон структурированной памяти: хронология, проблемы, улучшения | `~/.hermes/memories/MEMORY.md` |

## Инфраструктура

- **Proxmox VE** с ZFS (NVMe mirror + HDD RAIDZ)
- **6 LXC**: jellyfin, qbittorrent, sonarr, radarr, jackett, hermes
- **Samba** на хосте PVE с общими папками для медиа
- **Telegram-бот** (`unified-pve-tgbot-v4.sh`) — [отдельный репо](https://github.com/AlexSavicky/bash)

## Быстрая установка

```bash
# На хосте Proxmox (CT 106):
cd ~/.hermes
wget https://raw.githubusercontent.com/AlexSavicky/hermes-config/main/AGENTS.md
wget https://raw.githubusercontent.com/AlexSavicky/hermes-config/main/SOUL.md
wget https://raw.githubusercontent.com/AlexSavicky/hermes-config/main/MEMORY.md -O memories/MEMORY.md

# Настроить память в config.yaml:
hermes config set memory.memory_enabled true
hermes config set memory.memory_char_limit 4000

# Настроить SSH на хост PVE:
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
ssh-copy-id root@192.168.1.1

# Установить Hermes:
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
hermes setup --portal
```

## Связка с Telegram-ботом

Hermes отвечает на обычные сообщения (без `/`) через существующего бота `pve-tgbot`. Код бота: [AlexSavicky/bash](https://github.com/AlexSavicky/bash).
