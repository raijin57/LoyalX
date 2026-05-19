# LoyalX

Кросс-продуктовая программа лояльности на блокчейне TON с интерфейсом в виде Telegram Mini App. Бренды выпускают собственные жетоны (баллы), пользователи получают их за покупки и обменивают между разными брендами по заранее заданному курсу.

> Это учебный прототип, а не production-система.

---
[![Telegram](https://raw.githubusercontent.com/CLorant/readme-social-icons/main/large/filled/telegram.svg)](https://t.me/loyalxapp_bot)
---

## Содержание

- [Идея](#идея)
- [Архитектура](#архитектура)
- [Стек технологий](#стек-технологий)
- [Структура репозитория](#структура-репозитория)
- [Запуск локально](#запуск-локально)
- [Деплой](#деплой)
- [Текущие ограничения](#текущие-ограничения)
- [Документация](#документация)
- [Лицензия](#лицензия)

---

## Идея

Классические программы лояльности изолированы: баллы «Магазина А» нельзя потратить в «Магазине Б». LoyalX решает это на уровне смарт-контрактов: каждый бренд — это отдельный TEP-74 жетон, развёрнутый через единую фабрику. Между жетонами разных брендов настраиваются курсы обмена, и пользователь свободно конвертирует баллы одного бренда в баллы другого прямо из кошелька.

Доступ к системе — через Telegram Mini App: подключение кошелька по TonConnect, выпуск брендов, минтинг баллов, обмен и история операций без выхода из мессенджера.

## Архитектура

```
┌─────────────────────┐              ┌──────────────────────┐
│  Telegram Mini App  │ ───────────► │  TON Blockchain      │
│  (React + Vite)     │ TonConnect   │    Factory + Brands  │
└──────────┬──────────┘              └──────────┬───────────┘
           │                                    │
           │ HTTP                               │ events
           ▼                                    ▼
┌──────────────────────┐        ┌──────────────────────┐
│  Notify Server       │ ◄───── │  TON Indexer / API   │
│  (Express + Postgres)│        │  (tonhubapi.com)     │
└──────────────────────┘        └──────────────────────┘
```

Три независимых модуля:

1. **Смарт-контракты** (`Contracts/`) — Tact + Blueprint. Фабрика разворачивает жетоны брендов, каждый жетон хранит таблицу курсов обмена и обрабатывает входящие переводы как заявку на обмен.
2. **Frontend** (`tgapp/`) — React 19 + TypeScript + Vite + Tailwind CSS v4. Подключение кошелька через `@tonconnect/ui-react`, взаимодействие с контрактами через сгенерированные Blueprint-обёртки.
3. **Notify-сервер** (`server/`) — Express + PostgreSQL. Опрашивает блокчейн и отправляет уведомления о транзакциях в Telegram.

## Стек технологий

| Слой | Технологии |
| --- | --- |
| Блокчейн | TON (testnet) |
| Контракты | Tact 1.6, Blueprint, TEP-74 |
| Frontend | React 19, TypeScript 5.9, Vite 7, Tailwind 4, React Router 7 |
| Кошелёк | TonConnect UI 2.4 |
| Backend | Node.js, Express 4, PostgreSQL, ts-node |
| Тесты контрактов | Jest + `@ton/sandbox` |

## Структура репозитория

```
LoyalX/
├── Contracts/         Tact-контракты, тесты, скрипты деплоя
│   ├── contracts/     factory.tact, brand_jetton.tact, jetton_wallet.tact, messages.tact
│   ├── scripts/       deployFactory.ts, deployBrandJetton.ts
│   └── tests/         LoyalX.spec.ts (интеграционные тесты в Sandbox)
├── tgapp/             Telegram Mini App (React + Vite)
│   └── src/
│       ├── screens/   Wallet, CreateBrand, Mint, Swap, ExchangeRates, History
│       ├── services/  contractService, tonService и т.д.
│       └── hooks/     useContract, useWallet
├── server/            Notify-сервер (Express + Postgres)
│   └── src/           index.ts, poller.ts, telegram.ts, db.ts
├── Documentation/     Курсовые документы (ТЗ, ПЗ, ПМИ, ТП, РО)
└── AGENTS.md          Инструкции для AI-агентов и заметки по архитектуре
```

## Запуск локально

### Требования

- Node.js 20+
- PostgreSQL 14+ (для notify-сервера)
- Кошелёк TON с тестовыми монетами (например, Tonkeeper в режиме testnet)

### Контракты

```bash
cd Contracts
npm install
npm run build              # компиляция Tact
npm test                   # тесты в Sandbox
npm run deploy:factory     # деплой фабрики (потребуется кошелёк)
```

### Frontend

```bash
cd tgapp
npm install
npm run dev                # http://localhost:5173
npm run build              # production-сборка
```

Адрес фабрики берётся из переменной окружения `VITE_FACTORY_ADDRESS`; есть fallback в `src/hooks/useContract.ts` для локальной разработки.

### Notify-сервер

```bash
cd server
npm install
cp .env.example .env       # заполнить DATABASE_URL и TELEGRAM_BOT_TOKEN
npm run dev
```

Миграции лежат в `server/migrations/` и применяются вручную через `psql`.

## Деплой

Текущее окружение:

- Контракты — TON testnet, развёрнуты через Blueprint.
- Frontend — собирается под путь `/LoyalX` (см. `vite.config.ts`); подходит для GitHub Pages, Vercel или Netlify.
- Backend — поднят на Railway; в `server/` есть конфигурация под Node-окружение.

Mainnet-деплой не выполнялся.

## Лицензия

Распространяется по лицензии MIT — см. файл [LICENSE](LICENSE).
