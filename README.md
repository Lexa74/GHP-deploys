# GHP-deploys

Гайд по развёртыванию фронтенд-проекта на **GitHub Pages** для двух самых популярных сборщиков:

- [Vite](#развёртывание-vite-проекта)
- [Create React App (CRA)](#развёртывание-cra-проекта)

Для каждого варианта описаны два способа:

1. **GitHub Actions** — рекомендуемый способ, деплой запускается автоматически при пуше в ветку.
2. **Пакет `gh-pages`** — деплой руками (или по команде) прямо с локальной машины.

Готовые примеры workflow-файлов лежат в [`.github/workflows`](.github/workflows).

---

## Развёртывание Vite-проекта

### 1. Настройка `base` в конфиге

GitHub Pages для проекта отдаётся по адресу вида `https://<username>.github.io/<repo-name>/`, поэтому в `vite.config.js` (или `.ts`) нужно указать `base`, иначе пути к ассетам сломаются:

```js
// vite.config.js
import { defineConfig } from 'vite'

export default defineConfig({
  base: '/<repo-name>/', // например '/GHP-deploys/'
})
```

Если сайт будет опубликован на **user/organization pages** (репозиторий вида `<username>.github.io`), `base` указывать не нужно — оставьте `'/'` (значение по умолчанию).

### 2а. Способ через GitHub Actions (рекомендуется)

1. В настройках репозитория: **Settings → Pages → Build and deployment → Source** выберите **GitHub Actions**.
2. Добавьте workflow-файл `.github/workflows/deploy-vite.yml` (готовый пример — [`deploy-vite.yml`](.github/workflows/deploy-vite.yml)):

```yaml
name: Deploy Vite to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

3. Закоммитьте и запушьте в `main` — деплой запустится автоматически.

### 2б. Способ через пакет `gh-pages`

```bash
npm install -D gh-pages
```

В `package.json` добавьте скрипт:

```json
{
  "scripts": {
    "build": "vite build",
    "deploy": "npm run build && gh-pages -d dist"
  }
}
```

Запуск деплоя:

```bash
npm run deploy
```

Затем в **Settings → Pages → Source** выберите ветку `gh-pages` (создаётся автоматически пакетом `gh-pages`).

---

## Развёртывание CRA-проекта

### 1. Настройка `homepage` в `package.json`

```json
{
  "homepage": "https://<username>.github.io/<repo-name>"
}
```

Для user/organization pages (`<username>.github.io`) укажите просто `"homepage": "https://<username>.github.io"`.

### 2а. Способ через GitHub Actions (рекомендуется)

1. В настройках репозитория: **Settings → Pages → Build and deployment → Source** выберите **GitHub Actions**.
2. Добавьте workflow-файл `.github/workflows/deploy-cra.yml` (готовый пример — [`deploy-cra.yml`](.github/workflows/deploy-cra.yml)):

```yaml
name: Deploy CRA to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: build

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

3. Закоммитьте и запушьте в `main` — деплой запустится автоматически.

### 2б. Способ через пакет `gh-pages`

```bash
npm install -D gh-pages
```

В `package.json` добавьте скрипты:

```json
{
  "scripts": {
    "predeploy": "npm run build",
    "deploy": "gh-pages -d build"
  }
}
```

Запуск деплоя:

```bash
npm run deploy
```

Затем в **Settings → Pages → Source** выберите ветку `gh-pages`.

---

## Частые проблемы

- **Белая страница / 404 на ассетах** — почти всегда неверный `base` (Vite) или `homepage` (CRA): проверьте, что путь совпадает с именем репозитория.
- **Роутинг (React Router) даёт 404 при обновлении страницы** — GitHub Pages не умеет в SPA-роутинг из коробки. Варианты: `HashRouter` вместо `BrowserRouter`, либо трюк с копированием `index.html` в `404.html` после сборки.
- **Изменения не подхватываются** — Pages кэширует контент, подождите пару минут или сделайте hard refresh (Ctrl/Cmd+Shift+R).
