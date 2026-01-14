# Vue Color Picker

一个功能强大、美观的 Vue 3 颜色选择器组件，支持多种颜色格式和透明度调节。

## ✨ 特性

- 🎨 **多种颜色格式支持**: HEX、HEX8、RGB、RGBA、HSL、HSV
- 🔍 **实时预览**: 色谱图、色相条、透明度条实时预览
- 📋 **一键复制**: 点击颜色值即可复制到剪贴板
- 🔄 **格式切换**: 支持在多种格式间切换
- 🎯 **透明度支持**: 自动切换支持透明度的格式
- 🖱️ **拖拽交互**: 直观的拖拽操作调整颜色
- 📱 **响应式设计**: 适配不同屏幕尺寸
- 💎 **类型安全**: 完整的 TypeScript 支持

## 🚀 快速开始

### 安装依赖

```bash
npm install
```

### 开发模式

```bash
npm run dev
```

### 类型检查

```bash
npm run type-check
```

### 代码格式化

```bash
npm run format
```

### 代码检查

```bash
npm run lint
```

### 构建生产版本

```bash
npm run build
```

## 📖 使用示例

### 1. 基础使用

最简单的使用方式，使用 v-model 双向绑定颜色值：

```vue
<template>
  <div>
    <ColorPicker v-model="color" />
    <p>当前颜色: {{ color }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const color = ref('#FF5733')
</script>
```

### 2. 限制颜色格式

只显示指定的颜色格式，例如只支持 HEX 和 RGB：

```vue
<template>
  <ColorPicker
    v-model="color"
    :formats="['hex', 'rgb']"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const color = ref('#FF5733')
</script>
```

**可用格式**:
- `hex`: 不透明颜色，如 `#FF5733`
- `hex8`: 带透明度的颜色，如 `#FF573380`
- `rgb`: RGB 格式，如 `rgb(255, 87, 51)`
- `rgba`: RGBA 格式，如 `rgba(255, 87, 51, 0.5)`
- `hsl`: HSL 格式，如 `hsl(12, 100%, 60%)`
- `hsv`: HSV 格式（支持透明度），如 `hsv(12, 100%, 100%)`

### 3. 与 Input 组件集成

点击 Input 输入框弹出颜色选择器：

```vue
<template>
  <div class="color-input-wrapper">
    <input
      v-model="color"
      readonly
      @click="showPicker = !showPicker"
      placeholder="点击选择颜色"
    />
    <ColorPicker
      v-if="showPicker"
      v-model="color"
      :close-on-outside-click="true"
      @close="showPicker = false"
      class="color-picker-popup"
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const color = ref('#FF5733')
const showPicker = ref(false)
</script>

<style scoped>
.color-input-wrapper {
  position: relative;
  display: inline-block;
}

.color-picker-popup {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  z-index: 100;
}

input {
  padding: 10px 16px;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  min-width: 200px;
}
</style>
```

### 4. 智能定位的颜色选择器

自动计算位置，避免超出视口：

```vue
<template>
  <div>
    <input
      v-model="color"
      readonly
      @click="handleShowPicker"
      placeholder="点击选择颜色"
    />
    <div v-if="showPicker" class="picker-container" :style="pickerPosition">
      <ColorPicker
        v-model="color"
        :close-on-outside-click="true"
        @close="showPicker = false"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const color = ref('#FF5733')
const showPicker = ref(false)
const pickerPosition = ref({ top: '0px', left: '0px' })

function handleShowPicker(event: MouseEvent) {
  const target = event.target as HTMLElement
  const rect = target.getBoundingClientRect()
  const viewportWidth = window.innerWidth
  const viewportHeight = window.innerHeight

  const panelWidth = 280
  const panelHeight = 300
  const offset = 4

  // 计算水平位置
  let left = rect.left
  if (left + panelWidth > viewportWidth) {
    left = rect.left - panelWidth + rect.width
  }
  if (left < 0) left = 0

  // 计算垂直位置
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

<style scoped>
.picker-container {
  position: fixed;
  z-index: 9999;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
}

input {
  padding: 10px 16px;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  min-width: 200px;
}
</style>
```

### 5. 支持透明度的颜色选择

只显示支持透明度的格式：

```vue
<template>
  <ColorPicker
    v-model="color"
    :formats="['hex8', 'rgba', 'hsv']"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const color = ref('#FF573380')
</script>
```

### 6. 自定义样式覆盖

覆盖组件默认样式：

```vue
<template>
  <div class="custom-color-picker">
    <ColorPicker v-model="color" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const color = ref('#FF5733')
</script>

<style>
/* 修改面板背景色 */
.custom-color-picker :deep(.color-picker-panel) {
  background: #ffffff;
  border-color: #e0e0e0;
}

/* 修改颜色值文本颜色 */
.custom-color-picker :deep(.color-value) {
  color: #333;
  background: #f5f5f5;
  border-color: #e0e0e0;
}
</style>
```

### 7. 在表单中使用

结合表单验证使用：

```vue
<template>
  <form @submit.prevent="handleSubmit">
    <div class="form-group">
      <label>主题颜色</label>
      <div class="color-input-wrapper">
        <input
          v-model="formData.themeColor"
          readonly
          @click="showPicker = !showPicker"
          class="color-input"
        />
        <ColorPicker
          v-if="showPicker"
          v-model="formData.themeColor"
          :close-on-outside-click="true"
          @close="showPicker = false"
          class="picker-popup"
        />
      </div>
      <span v-if="errors.themeColor" class="error">{{ errors.themeColor }}</span>
    </div>
    <button type="submit">提交</button>
  </form>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const formData = reactive({
  themeColor: '#FF5733'
})

const errors = reactive({
  themeColor: ''
})

const showPicker = ref(false)

function validateForm() {
  errors.themeColor = ''
  if (!formData.themeColor) {
    errors.themeColor = '请选择主题颜色'
    return false
  }
  return true
}

function handleSubmit() {
  if (validateForm()) {
    console.log('表单提交:', formData)
    // 提交逻辑...
  }
}
</script>

<style scoped>
.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
}

.color-input-wrapper {
  position: relative;
  display: inline-block;
}

.picker-popup {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  z-index: 100;
}

.color-input {
  padding: 10px 16px;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  min-width: 200px;
}

.error {
  display: block;
  color: #ff4d4f;
  font-size: 12px;
  margin-top: 4px;
}

button {
  padding: 10px 20px;
  background: #1890ff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background: #40a9ff;
}
</style>
```

### 8. 多个颜色选择器

在同一页面使用多个颜色选择器：

```vue
<template>
  <div>
    <div class="color-item">
      <label>主色调</label>
      <ColorPicker v-model="colors.primary" />
    </div>
    <div class="color-item">
      <label>次要颜色</label>
      <ColorPicker v-model="colors.secondary" />
    </div>
    <div class="color-item">
      <label>强调色</label>
      <ColorPicker v-model="colors.accent" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const colors = reactive({
  primary: '#FF5733',
  secondary: '#33FF57',
  accent: '#3357FF'
})
</script>

<style scoped>
.color-item {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
}
</style>
```

## Props

| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `modelValue` | `string` | `undefined` | 当前颜色值，支持所有格式 |
| `formats` | `FormatType[]` | `undefined` | 可选的颜色格式列表，限制显示的格式 |
| `closeOnOutsideClick` | `boolean` | `false` | 是否在点击外部时触发 `close` 事件 |

### FormatType

```typescript
type FormatType = 'hex' | 'hex8' | 'rgb' | 'rgba' | 'hsl' | 'hsv'
```

**格式说明**:
- `hex`: 6位十六进制颜色，如 `#FF5733`（不支持透明度）
- `hex8`: 8位十六进制颜色，如 `#FF573380`（支持透明度，最后两位为透明度）
- `rgb`: RGB 格式，如 `rgb(255, 87, 51)`（不支持透明度）
- `rgba`: RGBA 格式，如 `rgba(255, 87, 51, 0.5)`（支持透明度）
- `hsl`: HSL 格式，如 `hsl(12, 100%, 60%)`（不支持透明度）
- `hsv`: HSV 格式，如 `hsv(12, 100%, 100%)`（支持透明度）

## Events

| 事件 | 参数 | 描述 |
|------|------|------|
| `update:modelValue` | `value: string` | 颜色值变化时触发，返回当前格式的颜色值 |
| `close` | `void` | 当 `closeOnOutsideClick` 为 true 且点击外部时触发 |

## 交互说明

### 格式切换
- 点击颜色值区域可以切换颜色格式和复制颜色值
- 切换顺序按照 `formats` 属性指定的顺序循环
- 如果未指定 `formats`，默认按照 `hex -> hex8 -> rgb -> rgba -> hsl -> hsv` 的顺序切换

### 透明度切换
- 当从不透明颜色（alpha = 1）调整为有透明度（alpha < 1）时，格式会自动切换到支持透明度的格式：
  - `hex` → `hex8`
  - `rgb` → `rgba`
  - `hsl` → `hsv`
- 当从有透明度调整为不透明时，格式会自动切换回对应的不带透明度的格式：
  - `hex8` → `hex`
  - `rgba` → `rgb`
  - `hsv` → `hsl`

### 颜色值保持
- 如果用户手动切换过格式，调整颜色时会保持用户选择的格式
- 如果用户通过 `v-model` 传入新格式的颜色值，会自动识别并切换到该格式

## 🎯 技术栈

- **Vue 3**: 渐进式 JavaScript 框架
- **TypeScript**: JavaScript 的超集，提供类型安全
- **Vite**: 下一代前端构建工具
- **Canvas API**: 用于绘制色谱图、色相条和透明度条

## 📄 许可证

MIT License

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📧 联系方式

作者: Sogrey
GitHub: [Sogrey](https://github.com/Sogrey)
