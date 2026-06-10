# VibeIDEProjects

Добро пожаловать в каталог проектов, созданных в **VibeIDE**.

Каждый проект живёт в своём собственном репозитории на GitHub:
👉 [github.com/VibeIDEProjects](https://github.com/VibeIDEProjects)

В этом репозитории — только **документация и ссылки**.

---

## 📦 Проекты

| №  | Название | Стек | ТЗ |
|----|----------|------|----|
| 1 | [BookCatalog](https://github.com/VibeIDEProjects/BookCatalog) | PHP 8.2, Yii2 Advanced, PostgreSQL, Nginx | [ТЗ →](./docs/bookcatalog.md) |
| 2 | BuzzBang 🚧 | Angular 21, Node.js/Express/Parse Server, Ionic/Capacitor, MongoDB, TypeScript | [ТЗ →](./docs/buzzbang.md) |
| 3 | [TgParser](https://github.com/VibeIDEProjects/TgParser) | Python 3.11+, Telethon, Playwright, Textual, Click | [ТЗ →](./docs/tgparser.md) |

---

## 🧭 О проектах

### BookCatalog
**Книжный каталог** — CRUD-приложение для управления книгами и авторами.
- 🌐 Деплой: [books.abxb.ru](https://books.abxb.ru)
- 🛠 Стек: PHP 8.2, Yii2 Advanced, PostgreSQL 15, Nginx

### BuzzBang 🚧
**Платформа для поиска мест и событий** — location-based сервис с админ-панелью, API и мобильным приложением.
- 🛠 Стек:
  - **Admin**: Angular 21, Angular Material, Tailwind CSS, TypeScript
  - **API**: Node.js, Express, Parse Server 7, TypeScript, MongoDB
  - **Mobile App**: Ionic, Angular 21, Capacitor (Android/iOS)
- ⏳ Статус: **в разработке** — репозиторий появится после первого пуша

### TgParser
**Telegram-канал парсер** — извлечение сообщений из открытых (MTProto) и закрытых (Web) каналов.
- 🌐 Репозиторий: [github.com/VibeIDEProjects/TgParser](https://github.com/VibeIDEProjects/TgParser)
- 🛠 Стек: Python 3.11+, Telethon, Playwright, BeautifulSoup, Click, Textual (TUI), PyInstaller
- 📦 Публикуется в PyPI как `tgparser-cli`

---

## 📁 Структура репозитория

```
/
├── docs/               # Технические задания (ТЗ) по каждому проекту
├── projects/           # Снапшоты README/roadmap уже сделанных проектов (только доки!)
├── .gitignore          # projects/ и .vibe/ полностью игнорируются
└── README.md           # Этот файл
```

> ⚠️ **Важно.** Папка `projects/` в этом репо содержит **только документацию** (README, LICENSE, roadmap) по уже завершённым проектам. Исходный код, тесты, билды, `.venv/`, `dist/` и runtime-данные живут **только** в отдельных репозиториях (`github.com/VibeIDEProjects/<ProjectName>`) и в этот моно-репо не попадают (см. `.gitignore:2`).

---

## 📄 Лицензия

Каждый проект может иметь свою лицензию. Подробнее — в соответствующем репозитории.
