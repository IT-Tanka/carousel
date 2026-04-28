<script setup lang="ts">
import { onMounted, ref } from 'vue'
import Carousel from './components/Carousel.vue'
import SelectedList from './components/SelectedList.vue'
import type { PicsumImage } from './types'

const images = ref<PicsumImage[]>([])
const selectedUrls = ref<string[]>([])
const isLoading = ref<boolean>(false)
const isError = ref<boolean>(false)
const errorMessage = ref<string>('')

const fetchImages = async () => {
  isLoading.value = true
  isError.value = false
  errorMessage.value = ''

  try {
    const response = await fetch('https://picsum.photos/v2/list?page=1&limit=30')
    if (!response.ok) {
      throw new Error(`Request failed (${response.status})`)
    }
    images.value = await response.json()
  } catch (err) {
    isError.value = true
    images.value = []
    selectedUrls.value = []
    errorMessage.value = err instanceof Error ? err.message : 'Unknown error'
  } finally {
    isLoading.value = false
  }
}

onMounted(fetchImages)
</script>

<template>
  <div class="container">
    <div v-if="isLoading" class="status card">Loading images…</div>

    <div v-else-if="isError" class="status card status--error">
      <div class="status__title">Couldn’t load images</div>
      <div class="status__message">{{ errorMessage }}</div>
      <button class="btn" type="button" @click="fetchImages">Retry</button>
    </div>

    <template v-else>
      <Carousel
        :images="images"
        :selected-urls="selectedUrls"
        @update:selected-urls="selectedUrls = $event"
      />
      <SelectedList :selected-urls="selectedUrls" />
    </template>
  </div>
</template>

<style scoped lang="scss">
.status {
  padding: 16px 18px;
}

.status--error {
  border-color: rgba(251, 113, 133, 0.5);
}

.status__title {
  font-weight: 700;
  margin-bottom: 6px;
}

.status__message {
  color: rgba(255, 255, 255, 0.72);
  font-size: 13px;
  margin-bottom: 12px;
}
</style>
