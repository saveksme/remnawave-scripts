# Remnawave Scripts — Fork by saveksme

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Shell](https://img.shields.io/badge/language-Bash-blue.svg)](#)
[![Fork of](https://img.shields.io/badge/fork%20of-DigneZzZ%2Fremnawave--scripts-orange.svg)](https://github.com/DigneZzZ/remnawave-scripts)

> Форк оригинального скрипта [DigneZzZ/remnawave-scripts](https://github.com/DigneZzZ/remnawave-scripts) с расширенными возможностями для `remnanode.sh`: автонастройка безопасности, управление торрент-блокировкой, сетевые оптимизации и отображение конфига профиля Xray после установки.

---

## ⚡ Установка

```bash
# RemnaNode (форк)
bash <(curl -Ls https://github.com/saveksme/remnawave-scripts/raw/main/remnanode.sh) @ install

# Caddy Selfsteal (оригинал, без изменений)
bash <(curl -Ls https://github.com/saveksme/remnawave-scripts/raw/main/selfsteal.sh) @ install --caddy
```

После установки управление через команду `remnanode`.

---

## 🆕 Отличия от оригинала

Все изменения касаются только **`remnanode.sh`**. Остальные скрипты (`remnawave.sh`, `selfsteal.sh`, `wtm.sh`, `netbird.sh`) не изменялись.

### 1. 🔍 Автоопределение домена Selfsteal

**Оригинал:** в подсказке Xray Reality конфига всегда показывался placeholder `"serverNames": ["<your-domain>"]`.

**Форк:** скрипт читает реальный домен из `/opt/caddy/.env` (`SELF_STEAL_DOMAIN=`) и подставляет его автоматически.

```
"serverNames": ["example.com"]   ← реальный домен вместо плейсхолдера
```

---

### 2. 🛡️ Fail2ban (пункт 18)

Предложение установить fail2ban в процессе установки ноды. Настраиваемые параметры:

| Параметр | По умолчанию | Описание |
|----------|-------------|----------|
| `bantime` | 3600 сек | Время бана |
| `findtime` | 600 сек | Окно поиска попыток |
| `maxretry` | 5 | Попыток до бана |

Управление после установки:

```bash
remnanode fail2ban        # или пункт 18 в меню
```

Возможности меню: статус, список забаненных IP, разбан, логи, включить/выключить, настройка параметров.

---

### 3. 🚫 Xray Torrent Blocker (пункт 19)

Установка [tblocker by kutovoy](https://github.com/kutovoy) из репозитория `repo.remna.dev`. Блокирует торрент-трафик через анализ логов Xray по тегу.

```bash
remnanode torrent-blocker   # или пункт 19 в меню
```

Настраиваемые параметры при установке:

| Параметр | По умолчанию | Описание |
|----------|-------------|----------|
| `BlockDuration` | 10 мин | Время блокировки IP |
| `TorrentTag` | `TORRENT` | Тег Xray inbound для отслеживания |
| `BlockMode` | `iptables` | Метод блокировки |

**Управление забаненными IP** (уникальная функция форка):

| Пункт | Действие |
|-------|----------|
| 5 | 📋 Список всех заблокированных IP с числом пакетов |
| 6 | 🔒 Вручную заблокировать IP |
| 7 | 🔓 Разблокировать IP — по номеру из списка или по IP |

Работает напрямую с iptables-цепочкой `TBLOCKER`, которую создаёт tblocker. Поддерживается также режим `nftables`.

---

### 4. 🌐 Сетевые оптимизации — BBR + CAKE и IPv6 (пункт 20)

```bash
remnanode net-opt   # или пункт 20 в меню
```

**BBR + CAKE:**
- Включает TCP-алгоритм BBR и qdisc CAKE через `/etc/sysctl.d/99-bbr-cake.conf`
- Применяется немедленно, сохраняется после перезагрузки

**Отключение IPv6:**
- Записывает правила в `/etc/sysctl.d/99-disable-ipv6.conf`
- Применяется немедленно через `sysctl -p`
- Обратимо: включить IPv6 обратно можно из того же меню

---

### 5. 🔥 UFW Firewall (пункт 22)

Настройка файрвола в процессе установки и управление из меню.

```bash
remnanode ufw   # или пункт 22 в меню
```

Опросник при установке:

```
SSH порт                   [22]
IP Панели Remnawave        (пусто = открыть для всех)
Открыть 443/tcp (Xray)     [Y/n]
```

Итоговые правила UFW:
- SSH — открыт для всех
- Порт ноды — только с IP панели (если указан) или для всех
- 443/tcp — открыт для Xray Reality
- Остальное входящее — запрещено

> **Примечание:** Порт Caddy selfsteal (9443) не нужно открывать в UFW — он доступен только через loopback `127.0.0.1` и не доступен снаружи.

---

### 6. 📋 Xray Profile Config (пункт 21)

После установки и в любой момент из меню показывается готовый JSON-конфиг для вставки в профиль ноды Remnawave с автоматически подставленными значениями:

```bash
remnanode profile-config   # или пункт 21 в меню
```

Автоматически подставляются:
- **domain** — из `/opt/caddy/.env` (`SELF_STEAL_DOMAIN`)
- **target port** — из `/opt/caddy/.env` (`SELF_STEAL_PORT`)
- **privateKey** — генерируется командой `xray x25519` из локального бинарника

Если selfsteal не установлен — показывается шаблон с placeholder'ами.

---

### 7. 🪵 Порядок отображения после установки

**Оригинал:** после запуска контейнера сразу шли логи (`docker compose logs -f`), и при Ctrl+C вся информация терялась.

**Форк:** итоговый экран и Profile Config показываются **до** логов. Логи запускаются только по нажатию Enter:

```
Нажмите Enter чтобы посмотреть логи контейнера (Ctrl+C чтобы выйти)...
```

---

## 📋 Полный список пунктов меню `remnanode`

| № | Команда | Описание |
|---|---------|----------|
| 1 | `install` | Установить RemnaNode |
| 2 | `up` | Запустить |
| 3 | `down` | Остановить |
| 4 | `restart` | Перезапустить |
| 5 | `uninstall` | Удалить |
| 6 | `status` | Статус |
| 7 | `logs` | Логи контейнера |
| 8 | `xray-log-out` | Xray access лог |
| 9 | `xray-log-err` | Xray error лог |
| 10 | `update` | Обновить скрипт и контейнер |
| 11 | `core-update` | Обновить Xray-core |
| 12 | `migrate` | Миграция переменных окружения |
| 13 | `edit` | Редактировать docker-compose.yml |
| 14 | `edit-env` | Редактировать .env |
| 15 | `ports` | Показать порты |
| 16 | `setup-logs` | Настроить logrotate |
| 17 | `auto-restart` | Расписание автоперезапуска |
| 18 | `fail2ban` | 🆕 Fail2ban менеджер |
| 19 | `torrent-blocker` | 🆕 Xray Torrent Blocker |
| 20 | `net-opt` | 🆕 Сетевые оптимизации (BBR / IPv6) |
| 21 | `profile-config` | 🆕 Показать конфиг профиля Xray |
| 22 | `ufw` | 🆕 UFW Firewall менеджер |

---

## 🛰 RemnaNode — Процесс установки

Интерактивная установка последовательно предлагает:

1. Параметры ноды — порт, SECRET_KEY, установка Xray-core
2. Caddy Selfsteal — для Reality-маскировки (если не установлен)
3. Fail2ban — защита от брутфорса
4. Xray Torrent Blocker — блокировка торрент-трафика
5. BBR + CAKE — оптимизация TCP
6. Отключение IPv6
7. UFW Firewall — настройка правил доступа
8. Запуск контейнера
9. Итоговая информация + Xray Profile Config
10. Логи — по нажатию Enter

---

## 📁 Структура файлов

```
/opt/remnanode/
├── .env
└── docker-compose.yml

/var/lib/remnanode/
└── xray                  # Xray-core бинарник (если установлен)

/var/log/remnanode/
├── access.log            # Xray access лог (читается tblocker)
└── error.log

/opt/caddy/               # Caddy selfsteal (если установлен)
├── .env                  # SELF_STEAL_DOMAIN, SELF_STEAL_PORT
├── Caddyfile
└── docker-compose.yml

/usr/local/bin/remnanode  # CLI-команда
```

---

## ⚙️ Системные требования

| | Минимум |
|--|---------|
| **CPU** | 1 ядро |
| **RAM** | 512 MB |
| **Диск** | 2 GB |
| **ОС** | Ubuntu 20.04+, Debian 11+, CentOS 7+, AlmaLinux 8+ |

**Зависимости** (устанавливаются автоматически): Docker Engine, Docker Compose V2, curl, openssl

---

## 🔗 Ссылки

| | |
|--|--|
| **Этот форк** | [github.com/saveksme/remnawave-scripts](https://github.com/saveksme/remnawave-scripts) |
| **Оригинальный скрипт** | [github.com/DigneZzZ/remnawave-scripts](https://github.com/DigneZzZ/remnawave-scripts) |
| **Remnawave** | [github.com/remnawave](https://github.com/remnawave) |
| **tblocker** | [repo.remna.dev](https://repo.remna.dev) |
| **Поддержка оригинала** | [gig.ovh](https://gig.ovh) |

---

## 📜 Лицензия

[MIT License](./LICENSE) — форк оригинального проекта DigneZzZ.
