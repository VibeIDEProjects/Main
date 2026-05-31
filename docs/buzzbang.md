# ТЗ: BuzzBang — Платформа для поиска мест и событий

## 1. Описание проекта

> 🚧 **Статус: в разработке.** Репозиторий появится после первого пуша.

BuzzBang — location-based платформа, позволяющая пользователям находить интересные места, события и людей рядом. Проект состоит из трёх компонентов: админ-панель (Angular), REST API (Node.js/Parse Server) и мобильное приложение (Ionic + Capacitor).

## 2. Цель

Создать масштабируемую платформу для поиска мест и событий с администрированием, push-уведомлениями, платежами и геолокацией.

## 3. Стек технологий

### 3.1. Admin Panel (admin/)
| Компонент | Версия / Название |
|-----------|-------------------|
| Фреймворк | Angular 21 |
| UI Kit | Angular Material |
| Стили | Tailwind CSS |
| Язык | TypeScript |
| Карты | Google Maps API |
| Интернационализация | Transloco |

### 3.2. API (api/)
| Компонент | Версия / Название |
|-----------|-------------------|
| Сервер | Node.js + Express |
| База данных | MongoDB 7 (Docker) |
| Backend-as-a-Service | Parse Server 7 |
| Язык | TypeScript |
| Аутентификация | Parse Auth (email, Apple, Facebook, Google) |
| Платежи | Stripe |
| Push-уведомления | OneSignal |
| Почта | Mailgun |
| Файлы | S3 / FS |

### 3.3. Mobile App (app/)
| Компонент | Версия / Название |
|-----------|-------------------|
| Фреймворк | Angular 21 + Ionic |
| Нативные возможности | Capacitor 6 |
| Платформы | Android, iOS |
| Геолокация | Capacitor Geolocation |
| Камера | Capacitor Camera |
| Карты | Google Maps |
| Аутентификация | Apple Sign-In, Facebook Login |
| PWA | Service Worker |

## 4. Функциональные требования

### 4.1. Админ-панель (admin/)
- Дашборд с аналитикой
- Управление пользователями (CRUD, блокировка)
- Управление местами и событиями
- Модерация контента
- Настройка push-уведомлений
- Просмотр платежей и подписок
- Мультиязычный интерфейс (Transloco)

### 4.2. API (api/)
- REST API на Parse Server
- Аутентификация (email, Apple, Facebook, Google)
- CRUD для мест, событий, пользователей
- Геопоиск (ближайшие места)
- Push-уведомления (OneSignal)
- Платежи (Stripe)
- Отправка email (Mailgun)
- Загрузка файлов (S3)
- Локализация (i18n)
- Parse Dashboard (/dashboard)

### 4.3. Мобильное приложение (app/)
- Регистрация и вход (email, Apple, Facebook)
- Главная — лента мест и событий рядом
- Карта с метками ближайших мест
- Детальная карточка места/события
- Избранное
- Профиль пользователя
- Поиск по категориям
- Push-уведомления
- Офлайн-режим (Service Worker)

## 5. Нефункциональные требования

- Контейнеризация (Docker) для API и БД
- CI/CD через GitHub Actions
- Адаптивный дизайн (Mobile First)
- TypeScript — строгая типизация
- ESLint — статический анализ кода

## 6. Модели данных (основные)

### User
| Поле | Тип | Описание |
|------|-----|----------|
| id | PK | Уникальный идентификатор |
| email | string | Email пользователя |
| username | string | Имя пользователя |
| authData | object | Данные аутентификации (Apple, Facebook, Google) |
| avatar | file | Аватар |
| location | GeoPoint | Текущее местоположение |
| createdAt | datetime | Дата регистрации |
| updatedAt | datetime | Дата обновления |

### Place (Место)
| Поле | Тип | Описание |
|------|-----|----------|
| id | PK | Уникальный идентификатор |
| name | string | Название места |
| description | string | Описание |
| category | string | Категория |
| location | GeoPoint | Координаты |
| address | string | Адрес |
| photos | array | Фотографии |
| rating | number | Рейтинг |
| createdBy | Pointer | Кто создал |

### Event (Событие)
| Поле | Тип | Описание |
|------|-----|----------|
| id | PK | Уникальный идентификатор |
| title | string | Название события |
| description | string | Описание |
| place | Pointer | Место проведения |
| startDate | datetime | Дата начала |
| endDate | datetime | Дата окончания |
| category | string | Категория |
| maxAttendees | number | Максимум участников |

## 7. Архитектура

```
Клиенты:
  Admin Panel (Angular 21)  ────┐
  Mobile App (Ionic/Capacitor) ─┼── HTTP/HTTPS ──▶ API Layer
                                │                   │
                                │           Parse Server 7 + Express
                                │           - REST API
                                │           - Live Queries
                                │           - Cloud Code
                                │                   │
                                └───────────────────┤
                                                    │
                          ┌─────────────────────────┼──────────┐
                          ▼                         ▼          ▼
                    MongoDB 7                   Redis     Внешние сервисы:
                    (Docker)                    (кэш)     Stripe, OneSignal,
                                                          Mailgun, S3
```

## 8. API Endpoints (основные)

Parse Server автоматически генерирует REST endpoints для каждого класса.

| Метод | Endpoint | Описание |
|-------|----------|----------|
| GET | /api/classes/Place | Список мест |
| POST | /api/classes/Place | Создать место |
| GET | /api/classes/Place/:id | Детали места |
| PUT | /api/classes/Place/:id | Обновить место |
| DELETE | /api/classes/Place/:id | Удалить место |
| GET | /api/classes/Event | Список событий |
| POST | /api/classes/Event | Создать событие |
| GET | /api/classes/_User | Список пользователей |
| POST | /api/login | Вход |
| POST | /api/users | Регистрация |
| POST | /api/requestPasswordReset | Сброс пароля |
| GET | /api/dashboard | Parse Dashboard |

## 9. Деплой

| Компонент | Платформа | Статус |
|-----------|-----------|--------|
| API | Dokku / Docker | ⏳ |
| Admin | Vercel / Netlify | ⏳ |
| Mobile App | App Store / Google Play | ⏳ |

## 10. Статус выполнения

| Задача | Статус |
|--------|--------|
| API — Parse Server настройка | ✅ Готово |
| API — Модели данных | ✅ Готово |
| API — Аутентификация | ✅ Готово |
| API — Геопоиск | ✅ Готово |
| Admin — Дашборд | ✅ Готово |
| Admin — CRUD пользователей | ✅ Готово |
| Admin — Управление местами | ✅ Готово |
| Mobile — Карта | ✅ Готово |
| Mobile — Профиль | ✅ Готово |
| Push-уведомления | ⏳ В работе |
| Платежи (Stripe) | ⏳ В работе |
| Деплой | ⏳ В работе |

## 11. Локальный запуск

### API
```powershell
cd api
Copy-Item .env.default .env
yarn install
yarn dev:win
```

### Admin
```powershell
cd admin
yarn install
ng serve
```
Открыть http://localhost:4200

### Mobile App
```powershell
cd app
yarn install
ng serve
```
Открыть http://localhost:8100

## 12. Ссылки

- **Репозиторий**: https://github.com/VibeIDEProjects/BuzzBang
- **Сайт проекта**: https://buzzbang.ru
- **API (dev)**: http://localhost:3000/api
- **Parse Dashboard**: http://localhost:3000/dashboard
