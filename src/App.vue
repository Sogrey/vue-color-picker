<script setup lang="ts">
import { ref } from 'vue'
import ColorPicker from './components/ColorPicker.vue'
import ColorPickerInput from './components/ColorPickerInput.vue'

const selectedColor = ref('#FF0000')
</script>

<template>
  <div class="app">
    <h1>Vue Color Picker</h1>

    <div class="demo-section">
      <h2>独立使用</h2>
      <ColorPicker />
    </div>

    <div class="demo-section">
      <h2>Input 组件使用</h2>
      <ColorPickerInput v-model="selectedColor" />
      <p class="color-display">选中的颜色: {{ selectedColor }}</p>
    </div>
  </div>
</template>

<style>
body {
  margin: 0;
  padding: 0;
  background: #15191c;
  min-height: 100vh;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
}

#app {
  width: 100%;
}

.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 40px 20px;
  gap: 40px;
}

h1 {
  color: #8b949e;
  margin-bottom: 10px;
  font-weight: 400;
  font-size: 24px;
  letter-spacing: 0.1em;
}

.demo-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.demo-section h2 {
  color: #8b949e;
  margin: 0;
  font-weight: 400;
  font-size: 18px;
  letter-spacing: 0.05em;
}

.color-display {
  color: #8b949e;
  margin: 0;
  font-size: 14px;
}
</style>
