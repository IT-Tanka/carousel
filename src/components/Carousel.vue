<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref, watch } from 'vue'
import type { PicsumImage } from '../types'

const props = defineProps<{
  images: PicsumImage[]
  selectedUrls: string[]
}>()

const emit = defineEmits<{
  (e: 'update:selectedUrls', value: string[]): void
}>()

const currentIndex = ref<number>(0)
const visibleCount = ref<number>(4)
const shift = ref<number>(1) // 0=prev buffer, 1=active window start, 2=next buffer
const isAnimating = ref<boolean>(false)
const pendingDirection = ref<'prev' | 'next' | null>(null)

const updateVisibleCount = () => {
  visibleCount.value =
    window.innerWidth < 768 ? 1 : Math.max(1, Math.floor(window.innerWidth / 220))
}

const toggleSelect = (image: PicsumImage) => {
  const url = image.download_url
  const idx = props.selectedUrls.indexOf(url)
  if (idx === -1) {
    emit('update:selectedUrls', [...props.selectedUrls, url])
  } else {
    const next = props.selectedUrls.slice()
    next.splice(idx, 1)
    emit('update:selectedUrls', next)
  }
}

const isSelected = (image: PicsumImage): boolean => {
  return props.selectedUrls.includes(image.download_url)
}

watch(
  () => props.images.length,
  (len) => {
    if (len === 0) {
      currentIndex.value = 0
      isAnimating.value = false
      pendingDirection.value = null
      shift.value = 1
      return
    }
    currentIndex.value = ((currentIndex.value % len) + len) % len
    isAnimating.value = false
    pendingDirection.value = null
    shift.value = 1
  },
)

onMounted(() => {
  updateVisibleCount()
  window.addEventListener('resize', updateVisibleCount)
})

onUnmounted(() => {
  window.removeEventListener('resize', updateVisibleCount)
})

const prev = () => {
  const len = props.images.length
  if (len === 0 || isAnimating.value) return
  isAnimating.value = true
  pendingDirection.value = 'prev'
  shift.value = 0
}

const next = () => {
  const len = props.images.length
  if (len === 0 || isAnimating.value) return
  isAnimating.value = true
  pendingDirection.value = 'next'
  shift.value = 2
}

const onCarouselKeydown = (e: KeyboardEvent) => {
  if (e.key === 'ArrowLeft') {
    e.preventDefault()
    prev()
  }
  if (e.key === 'ArrowRight') {
    e.preventDefault()
    next()
  }
}

const safeVisibleCount = computed(() => {
  const len = props.images.length
  if (len === 0) return 1
  return Math.min(visibleCount.value, len)
})

const trackImages = computed<PicsumImage[]>(() => {
  const len = props.images.length
  if (len === 0) return []
  const count = safeVisibleCount.value
  const start = (currentIndex.value - 1 + len) % len
  return Array.from({ length: count + 2 }, (_, i) => props.images[(start + i) % len]!)
})

const getImageUrl = (image: PicsumImage, width = 400, height = 300): string => {
  return `https://picsum.photos/id/${image.id}/${width}/${height}`
}

const failedImageIds = ref<Set<string>>(new Set())

const FALLBACK_W = 400
const FALLBACK_H = 220

const getFallbackDataUrl = (image: PicsumImage): string => {
  const line2 = `by ${image.author}`
  const line3 = `(id: ${image.id})`
  const esc = (s: string) =>
    s.replaceAll('&', '&amp;').replaceAll('<', '&lt;').replaceAll('>', '&gt;').replaceAll('"', '&quot;')

  const svg = `<?xml version="1.0" encoding="UTF-8"?>\n` +
    `<svg xmlns="http://www.w3.org/2000/svg" width="${FALLBACK_W}" height="${FALLBACK_H}" viewBox="0 0 ${FALLBACK_W} ${FALLBACK_H}" preserveAspectRatio="xMidYMid meet">\n` +
    `<g font-family="ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Arial">\n` +
    `<text x="24" y="92" fill="rgba(255,255,255,0.90)" font-size="18" font-weight="700">Couldn’t load image</text>\n` +
    `<text x="24" y="122" fill="rgba(255,255,255,0.78)" font-size="14">\n` +
    `<tspan x="24" dy="0">${esc(line2)}</tspan>\n` +
    `<tspan x="24" dy="20">${esc(line3)}</tspan>\n` +
    `</text>\n` +
    `</g>\n` +
    `</svg>`
  return `data:image/svg+xml;charset=utf-8,${encodeURIComponent(svg)}`
}

const getDisplaySrc = (image: PicsumImage): string => {
  return failedImageIds.value.has(image.id) ? getFallbackDataUrl(image) : getImageUrl(image)
}

const isFallback = (image: PicsumImage): boolean => {
  return failedImageIds.value.has(image.id)
}

const onImageError = (image: PicsumImage) => {
  if (failedImageIds.value.has(image.id)) return
  const next = new Set(failedImageIds.value)
  next.add(image.id)
  failedImageIds.value = next
}

const onTrackTransitionEnd = (e: TransitionEvent) => {
  if (e.propertyName !== 'transform') return
  if (!isAnimating.value || !pendingDirection.value) return

  const len = props.images.length
  if (len === 0) {
    isAnimating.value = false
    pendingDirection.value = null
    shift.value = 1
    return
  }

  if (pendingDirection.value === 'next') {
    currentIndex.value = (currentIndex.value + 1) % len
  } else {
    currentIndex.value = (currentIndex.value - 1 + len) % len
  }

  // Snap back to the centered window without animation.
  isAnimating.value = false
  pendingDirection.value = null
  shift.value = 1
}
</script>

<template>
  <section class="carousel card" tabindex="0" @keydown="onCarouselKeydown">
    <header class="carousel__header">
      <div class="carousel__title">Image carousel</div>
      <div class="carousel__controls">
        <button class="btn" type="button" :disabled="props.images.length === 0" @click="prev">
          Prev
        </button>
        <button class="btn" type="button" :disabled="props.images.length === 0" @click="next">
          Next
        </button>
      </div>
    </header>

    <div class="carousel__viewport">
      <div
        class="carousel__track"
        :class="{ 'carousel__track--animating': isAnimating }"
        :style="{
          '--visible': String(safeVisibleCount),
          '--shift': String(shift),
        }"
        @transitionend="onTrackTransitionEnd"
      >
        <article
          v-for="image in trackImages"
          :key="`${image.id}-${image.download_url}`"
          class="slide"
          :class="{ selected: isSelected(image) }"
          role="button"
          tabindex="0"
          @click="toggleSelect(image)"
          @keydown.enter.prevent="toggleSelect(image)"
          @keydown.space.prevent="toggleSelect(image)"
        >
          <img
            class="slide__img"
            :class="{ 'slide__img--fallback': isFallback(image) }"
            :src="getDisplaySrc(image)"
            :alt="`Photo by ${image.author}`"
            loading="lazy"
            decoding="async"
            @error="onImageError(image)"
          />
          <div class="slide__meta">
            <div class="slide__author">{{ image.author }}</div>
            <div class="slide__id">id: {{ image.id }}</div>
          </div>
        </article>
      </div>
    </div>
  </section>
</template>

<style scoped lang="scss">
.carousel {
  padding: 18px;
}

.carousel__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding-bottom: 14px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.12);
  margin-bottom: 14px;
}

.carousel__title {
  font-weight: 700;
  letter-spacing: 0.2px;
}

.carousel__controls {
  display: flex;
  gap: 10px;
}

.carousel__viewport {
  overflow: hidden;
  padding: 8px;
}

.carousel__track {
  --gap: 12px;
  display: flex;
  gap: var(--gap);
  // Slide width = (100% - gap*(visible-1))/visible
  // Step = slide width + gap
  transform: translateX(
    calc(
      var(--shift) * -1 *
        ( (100% - (var(--gap) * (var(--visible) - 1))) / var(--visible) + var(--gap) )
    )
  );
}

.carousel__track--animating {
  transition: transform 260ms ease;
}

.slide {
  position: relative;
  flex: 0 0 calc((100% - (var(--gap) * (var(--visible) - 1))) / var(--visible));
  border-radius: 14px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: rgba(255, 255, 255, 0.04);
  cursor: pointer;
  outline: none;
  transition: transform 120ms ease, border-color 120ms ease, box-shadow 120ms ease;
}

.slide:hover {
  transform: translateY(-1px);
  border-color: rgba(255, 255, 255, 0.22);
}

.slide:focus-visible {
  box-shadow: 0 0 0 3px rgba(120, 178, 255, 0.55);
}

.slide.selected {
  border-color: rgba(94, 234, 212, 0.75);
  box-shadow: 0 0 0 2px rgba(94, 234, 212, 0.35);
}

.slide.selected::after {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(94, 234, 212, 0.12);
  pointer-events: none;
}

.slide__img {
  display: block;
  width: 100%;
  height: 220px;
  object-fit: cover;
  background: rgba(255, 255, 255, 0.03);
}

.slide__img--fallback {
  object-fit: contain;
  background: rgba(255, 255, 255, 0.04);
}

.slide__meta {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  padding: 10px 10px 11px;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.75);
}

.slide__author {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.slide__id {
  flex: 0 0 auto;
  color: rgba(255, 255, 255, 0.65);
}
</style>

