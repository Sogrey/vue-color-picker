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

```vue
<template>
  <ColorPicker v-model="color" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const color = ref('#FF5733')
</script>
```

## Props

| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `modelValue` | `string` | `undefined` | 当前颜色值 |
| `formats` | `FormatType[]` | `undefined` | 可选的颜色格式列表 |

### FormatType

```typescript
type FormatType = 'hex' | 'hex8' | 'rgb' | 'rgba' | 'hsl' | 'hsv'
```

## Events

| 事件 | 参数 | 描述 |
|------|------|------|
| `update:modelValue` | `value: string` | 颜色值变化时触发 |

## 🎯 技术栈

- **Vue 3**: 渐进式 JavaScript 框架
- **TypeScript**: JavaScript 的超集，提供类型安全
- **Vite**: 下一代前端构建工具
- **Canvas API**: 用于绘制色谱图、色相条和透明度条

## 🛠️ 开发工具推荐

### IDE

- [VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar)

### 浏览器插件

- Chromium-based 浏览器 (Chrome, Edge, Brave 等):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
- Firefox:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)

## 📄 许可证

MIT License

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📧 联系方式

作者: Sogrey
GitHub: [Sogrey](https://github.com/Sogrey)

