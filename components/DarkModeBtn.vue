<template>
  <div class="flex">
    <button class="btn btn-circle" @click="switchTheme">
      <component
        :is="iconName"
        :class="[
          'w-10 h-10',
          {
            'text-white': $colorMode.value === 'dark',
          },
        ]"
      />
    </button>
  </div>
</template>

<script setup>
import { useContext, onMounted, computed } from '@nuxtjs/composition-api'

const ctx = useContext()
const iconName = computed(() => {
  return ctx.$colorMode.value === 'dark'
    ? 'solid-moon-icon'
    : 'outline-light-bulb-icon'
})

if (ctx.$colorMode.value === 'dark') {
  iconName.value = 'solid-moon-icon'
}

onMounted(() => {
  // Daisy
  document.documentElement.setAttribute('data-theme', ctx.$colorMode.value)
})

const switchTheme = () => {
  const color = ctx.$colorMode.value === 'dark' ? 'light' : 'dark'
  ctx.$colorMode.value = color
  // Daisy
  document.documentElement.setAttribute('data-theme', color)
}
</script>
