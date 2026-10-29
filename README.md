# Неон Маркет — Telegram Mini App

Статическое веб-приложение на **Next.js 14 + React + TypeScript** со статическим экспортом
(`output: 'export'`, `trailingSlash: true`, `images.unoptimized: true`). Работает внутри Telegram
как Mini App и в обычном браузере. Серверной части нет: после сборки получается папка `out/`
с HTML/CSS/JS и локальными SVG, которую можно разместить на любом статическом хостинге.

Тёмный неоновый дизайн: фон `#0a0a0f` / `#101018`, поверхности `#14141c`, циан `#22d3ee`,
маджента `#f0f`, лайм `#a3e635` для статусов, тонкие неоновые рамки и свечение при наведении.

## Возможности

- Две вкладки с нижней фиксированной навигацией: **Товары** и **Каналы** (переключение без перезагрузки).
- **Поиск** по названию и описанию в обоих каталогах, debounce 250 мс, подсветка совпадений.
- **Фильтры**: чипы категорий с мультивыбором, диапазон цены для товаров, счётчик активных фильтров, сброс.
- **Сортировка**: цена ↑/↓, название, рейтинг/популярность; для каналов — по подписчикам.
- **Показ порциями** («Показать ещё») без сброса прокрутки + индикатор «показано X из Y».
- **Избранное** для товаров и каналов: отдельная панель, счётчик в навигации, `localStorage`.
- **Корзина**: добавление, количество +/−, удаление, итоговая сумма, счётчик, сохранение.
- **Оформление заявки** — реальное действие: формируется текст заказа и открывается
  `https://t.me/share/url?url=...&text=...` (через `Telegram.WebApp.openTelegramLink`, если API доступен,
  иначе `window.open` в новой вкладке). Никакой имитации «успешной отправки».
- Карточка товара открывает **модальное окно** с описанием, характеристиками, ценой и кнопками.
- Канал открывается в новой вкладке; бейдж «Проверен» и число подписчиков.
- Toast-уведомления, пустые состояния «Ничего не найдено» с кнопкой сброса.
- Доступность: `aria-label`, `aria-pressed`, фокус-стили, Esc закрывает модалку, ловушка фокуса,
  skip-link, `aria-live` для уведомлений.
- Учтены `env(safe-area-inset-bottom)` и высота viewport Telegram (`viewportStableHeight`).
- SEO/OG-метаданные, `theme-color #0a0a0f`, `manifest.json`, `apple-mobile-web-app-capable`,
  `robots.txt`, `sitemap.xml`, `lang="ru"`.

Основной контент (h1, заголовки, первые карточки товаров и каналов, тексты) присутствует в
`out/index.html` **без выполнения JavaScript**: данные рендерятся на этапе экспорта, а не в `useEffect`.
Состояние из `localStorage` подхватывается только после монтирования, поэтому HTML не «прыгает».

## Демо-данные (важно)

Все товары, каналы, цены, рейтинги, число подписчиков и ссылки — **демонстрационные**.
Названия каналов и их ссылки `https://t.me/..._demo` вымышленные: замените их на свои перед
публикацией. Изображения — локальные неоновые SVG-заглушки, внешние картинки не используются.

| Что менять | Файл |
| --- | --- |
| Товары (15 шт., категории: Электроника, Аксессуары, Дом, Спорт, Книги) | `src/data/products.ts` |
| Каналы (9 шт., категории: Технологии, Спорт, Обучение, Новости, Развлечения) | `src/data/channels.ts` |
| Типы `Product`, `Channel` | `src/lib/types.ts` |
| SVG-заглушки товаров/каналов и иконка | `scripts/generate-assets.mjs` → `public/assets/` |
| Текст заявки и работа с Telegram | `src/lib/telegram.ts` |

Как заменить данные:

1. Отредактируйте массив `products` или `channels` — интерфейсы описаны в `src/lib/types.ts`.
2. Поле `image` у товара: `/assets/products/<id>.svg`, у канала `avatar`: `/assets/channels/<id>.svg`.
3. Выполните `node --experimental-strip-types scripts/generate-assets.mjs` (или просто `npm run build` —
   генератор запускается автоматически в `prebuild`), чтобы создать SVG для новых позиций.
   Скрипт читает те же `src/data/*.ts`, поэтому рассинхронизации не будет. Лишние старые SVG
   можно удалить из `public/assets/`.
4. Хотите настоящие фотографии — положите файлы в `public/assets/products/` и пропишите пути в данных.
   Путь оборачивается в helper `asset()` из `src/lib/asset.ts`, поэтому `basePath` учитывается автоматически.

## Установка и запуск

Требуется Node.js 20+ (рекомендуется 22).

```bash
npm install       # установка зависимостей
npm run dev       # http://localhost:3000, режим разработки
npm run build     # генерация ассетов + статический экспорт в out/
npm run preview   # локальный просмотр собранной статики на :3000
npm run typecheck # проверка TypeScript
npm test          # самотесты логики: поиск, фильтры, сортировка, сумма заявки
npm run verify    # проверка out/: контент без JS, битые ссылки, наличие SVG
```

## Скриншоты

В репозитории нет растровых скриншотов; раздел описывает, что должно быть на них и как их получить.

1. **Мобильный, вкладка «Товары» (390×844).** Шапка «Неон Маркет» с кнопками избранного и корзины,
   поисковая строка, панель фильтров с чипами категорий и полями цены, селект сортировки,
   карточки в одну колонку с неоновыми SVG, кнопкой «В корзину» и сердечком. Внизу — фиксированный
   TabBar «Товары / Каналы / Избранное / Корзина» с бейджами.
   Как снять: `npm run dev` → DevTools → Toggle device toolbar → iPhone 12/13 → скриншот.
2. **Модальное окно товара.** Поверх затемнённого каталога — карточка с крупным SVG, описанием,
   таблицей характеристик, тегами, ценой и кнопками «В избранное» / «В корзину».
3. **Вкладка «Каналы» и шторка корзины.** Карточки каналов с круглыми SVG-аватарами, бейджем
   «Проверен» и числом подписчиков; справа снизу — корзина с шагером количества, итогом
   и кнопкой «Оформить заявку в Telegram».
4. **Десктоп (1440 px).** Сетка товаров в 3 колонки, каналов — в 2, нижняя навигация остаётся
   по центру экрана.

## basePath и GitHub Pages

Подпапка настраивается **одной переменной окружения** `NEXT_PUBLIC_BASE_PATH` (по умолчанию пусто):

```bash
# корень домена
npm run build

# подпапка вида /repo-name/ (GitHub Pages проекта)
NEXT_PUBLIC_BASE_PATH=/repo-name npm run build
```

Переменная попадает в `basePath` и `assetPrefix` в `next.config.mjs`, а helper `asset()`
подставляет её всем локальным картинкам и ссылкам на манифест. Пути в `manifest.json`
относительные, поэтому иконки работают и в подпапке. После сборки в подпапке проверьте:
`npm run preview` откроет сайт сразу по адресу `http://localhost:3000/repo-name/`.

Дополнительно есть `NEXT_PUBLIC_SITE_URL` — абсолютный адрес публикации для `robots.txt`
и `sitemap.xml` (генерируются скриптом `scripts/generate-seo.mjs`). Если переменная не задана,
на GitHub Actions адрес вычисляется из `GITHUB_REPOSITORY`, локально используется
`http://localhost:3000/` — выдуманный домен нигде не подставляется.

### Деплой на GitHub Pages

1. Создайте репозиторий и запушьте код в ветку `main`.
2. В репозитории откройте **Settings → Pages → Build and deployment → Source: GitHub Actions**.
3. Workflow `.github/workflows/deploy.yml` уже настроен: на пуш в `main` выполняются
   `npm ci` → `npm run build` (с `NEXT_PUBLIC_BASE_PATH=/<имя-репозитория>` и
   `NEXT_PUBLIC_SITE_URL`) → `actions/upload-pages-artifact` (`./out`) → `actions/deploy-pages`.
4. После успешного прогона сайт доступен по адресу `https://<owner>.github.io/<repo>/`.

Особые случаи:

- Репозиторий `<owner>.github.io` (сайт в корне) или собственный домен — уберите/очистите
  `NEXT_PUBLIC_BASE_PATH` в шаге **Build static export** и поправьте `NEXT_PUBLIC_SITE_URL`.
- Файл `public/.nojekyll` уже добавлен, чтобы Pages не обрабатывал сайт Jekyll'ом.
- Любой другой статический хостинг (Nginx, Netlify, Cloudflare Pages): опубликуйте содержимое `out/`.
- Папка `out/` не внесена в `.gitignore`, чтобы готовая статика сохранялась вместе с проектом.
  Если не хотите хранить сборку в git, добавьте строку `out/` в `.gitignore` — workflow соберёт её заново.

## Привязка к Telegram-боту (BotFather)

Mini App открывается по HTTPS-URL, поэтому сначала опубликуйте `out/`.

1. В Telegram откройте **@BotFather** → `/mybots` → выберите бота.
2. **Bot Settings → Menu Button → Configure/Edit menu button** и укажите **Web App URL**
   (например, `https://<owner>.github.io/<repo>/`). Текст кнопки — например, «Открыть маркет».
   В новых версиях BotFather то же самое доступно через **Bot Settings → Mini App → Set Mini App URL**.
3. Дополнительно можно настроить команду `/setcommands` (например, `shop — Открыть каталог`)
   и кнопку `web_app` в сообщениях бота.
4. Откройте бота → кнопка меню снизу запускает приложение внутри Telegram.

Что используется из Telegram WebApp API (`src/lib/telegram.ts`, `src/components/CatalogApp.tsx`):

- подключён официальный скрипт `https://telegram.org/js/telegram-web-app.js`;
- `ready()`, `expand()`, `setHeaderColor('#0a0a0f')`, `setBackgroundColor('#0a0a0f')`;
- `viewportStableHeight` → CSS-переменная `--tg-height`;
- `openTelegramLink()` для оформления заявки;
- `initData` / `themeParams` доступны через `window.Telegram.WebApp`, если понадобится авторизация
  или адаптация темы (сейчас тема фиксированная неоновая).

Без Telegram (обычный браузер) приложение полностью работоспособно: API проверяется на наличие,
ошибок в консоли быть не должно.

## Структура проекта

```
src/
  app/            layout.tsx (метаданные, Inter, Telegram-скрипт), page.tsx, globals.css, components.css
  components/     Header, TabBar, ProductCard, ChannelCard, SearchBar, FilterBar, SortSelect,
                  LoadMore, ProductModal, ModalShell, CartDrawer, FavoritesPanel, ToastViewport,
                  EmptyState, Highlight, Icon, AppProvider, CatalogApp, ProductsSection, ChannelsSection
  data/           products.ts, channels.ts          ← демо-данные для замены
  hooks/          useDebouncedValue, usePersistentState
  lib/            types, catalog (поиск/фильтры/сортировка), telegram, asset, format
  types/          telegram.d.ts
public/
  assets/products/*.svg, assets/channels/*.svg, assets/icons/*
  manifest.json, robots.txt, sitemap.xml, .nojekyll
scripts/          generate-assets.mjs, generate-seo.mjs, generate-icons.py, preview.mjs, verify-build.mjs
.github/workflows/deploy.yml
```

## Ограничения

- Это фронтенд без backend: нет реальных платежей, складских остатков и личного кабинета.
  Заявка уходит как текст через штатный диалог «Поделиться» в Telegram.
- Избранное и корзина хранятся в `localStorage` текущего устройства/браузера.
- Демо-ссылки каналов вымышленные — замените на реальные.
- PNG-иконки (`icon-192.png`, `icon-512.png`, `apple-touch-icon.png`) уже лежат в `public/assets/icons/`.
  При смене дизайна их можно пересоздать вручную: `python3 scripts/generate-icons.py` (нужен Pillow).
  В обычной сборке Python не требуется.
- Шрифт Inter подключается через `next/font` на этапе сборки, внешних запросов в рантайме нет.
  Единственный внешний скрипт — официальный `telegram-web-app.js`.

## Лицензия

MIT — см. `LICENSE`.
