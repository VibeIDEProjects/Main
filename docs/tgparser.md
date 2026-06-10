# ТЗ: TgParser — Telegram-канал парсер

## 1. Описание проекта

TgParser — утилита для извлечения сообщений из открытых (MTProto API) и закрытых (Web HTML) Telegram-каналов. Поставляется как CLI-утилита и пакет для PyPI, опционально — с TUI-интерфейсом на Textual.

- 🌐 Репозиторий: [github.com/VibeIDEProjects/TgParser](https://github.com/VibeIDEProjects/TgParser)
- 🛠 Стек: Python 3.11/3.12, Telethon, Playwright, BeautifulSoup, Click, Textual, PyInstaller

## 2. Цель

Дать удобный инструмент для оффлайн-аналитики, архивирования и экспорта сообщений из Telegram-каналов — как открытых (через MTProto), так и закрытых (через web-версию с обходом `user-select: none`).

## 3. Возможности

- **Авторизация**: QR-код (Web) или MTProto (Telethon) с сохранением сессии
- **Парсинг открытых каналов** — Telethon
- **Парсинг закрытых каналов** — Playwright + BeautifulSoup, снятие CSS-защиты от копирования
- **Экспорт** в JSON, CSV, plain-text, Markdown, SQLite
- **Markdown-экспорт** с hard-breaks (корректно рендерится в GitHub / Obsidian / VS Code)
- **Инкрементальный парсинг** — только новые сообщения
- **TUI-интерфейс** на Textual (Main / Auth / Parse / Result / Files экраны)
- **Сборка standalone** через PyInstaller

## 4. Стек технологий

| Компонент            | Технология                       |
|----------------------|----------------------------------|
| Язык                 | Python 3.11 / 3.12               |
| MTProto-клиент       | Telethon                         |
| Web-парсинг          | Playwright + Chromium, BeautifulSoup |
| CLI                  | Click                            |
| TUI                  | Textual                          |
| Сборка standalone    | PyInstaller (`bin/build_standalone.py`) |
| Тесты                | pytest (smoke + интеграционные)  |
| Линт/формат          | ruff                             |
| Публикация           | PyPI / TestPyPI через `bin/publish_to_pypi.py` |
| CI                   | GitHub Actions (`.github/workflows/{ci,publish}.yml`) |

## 5. Структура репозитория

```
TgParser/
├── src/tgparser/            # Исходный код (auth, parsers, storage, gui, cli, config)
├── tests/                   # pytest
├── bin/                     # Лаунчеры GUI + сборка + публикация
├── scripts/                 # Диагностические одноразовые скрипты
├── data/                    # output/ и sessions/ (только .gitkeep в репо)
├── logs/                    # runtime-логи (только .gitkeep в репо)
├── docs/                    # Документация (roadmap)
├── .github/workflows/       # CI
├── config.yaml              # Конфигурация (опционально)
├── .env.example             # Шаблон секретов
└── pyproject.toml
```

## 6. Дорожная карта

Полный roadmap: [`docs/roadmap.md`](https://github.com/VibeIDEProjects/TgParser/blob/master/docs/roadmap.md).

- [x] **Фаза 6** (v0.1.0): MTProto + Web парсинг, инкрементальный режим, экспорт JSON/CSV/TXT/SQLite
- [x] **Фаза 7**: TUI-интерфейс (Textual), PyInstaller-сборка, публикация в PyPI
- [ ] Telegram Premium (MTProto)
- [ ] Парсинг комментариев
