<script setup lang="ts">
import { ref } from 'vue'
import ColorPicker from './components/ColorPicker.vue'
import ColorPickerInput from './components/ColorPickerInput.vue'

const selectedColor = ref('#FF0000')
const customColor = ref('#FF5733')
const showPicker = ref(false)
const pickerPosition = ref({ top: '0px', left: '0px' })

// 点击 input 显示颜色选择器
function showColorPicker(event: Event) {
  const target = event.target as HTMLElement
  const rect = target.getBoundingClientRect()
  const viewportWidth = window.innerWidth
  const viewportHeight = window.innerHeight

  const panelWidth = 280
  const panelHeight = 300
  const offset = 4

  let left = rect.left
  if (left + panelWidth > viewportWidth) {
    left = rect.left - panelWidth + rect.width
  }
  if (left < 0) left = 0

  let top = rect.bottom + offset
  if (top + panelHeight > viewportHeight) {
    top = rect.top - panelHeight - offset
  }
  if (top < 0) top = 0

  pickerPosition.value = {
    top: `${top}px`,
    left: `${left}px`,
  }
  showPicker.value = true
}
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

    <div class="demo-section">
      <h2>自定义 Input 绑定</h2>
      <p class="tip">点击 input 即可呼出颜色选择器，选择颜色自动同步</p>
      <div class="custom-input-wrapper">
        <input
          ref="customInputRef"
          v-model="customColor"
          readonly
          @click="showColorPicker"
          class="custom-input"
          placeholder="点击选择颜色"
        />
        <div
          v-if="showPicker"
          class="custom-picker-container"
          :style="pickerPosition"
        >
          <ColorPicker
            v-model="customColor"
            :close-on-outside-click="true"
            @close="showPicker = false"
            @mousedown.stop
          />
        </div>
      </div>
      <p class="color-display">选中的颜色: {{ customColor }}</p>
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

.tip {
  color: #6e7681;
  margin: 0 0 10px 0;
  font-size: 13px;
  text-align: center;
}

.custom-input-wrapper {
  position: relative;
  display: inline-block;
}

.custom-input {
  padding: 10px 16px;
  border: 1px solid #364347;
  border-radius: 4px;
  background: #15191c;
  color: #8b949e;
  font-size: 14px;
  cursor: pointer;
  min-width: 200px;
  text-align: center;
  transition: border-color 0.2s, background 0.2s;
}

.custom-input:hover {
  border-color: #58a6ff;
  background: #1a1f24;
}

.custom-input:focus {
  outline: none;
  border-color: #58a6ff;
}

.custom-picker-container {
  position: fixed;
  z-index: 9999;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
  animation: fadeIn 0.2s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
