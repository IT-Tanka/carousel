<script setup lang="ts">
const props = defineProps<{
  selectedUrls: string[]
}>()
</script>

<template>
  <section class="selected card">
    <header class="selected__header">
      <div class="selected__title">Selected</div>
      <div class="selected__count">{{ props.selectedUrls.length }}</div>
    </header>

    <div v-if="props.selectedUrls.length === 0" class="selected__empty">
      No images selected yet.
    </div>

    <TransitionGroup v-else name="selected" tag="ul" class="selected__list">
      <li v-for="url in props.selectedUrls" :key="url" class="selected__item">
        <code class="selected__url">{{ url }}</code>
      </li>
    </TransitionGroup>
  </section>
</template>

<style scoped lang="scss">
.selected {
  margin-top: 14px;
  padding: 14px 18px 16px;
}

.selected__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding-bottom: 10px;
  margin-bottom: 10px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.12);
}

.selected__title {
  font-weight: 700;
}

.selected__count {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.72);
  padding: 3px 8px;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: rgba(255, 255, 255, 0.06);
}

.selected__empty {
  color: rgba(255, 255, 255, 0.72);
  font-size: 13px;
  padding: 6px 0 2px;
}

.selected__list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  gap: 8px;
}

.selected__item {
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: rgba(255, 255, 255, 0.04);
}

.selected__url {
  display: block;
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono', monospace;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.86);
  word-break: break-all;
}

.selected-enter-active,
.selected-leave-active {
  transition: opacity 160ms ease, transform 160ms ease;
}

.selected-enter-from,
.selected-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}

.selected-move {
  transition: transform 160ms ease;
}
</style>

