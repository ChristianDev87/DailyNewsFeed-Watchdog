# DailyNewsFeed — Watchdog

Python-Service der `bot_commands` aus der MySQL-Datenbank liest und systemd-Befehle ausführt. Bindeglied zwischen dem PHP Web-Interface und dem Discord Bot.

## Funktionsweise

Das Web-Interface schreibt Befehle in die `bot_commands`-Tabelle (Status: `pending`). Der Watchdog pollt diese Tabelle alle 10 Sekunden, führt die entsprechenden systemd-Aktionen aus und setzt den Status auf `done` oder `failed`.

| Befehl | Aktion |
|---|---|
| `restart_bot` | `systemctl restart daily-news-bot` |
| `run_digest` | `systemctl restart daily-news-bot` |

## Voraussetzungen

- Python 3.10+
- MySQL 8 / MariaDB 10.6+
- systemd (Linux)
- Der `daily-news-bot` systemd-Service muss bereits eingerichtet sein

## Installation

### 1. Repository klonen

```bash
git clone https://github.com/ChristianDev87/DailyNewsFeed-Watchdog.git /opt/daily-news-watchdog
cd /opt/daily-news-watchdog
```

### 2. Virtuelle Umgebung + Abhängigkeiten

```bash
python3 -m venv venv
venv/bin/pip install -r requirements.txt
```

### 3. Umgebungsvariablen konfigurieren

```bash
cp .env.example .env
nano .env
```

| Variable | Beschreibung | Standard |
|---|---|---|
| `DB_HOST` | Datenbank-Host | `localhost` |
| `DB_PORT` | Datenbank-Port | `3306` |
| `DB_NAME` | Datenbankname | `daily_news` |
| `DB_USER` | Datenbankbenutzer | — |
| `DB_PASS` | Datenbankpasswort | — |
| `WATCHDOG_INTERVAL` | Polling-Intervall in Sekunden | `10` |

### 4. Manuell testen

```bash
venv/bin/python watchdog.py
```

Der Watchdog meldet sich mit `Watchdog gestartet (Intervall: 10s)`.

### 5. Als systemd-Service einrichten

```bash
cp daily-news-watchdog.service /etc/systemd/system/
systemctl daemon-reload
systemctl enable daily-news-watchdog
systemctl start daily-news-watchdog
systemctl status daily-news-watchdog --no-pager
```

## Logs

```bash
journalctl -u daily-news-watchdog -f
```

## Technologie-Stack

- **Python 3.10+**
- **mysql-connector-python** — Datenbankzugriff
- **python-dotenv** — Konfiguration über `.env`
