# EMIR TECH

Production-ready MVP интернет-магазина электроники на **Next.js 15 + App Router** с публичной витриной, каталогом, карточками товаров, корзиной, checkout, email/password авторизацией, личным кабинетом, встроенной админкой `/admin`, PostgreSQL + Prisma, S3-compatible storage и Telegram-уведомлениями.

## Что реализовано

- публичная часть:
  - `/`
  - `/shop`
  - `/shop/[categorySlug]`
  - `/product/[slug]`
  - `/cart`
  - `/checkout`
  - `/checkout/success`
  - `/auth/login`
  - `/auth/register`
  - `/auth/forgot-password`
  - `/auth/reset-password`
  - `/account`
  - `/account/orders`
  - `/account/orders/[orderId]`
  - `/faq`
  - `/delivery-payment`
  - `/privacy-policy`
  - `/personal-data-policy`
  - `/personal-data-consent`
  - `/public-offer`
  - `/requisites`
  - `/about`
  - `/contacts`
  - `not-found`
- каталог с поиском, фильтрами, сортировкой, пагинацией и query params
- карточка товара с JSON-LD Product, Open Graph и related products
- guest cart в localStorage/cookie
- merge гостевой корзины в DB cart после логина
- checkout только для авторизованного пользователя
- бизнес-логика предзаказа: заказ создается как `NEW`, деньги не списываются автоматически
- статусы заказов: `NEW`, `CONFIRMED`, `PAID`, `IN_TRANSIT`, `DELIVERED`, `CANCELED`
- личный кабинет с профилем, заказами, timeline и трек-номером
- админка `/admin`:
  - dashboard
  - категории CRUD
  - бренды CRUD
  - товары CRUD
  - заказы: фильтрация, изменение статуса, предоплаты, трек-номера, комментария
  - пользователи и их заказы
  - редактирование контент-страниц
  - настройки сайта
- интеграции:
  - Telegram Bot API уведомления
  - SMTP reset password
  - S3 / MinIO upload для изображений
- SEO:
  - SSR для основных публичных страниц
  - metadata / canonical / Open Graph
  - `sitemap.xml`
  - `robots.txt`
  - JSON-LD `Organization` и `Product`

## Техстек

- Next.js 15+
- TypeScript
- Tailwind CSS
- shadcn-style UI components
- PostgreSQL
- Prisma ORM
- Auth.js / NextAuth (credentials + JWT sessions)
- React Hook Form
- Zod
- Route Handlers + server components
- AWS SDK S3 client / MinIO
- Telegram Bot API
- Docker / docker-compose

## Структура проекта

```text
emir-tech/
├─ prisma/
│  ├─ migrations/0001_init/migration.sql
│  ├─ schema.prisma
│  └─ seed.ts
├─ src/
│  ├─ app/
│  │  ├─ admin/
│  │  ├─ api/
│  │  ├─ auth/
│  │  ├─ account/
│  │  ├─ shop/
│  │  ├─ product/
│  │  ├─ cart/
│  │  ├─ checkout/
│  │  ├─ faq/
│  │  ├─ delivery-payment/
│  │  ├─ privacy-policy/
│  │  ├─ personal-data-policy/
│  │  ├─ personal-data-consent/
│  │  ├─ public-offer/
│  │  ├─ requisites/
│  │  ├─ about/
│  │  ├─ contacts/
│  │  ├─ layout.tsx
│  │  ├─ page.tsx
│  │  ├─ sitemap.ts
│  │  └─ robots.ts
│  ├─ components/
│  │  ├─ admin/
│  │  ├─ auth/
│  │  ├─ account/
│  │  ├─ cart/
│  │  ├─ checkout/
│  │  ├─ content/
│  │  ├─ home/
│  │  ├─ layout/
│  │  ├─ product/
│  │  ├─ providers/
│  │  ├─ shared/
│  │  ├─ shop/
│  │  └─ ui/
│  ├─ lib/
│  ├─ types/
│  └─ auth.ts
├─ .env.example
├─ docker-compose.yml
├─ Dockerfile
├─ next.config.ts
├─ package.json
└─ README.md
```

## Подготовка окружения

### 1) Скопировать env

```bash
cp .env.example .env
```

### 2) Поднять PostgreSQL и MinIO

```bash
docker compose up -d postgres minio minio-init
```

### 3) Установить зависимости

```bash
npm install
```

### 4) Сгенерировать Prisma client и применить миграции

```bash
npm run db:generate
npx prisma migrate deploy
```

Для чистой локальной разработки можно использовать:

```bash
npm run db:migrate -- --name init
```

### 5) Заполнить seed-данные

```bash
npm run db:seed
```

### 6) Запустить dev

```bash
npm run dev
```

Сайт будет доступен на `http://localhost:3000`.

## Docker-режим

`docker-compose.yml` в этом репозитории поднимает инфраструктуру: PostgreSQL и MinIO.

Само приложение можно запускать локально через `npm run dev` либо собрать контейнером:

```bash
docker build -t emir-tech .
docker run --env-file .env -p 3000:3000 emir-tech
```

## Seed-аккаунты

После `npm run db:seed` доступны:

- admin: `admin@emirtech.local` / `Admin123!`
- customer: `customer@emirtech.local` / `User123!`

## Переменные окружения

См. `.env.example`.

Критичные переменные:

- `DATABASE_URL`
- `NEXTAUTH_URL`
- `NEXTAUTH_SECRET` / `AUTH_SECRET`
- `SMTP_*`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_CHAT_ID`
- `S3_BUCKET`
- `S3_ENDPOINT`
- `S3_ACCESS_KEY`
- `S3_SECRET_KEY`
- `S3_FORCE_PATH_STYLE`

## Telegram

При настроенных `TELEGRAM_BOT_TOKEN` и `TELEGRAM_CHAT_ID` будут отправляться уведомления:

- новый заказ
- заказ подтвержден
- заказ оплачен
- заказ отменен

Если Telegram недоступен, приложение не падает: ошибка логируется и запрос продолжается.

## Storage / изображения

Реализована загрузка файлов через `/api/admin/upload`.

Dev-сценарий:

- используется MinIO
- бакет создается сервисом `minio-init`
- bucket делается публичным для прямого отображения изображений

Prod-сценарий:

- задайте ваш S3 / R2 endpoint и credentials
- при необходимости скорректируйте CORS/bucket policy со стороны storage

## Как работает checkout

1. Пользователь кладет товар в корзину.
2. Гость хранит корзину в localStorage/cookie.
3. После логина корзина merge-ится в DB cart.
4. Оформление доступно только авторизованному пользователю.
5. После submit создается заказ со статусом `NEW`.
6. Менеджер видит заказ в `/admin/orders`.
7. Администратор подтверждает заказ, указывает/корректирует фиксированную предоплату и переводит статусы вручную.

## Контент-страницы

Страницы `FAQ`, `Доставка и оплата`, `Политики`, `Оферта`, `Реквизиты`, `О компании`, `Контакты` берутся из таблицы `ContentPage` и редактируются в `/admin/content/[key]` через markdown.

## Настройки сайта

В `/admin/settings` редактируются:

- название магазина
- телефон / email / Telegram / адрес / режим работы
- глобальная предоплата
- глобальный срок доставки
- hero-блок главной
- SEO defaults
- логотип / favicon

## Безопасность

В проекте реализованы базовые меры:

- bcrypt hash для паролей
- Auth.js / NextAuth credentials auth
- protected account/admin routes
- server-side validation через Zod
- client-side validation через RHF + Zod
- sanitize markdown/html контент-страниц
- базовый in-memory rate limit на auth/reset endpoints

## Что стоит доработать перед production launch

Этот репозиторий уже можно запускать локально и использовать как рабочую базу, но перед публичным запуском рекомендуется:

- подключить продовый SMTP
- заменить демонстрационные юридические тексты на финальные документы
- вынести rate limiting в Redis / reverse proxy слой
- настроить CDN / image optimization strategy для storage
- добавить audit-лог действий в админке
- добавить единые toast-нотификации и e2e smoke tests
- при необходимости расширить role model и granular permissions

## Деплой на VPS

Базовый сценарий:

1. Установить Docker + Node.js 22+
2. Поднять PostgreSQL и MinIO через `docker compose up -d`
3. Скопировать `.env`
4. `npm install`
5. `npm run db:generate`
6. `npx prisma migrate deploy`
7. `npm run db:seed` (один раз)
8. `npm run build`
9. `npm run start`
10. Поставить Nginx reverse proxy на `localhost:3000`
11. Настроить HTTPS

## Acceptance checklist

После seed и заполнения `.env` можно проверить:

- вход под admin
- доступ в `/admin`
- создание категории
- создание бренда
- создание товара с загрузкой изображения
- появление товара в каталоге
- работа поиска / фильтров / сортировок
- добавление в корзину
- оформление заказа авторизованным пользователем
- появление заказа в админке
- Telegram-уведомление при создании заказа
- изменение статуса заказа из админки
- отображение статуса и трек-номера в личном кабинете
- редактирование служебных страниц

