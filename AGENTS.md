# Repository Guidelines

## Project Structure & Module Organization
- `src/` houses the Vue 3 application: `App.vue` controls quiz flow, `main.js` mounts the app, and `style.css` defines global styling.
- `index.html` is the Vite entry document; `vite.config.js` configures the build with `@vitejs/plugin-vue`.
- `dist/` and `node_modules/` are generated directories (ignored by git); rerun builds to regenerate artifacts.
- Add new composables or utilities under `src/` using subfolders (`src/composables/`, `src/utils/`) if the codebase grows.

## Build, Test, and Development Commands
- `npm install` installs dependencies; required once per environment or after `package.json` changes.
- `npm run dev` launches the Vite dev server with hot module replacement at the printed localhost URL.
- `npm run build` runs the production build and outputs to `dist/`.
- `npm run preview` serves the production bundle locally for smoke testing.

## Coding Style & Naming Conventions
- Use Vue 3 Composition API in `.vue` files; keep script sections in `<script setup>` form when possible.
- Prefer PascalCase for Vue components (`QuizTimer.vue`), camelCase for JavaScript variables, and kebab-case for CSS classes.
- Indentation: two spaces across JS/TS, Vue, and CSS files. Avoid trailing whitespace.

## Testing Guidelines
- No automated tests are configured yet; integrate Vitest or Cypress if testing is required.
- Name new test files `*.spec.ts` or `*.spec.js` under `src/` or a dedicated `tests/` directory and document the chosen pattern in this guide when added.

## Commit & Pull Request Guidelines
- Follow concise, descriptive commit messages in Japanese (example: `カテゴリ別クイズ機能を実装し依存関係を更新`).
- For pull requests, include: summary of changes, testing evidence (`npm run build` logs or screenshots), and linked issues when available.
- Keep PRs focused on a single feature or fix; split large efforts into smaller, reviewable pieces.

## Security & Configuration Tips
- Vite 7 requires Node.js 18+; ensure local environments match.
- Local storage captures quiz history; avoid storing sensitive data there.
- Run `npm audit` after dependency updates and apply fixes (`npm audit fix` or `--force` when vetted) to keep build tooling secure.
