# Прогресс: Неон Маркет (Telegram Mini App)

## Статус
Проект создан с нуля и полностью собран. Готова статика `out/` (корневой вариант, `NEXT_PUBLIC_BASE_PATH` пуст).
Дата последней сборки: 2026-10-29. Next.js 14.2.35, React 18.3.1, TypeScript strict.

## Что сделано
- Статический экспорт Next.js: `output: 'export'`, `trailingSlash: true`, `images.unoptimized: true`.
- Две вкладки («Товары», «Каналы») + панели «Избранное» и «Корзина», нижний фиксированный TabBar с бейджами.
- Поиск с debounce 250 мс и подсветкой совпадений, фильтры (категории-мультивыбор, диапазон цены),
  счётчик активных фильтров, сброс, сортировка, показ порциями без сброса прокрутки.
- Избранное и корзина с сохранением в `localStorage` (ключи `neon-market:favorites`, `neon-market:cart`).
- Оформление заявки: текст заказа + `https://t.me/share/url?url=...&text=...`
  (через `Telegram.WebApp.openTelegramLink`, иначе `window.open`). Контакт берётся из `initDataUnsafe`.
- Telegram WebApp: `ready()`, `expand()`, цвета шапки/фона, `viewportStableHeight` → `--tg-height`,
  статус режима в подвале. Без Telegram работает как обычный сайт.
- Модалка товара (описание, характеристики, цена, действия), Esc/клик по фону, ловушка фокуса.
- Toast-уведомления, пустые состояния, aria-label/aria-pressed, skip-link, focus-стили.
- 15 товаров и 9 каналов демо-данных; 24 локальные неоновые SVG-заглушки + иконка приложения,
  PNG-иконки 192/512/180 для manifest и apple-touch-icon.
- SEO: title/description/OG, `lang="ru"`, `theme-color #0a0a0f`, `manifest.json`,
  `robots.txt`, `sitemap.xml` (генерируются из `NEXT_PUBLIC_SITE_URL`), `public/.nojekyll`.
- README на русском, MIT LICENSE, `.github/workflows/deploy.yml` (build → upload-pages-artifact → deploy-pages),
  `basePath`/`assetPrefix` через `NEXT_PUBLIC_BASE_PATH`.

## Важные файлы
- Данные для замены: `src/data/products.ts`, `src/data/channels.ts` (демо, ссылки каналов вымышленные).
- Логика: `src/lib/catalog.ts`, `src/lib/telegram.ts`, `src/lib/format.ts`, `src/lib/asset.ts`.
- UI: `src/components/*` (CatalogApp, ProductsSection, ChannelsSection, AppProvider, карточки, панели).
- Стили: `src/app/globals.css`, `src/app/components.css`.
- Скрипты: `scripts/generate-assets.mjs` (SVG), `scripts/generate-seo.mjs` (robots/sitemap),
  `scripts/preview.mjs`, `scripts/verify-build.mjs`, `scripts/self-test.mjs`, `scripts/generate-icons.py`.
- Архивы для скачивания: `neon-market-mini-app.zip` (проект + out), `neon-market-out.zip` (только статика).

## Проверено
- `npx tsc --noEmit` — без ошибок.
- `npm test` — 7 самотестов логики пройдено.
- `npm run build` — успешно, `out/index.html` содержит h1, заголовки, первые карточки обоих каталогов без JS.
- `npm run verify` — все проверки: контент, 25 локальных ссылок из HTML, 24 картинки из данных,
  robots/sitemap/manifest/.nojekyll/иконки.
- Сборка с `NEXT_PUBLIC_BASE_PATH=/neon-market` — успешна, пути `/neon-market/...` корректны, verify проходит.
- 25 SVG валидны как XML, 3 PNG открываются; локальный preview отдаёт страницу, JS, CSS, SVG, manifest — HTTP 200.
- Визуально осмотрены только сгенерированные ассеты (превью 15 товарных SVG, 9 аватаров, иконка):
  монограммы/иконки читаются, наложений текста нет.

## Ограничения
- Браузерной проверки desktop/mobile НЕ проводилось: в среде нет браузера/инструмента рендера страниц,
  Playwright/Chromium не устанавливались (по требованию). Верстка проверена только по коду и HTML.
- Реальные клики внутри Telegram не проверялись; интеграция написана по документированному API.
- Демо-ссылки каналов `https://t.me/*_demo` вымышленные — заменить перед публикацией.

## Как продолжать
1. `npm install`, затем `npm run dev` для разработки; `npm run build` для статики.
2. Заменить демо-данные в `src/data/*.ts`, перегенерировать SVG (`npm run build` делает это сам).
3. Проверить в браузере мобильный (360–430 px) и десктоп, открыть модалку/корзину, оформить заявку.
4. Запушить в GitHub (main), включить Pages → Source: GitHub Actions; workflow опубликует `out/`
   в `/<repo>/`. Для корня домена очистить `NEXT_PUBLIC_BASE_PATH` в workflow.
5. Привязать к боту: @BotFather → Bot Settings → Menu Button → Web App URL.
