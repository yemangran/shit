# Repository Guidelines

## Project Structure & Module Organization
This repository is a Slidev deck for the thesis presentation in `slides.md`. Reusable interactive slides live in `components/` as Vue 3 single-file components such as `ImageScanCompare.vue`. Static assets belong in `public/`, with presentation images currently under `public/imgs/`. Reference content and experiments can be kept in `snippets/` and `pages/`. Build output is generated into `dist/` and should not be edited manually.

## Build, Test, and Development Commands
Install dependencies with `pnpm install`.

- `pnpm dev`: starts the Slidev dev server, usually at `http://localhost:3030`, with live reload.
- `pnpm build`: creates the production deck in `dist/`.
- `pnpm export`: exports the slides to PDF or other supported formats through Slidev.

Run `pnpm build` before submitting changes that affect slides, assets, or components.

## Coding Style & Naming Conventions
Use Vue 3 with `<script setup lang="ts">` for interactive components. Name components in PascalCase, for example `ThresholdStrategyDemo.vue`; use descriptive prop names in camelCase. Follow the existing formatting style in the file you touch; most Vue code here uses double quotes and a simple, readable layout. Keep slide content concise, prefer Chinese copy that matches the presentation language, and use clear asset paths such as `/public/imgs/page2a.png`.

## Testing Guidelines
There is no automated test suite configured in `package.json`. Validation is therefore manual:

- run `pnpm dev` and step through affected slides;
- confirm assets load and animations render correctly;
- run `pnpm build` to catch broken imports or Slidev syntax issues.

If you add logic-heavy Vue components, include a short manual test note in the pull request.

## Commit & Pull Request Guidelines
Recent commits use short, imperative messages such as `update resource path` and `Add ImageScanCompare component for interactive denoising demo`. Keep commits focused and use a one-line summary that states the change clearly. For pull requests, include:

- a brief description of the slide or component changes;
- screenshots or exported slide previews for visual updates;
- any asset-path, deployment, or export notes if relevant.

## Deployment & Configuration Notes
`netlify.toml` and `vercel.json` both build from `dist/` using `npm run build`. If you change output behavior, update both files to keep Netlify and Vercel deployment settings aligned.
