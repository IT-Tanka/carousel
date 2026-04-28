# Image Carousel (Vue 3)

A responsive **Image Carousel** that fetches photos from `picsum.photos`, supports **infinite scrolling**, **selection**, and a **selected URLs** list below the carousel. Built with **Vue 3 + TypeScript + SCSS** and **no 3rd‑party carousel libraries**.

## Features

- **Data fetching**: loads 30 images from `picsum.photos` on mount
- **Loading + error + retry**: full API failures show an error state with a Retry button
- **Infinite carousel**: Prev/Next loop using modulo logic
- **Step navigation**: Prev/Next moves exactly **one image** per click
- **Responsive**
  - **Mobile (< 768px)**: 1 image (full width)
  - **Desktop**: number of visible images depends on viewport (`Math.floor(window.innerWidth / 220)`)
- **Selection**
  - click a slide to select/unselect
  - selected state is highlighted
  - selected image **URLs** are listed below the carousel
- **Per-image error handling**: if a single image fails to load, only that slide shows a fallback (the UI continues working)
- **Animations (extra)**: carousel movement uses `transform` + `transition`

## Tech stack

- Vue 3 (Composition API, `<script setup lang="ts">`)
- TypeScript
- SCSS (no CSS frameworks)
- Vite

## Project structure

```
src/
├── components/
│   ├── Carousel.vue
│   └── SelectedList.vue
├── types/
│   └── index.ts
├── composables/
│   └── useCarousel.ts
├── styles/
│   └── main.scss
├── App.vue
├── main.ts
```

## Getting started

Install dependencies:

```sh
npm install
```

Run dev server:

```sh
npm run dev
```

Type-check:

```sh
npm run type-check
```

Build:

```sh
npm run build
```

