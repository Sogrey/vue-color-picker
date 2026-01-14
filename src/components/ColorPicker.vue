<template>
  <div ref="panelRef" class="color-picker-panel">
    <div class="panel-row">
      <div class="spectrum-map" ref="spectrumMapRef">
        <button class="color-cursor spectrum-cursor" :style="spectrumCursorStyle"></button>
        <canvas ref="spectrumCanvasRef" @mousedown="startSpectrumDrag"></canvas>
      </div>
      <div class="hue-map">
        <button class="color-cursor hue-cursor" :style="hueCursorStyle"></button>
        <canvas ref="hueCanvasRef" @mousedown="startHueDrag"></canvas>
      </div>
      <div v-if="showAlphaBar" class="alpha-map">
        <div class="alpha-checkboard"></div>
        <button class="color-cursor alpha-cursor" :style="alphaCursorStyle"></button>
        <canvas ref="alphaCanvasRef" @mousedown="startAlphaDrag"></canvas>
      </div>
    </div>
    <div class="panel-row value-row">
      <div class="color-preview-container">
        <div class="color-preview-checkboard"></div>
        <div class="color-preview-color" :style="colorPreviewStyle"></div>
      </div>
      <div
        class="color-value"
        @click="
          () => {
            showFormatToggle && toggleFormat()
            copyColorValue()
          }
        "
        :title="copyTitle"
      >
        {{ colorValue }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch, onBeforeUnmount } from 'vue'

// Props
interface Props {
  modelValue?: string
  formats?: FormatType[]
  closeOnOutsideClick?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: undefined,
  formats: undefined,
  closeOnOutsideClick: true,
})

// Emits
const emit = defineEmits<{
  'update:modelValue': [value: string]
  close: []
}>()

// 状态
const hue = ref(0)
const saturation = ref(1)
const lightness = ref(0.5)
const alpha = ref(1)

// 拖拽状态
const isDraggingSpectrum = ref(false)
const isDraggingHue = ref(false)
const isDraggingAlpha = ref(false)

// 复制状态
const copyTitle = ref('点击复制')

// 是否正在设置颜色（用于跳过格式检查）
const isSettingColor = ref(false)

// 用户是否手动切换过格式（用于保持用户选择的格式）
const userHasSwitchedFormat = ref(false)

// 所有格式
const allFormats = ['hex', 'hex8', 'rgb', 'rgba', 'hsl', 'hsv'] as const
type FormatType = (typeof allFormats)[number]

// 带透明度的格式
const alphaFormats = ['hex8', 'rgba', 'hsv'] as const

// 类型保护函数：检查格式是否支持透明度
function isAlphaFormat(format: FormatType): format is (typeof alphaFormats)[number] {
  return alphaFormats.includes(format as 'hex8' | 'rgba' | 'hsv')
}

// 当前可用格式（根据传入的 props 和是否透明动态变化）
const formats = computed(() => {
  // 如果用户指定了格式列表，使用用户指定的
  if (props.formats && props.formats.length > 0) {
    const userFormats = props.formats
    // 如果有透明度，只返回带透明度的格式（在用户指定范围内）
    if (alpha.value < 1) {
      return userFormats.filter(isAlphaFormat)
    }
    return userFormats
  }

  // 默认行为：如果未指定格式，使用所有格式
  // 如果有透明度，只显示带透明度的格式
  if (alpha.value < 1) {
    return alphaFormats
  }
  // 如果不透明，显示所有格式
  return allFormats
})

// 初始化当前格式
const currentFormat = ref<FormatType>('hex')

// 是否显示格式切换按钮（只有当有多种格式可选时才显示）
const showFormatToggle = computed(() => formats.value.length > 1)

// 是否显示透明度条（只有当至少有一种格式支持透明度时才显示）
const showAlphaBar = computed(() => {
  return formats.value.some((f) => alphaFormats.includes(f as 'hex8' | 'rgba' | 'hsv'))
})

// 引用
const spectrumCanvasRef = ref<HTMLCanvasElement | null>(null)
const hueCanvasRef = ref<HTMLCanvasElement | null>(null)
const alphaCanvasRef = ref<HTMLCanvasElement | null>(null)
const spectrumMapRef = ref<HTMLDivElement | null>(null)

// 鼠标位置
const spectrumCursorPos = ref({ x: 0, y: 0 })
const hueCursorPos = ref({ y: 0 })
const alphaCursorPos = ref({ y: 0 })

// 样式计算
const spectrumCursorStyle = computed(() => ({
  left: `${spectrumCursorPos.value.x}px`,
  top: `${spectrumCursorPos.value.y}px`,
  backgroundColor: currentColor.value,
}))

const hueCursorStyle = computed(() => ({
  top: `${hueCursorPos.value.y}px`,
  backgroundColor: `hsl(${hue.value}, 100%, 50%)`,
}))

const alphaCursorStyle = computed(() => ({
  top: `${alphaCursorPos.value.y}px`,
  backgroundColor: `linear-gradient(to bottom, transparent, ${currentColorWithoutAlpha.value})`,
}))

const colorPreviewStyle = computed(() => ({
  backgroundColor: currentColor.value,
}))

// 当前颜色
const currentColor = computed(() => {
  const h = hue.value
  const s = saturation.value
  const l = lightness.value
  const a = alpha.value

  if (a < 1) {
    return `hsla(${h}, ${Math.round(s * 100)}%, ${Math.round(l * 100)}%, ${a})`
  }
  return `hsl(${h}, ${Math.round(s * 100)}%, ${Math.round(l * 100)}%)`
})

const currentColorWithoutAlpha = computed(() => {
  const h = hue.value
  const s = saturation.value
  const l = lightness.value
  return `hsl(${h}, ${Math.round(s * 100)}%, ${Math.round(l * 100)}%)`
})

// 颜色值显示
const colorValue = computed(() => {
  const h = hue.value
  const s = saturation.value
  const l = lightness.value
  const a = alpha.value

  switch (currentFormat.value) {
    case 'hex':
      return rgbToHex(hslToRgb(h, s, l))
    case 'hex8':
      return rgbToHex8(hslToRgb(h, s, l), a)
    case 'rgb':
      const rgb = hslToRgb(h, s, l)
      return `rgb(${rgb.r}, ${rgb.g}, ${rgb.b})`
    case 'rgba':
      const rgb2 = hslToRgb(h, s, l)
      return `rgba(${rgb2.r}, ${rgb2.g}, ${rgb2.b}, ${a.toFixed(2)})`
    case 'hsl':
      return `hsl(${Math.round(h)}, ${Math.round(s * 100)}%, ${Math.round(l * 100)}%)`
    case 'hsv':
      const hsv = hslToHsv(h, s, l)
      return `hsv(${Math.round(hsv.h)}, ${Math.round(hsv.s * 100)}%, ${Math.round(hsv.v * 100)}%)`
    default:
      return '#000000'
  }
})

// HSL 转 RGB
function hslToRgb(h: number, s: number, l: number): { r: number; g: number; b: number } {
  let r: number, g: number, b: number

  if (s === 0) {
    r = g = b = l
  } else {
    const hue2rgb = (p: number, q: number, t: number) => {
      if (t < 0) t += 1
      if (t > 1) t -= 1
      if (t < 1 / 6) return p + (q - p) * 6 * t
      if (t < 1 / 2) return q
      if (t < 2 / 3) return p + (q - p) * (2 / 3 - t) * 6
      return p
    }

    const q = l < 0.5 ? l * (1 + s) : l + s - l * s
    const p = 2 * l - q
    r = hue2rgb(p, q, h / 360 + 1 / 3)
    g = hue2rgb(p, q, h / 360)
    b = hue2rgb(p, q, h / 360 - 1 / 3)
  }

  return {
    r: Math.round(r * 255),
    g: Math.round(g * 255),
    b: Math.round(b * 255),
  }
}

// HSL 转 HSV
function hslToHsv(h: number, s: number, l: number): { h: number; s: number; v: number } {
  const rgb = hslToRgb(h, s, l)
  const r = rgb.r / 255
  const g = rgb.g / 255
  const b = rgb.b / 255

  const max = Math.max(r, g, b)
  const min = Math.min(r, g, b)
  const v = max
  const d = max - min

  const s2 = max === 0 ? 0 : d / max

  return { h, s: s2, v }
}

// RGB 转 Hex
function rgbToHex(rgb: { r: number; g: number; b: number }): string {
  const toHex = (n: number) => {
    const clamped = Math.max(0, Math.min(255, Math.round(n)))
    const hex = clamped.toString(16)
    return hex.length === 1 ? '0' + hex : hex
  }
  return `#${toHex(rgb.r)}${toHex(rgb.g)}${toHex(rgb.b)}`.toUpperCase()
}

// RGB 转 Hex8
function rgbToHex8(rgb: { r: number; g: number; b: number }, a: number): string {
  const hex = rgbToHex(rgb)
  const alphaHex = Math.round(a * 255)
    .toString(16)
    .toUpperCase()
  return `${hex}${alphaHex.length === 1 ? '0' + alphaHex : alphaHex}`
}

// 初始化 Canvas
function initCanvases() {
  // 色谱图
  if (spectrumCanvasRef.value && spectrumMapRef.value) {
    const canvas = spectrumCanvasRef.value
    const rect = spectrumMapRef.value.getBoundingClientRect()
    canvas.width = rect.width
    canvas.height = rect.height
    drawSpectrum()
  }

  // 色相条
  if (hueCanvasRef.value) {
    const canvas = hueCanvasRef.value
    const parent = canvas.parentElement
    if (parent) {
      const rect = parent.getBoundingClientRect()
      canvas.width = rect.width
      canvas.height = rect.height
      drawHue()
      // 初始化色相游标位置
      hueCursorPos.value = { y: 0 }
    }
  }

  // 透明度条
  if (alphaCanvasRef.value) {
    const canvas = alphaCanvasRef.value
    const parent = canvas.parentElement
    if (parent) {
      const rect = parent.getBoundingClientRect()
      canvas.width = rect.width
      canvas.height = rect.height
      drawAlpha()
      // 初始化透明度游标位置（alpha=1，在最顶部）
      alphaCursorPos.value = { y: 0 }
    }
  } else {
    // 如果没有透明度条，将 alpha 固定为 1
    alpha.value = 1
  }

  // 初始化游标位置
  updateCursorPositions()
}

// 绘制色谱图
function drawSpectrum() {
  if (!spectrumCanvasRef.value) return
  const canvas = spectrumCanvasRef.value
  const ctx = canvas.getContext('2d')
  if (!ctx) return

  const width = canvas.width
  const height = canvas.height

  // 基础色相
  ctx.fillStyle = `hsl(${hue.value}, 100%, 50%)`
  ctx.fillRect(0, 0, width, height)

  // 白色渐变（水平）
  const whiteGradient = ctx.createLinearGradient(0, 0, width, 0)
  whiteGradient.addColorStop(0, '#fff')
  whiteGradient.addColorStop(1, 'transparent')
  ctx.fillStyle = whiteGradient
  ctx.fillRect(0, 0, width, height)

  // 黑色渐变（垂直）
  const blackGradient = ctx.createLinearGradient(0, 0, 0, height)
  blackGradient.addColorStop(0, 'transparent')
  blackGradient.addColorStop(1, '#000')
  ctx.fillStyle = blackGradient
  ctx.fillRect(0, 0, width, height)
}

// 绘制色相条
function drawHue() {
  if (!hueCanvasRef.value) return
  const canvas = hueCanvasRef.value
  const ctx = canvas.getContext('2d')
  if (!ctx) return

  const height = canvas.height
  const hueGradient = ctx.createLinearGradient(0, 0, 0, height)
  hueGradient.addColorStop(0, 'hsl(0, 100%, 50%)')
  hueGradient.addColorStop(0.17, 'hsl(298.8, 100%, 50%)')
  hueGradient.addColorStop(0.33, 'hsl(241.2, 100%, 50%)')
  hueGradient.addColorStop(0.5, 'hsl(180, 100%, 50%)')
  hueGradient.addColorStop(0.67, 'hsl(118.8, 100%, 50%)')
  hueGradient.addColorStop(0.83, 'hsl(61.2, 100%, 50%)')
  hueGradient.addColorStop(1, 'hsl(360, 100%, 50%)')

  ctx.fillStyle = hueGradient
  ctx.fillRect(0, 0, canvas.width, height)
}

// 绘制透明度条
function drawAlpha() {
  if (!alphaCanvasRef.value) return
  const canvas = alphaCanvasRef.value
  const ctx = canvas.getContext('2d')
  if (!ctx) return

  const width = canvas.width
  const height = canvas.height

  // 透明度渐变
  const alphaGradient = ctx.createLinearGradient(0, 0, 0, height)
  alphaGradient.addColorStop(0, 'rgba(255, 255, 255, 1)')
  alphaGradient.addColorStop(1, 'rgba(255, 255, 255, 0)')

  ctx.fillStyle = alphaGradient
  ctx.fillRect(0, 0, width, height)
}

// 色谱拖拽
function startSpectrumDrag(e: MouseEvent) {
  if (!spectrumCanvasRef.value) return
  isDraggingSpectrum.value = true
  handleSpectrumDrag(e)
  window.addEventListener('mousemove', handleSpectrumDrag)
  window.addEventListener('mouseup', endSpectrumDrag)
}

function handleSpectrumDrag(e: MouseEvent) {
  if (!spectrumCanvasRef.value || !spectrumMapRef.value) return
  const rect = spectrumMapRef.value.getBoundingClientRect()

  let x = e.clientX - rect.left
  let y = e.clientY - rect.top

  // 限制范围
  x = Math.max(0, Math.min(x, rect.width))
  y = Math.max(0, Math.min(y, rect.height))

  spectrumCursorPos.value = { x, y }

  // 更新饱和度和亮度
  const xRatio = x / rect.width
  const yRatio = y / rect.height
  const hsvValue = 1 - yRatio
  const hsvSaturation = xRatio

  const newLightness = (hsvValue / 2) * (2 - hsvSaturation)
  const denominator = 1 - Math.abs(2 * newLightness - 1)

  lightness.value = Math.max(0, Math.min(1, newLightness))
  saturation.value =
    denominator !== 0 ? Math.max(0, Math.min(1, (hsvValue * hsvSaturation) / denominator)) : 0
}

function endSpectrumDrag() {
  isDraggingSpectrum.value = false
  window.removeEventListener('mousemove', handleSpectrumDrag)
  window.removeEventListener('mouseup', endSpectrumDrag)
}

// 色相拖拽
function startHueDrag(e: MouseEvent) {
  if (!hueCanvasRef.value) return
  isDraggingHue.value = true
  handleHueDrag(e)
  window.addEventListener('mousemove', handleHueDrag)
  window.addEventListener('mouseup', endHueDrag)
}

function handleHueDrag(e: MouseEvent) {
  if (!hueCanvasRef.value) return
  const canvas = hueCanvasRef.value
  const rect = canvas.getBoundingClientRect()

  let y = e.clientY - rect.top
  y = Math.max(0, Math.min(y, rect.height))

  hueCursorPos.value = { y }

  const percent = y / rect.height
  hue.value = 360 - 360 * percent

  drawSpectrum()
}

function endHueDrag() {
  isDraggingHue.value = false
  window.removeEventListener('mousemove', handleHueDrag)
  window.removeEventListener('mouseup', endHueDrag)
}

// 透明度拖拽
function startAlphaDrag(e: MouseEvent) {
  if (!alphaCanvasRef.value) return
  isDraggingAlpha.value = true
  handleAlphaDrag(e)
  window.addEventListener('mousemove', handleAlphaDrag)
  window.addEventListener('mouseup', endAlphaDrag)
}

function handleAlphaDrag(e: MouseEvent) {
  if (!alphaCanvasRef.value) return
  const canvas = alphaCanvasRef.value
  const rect = canvas.getBoundingClientRect()

  let y = e.clientY - rect.top
  y = Math.max(0, Math.min(y, rect.height))

  alphaCursorPos.value = { y }

  const percent = 1 - y / rect.height
  alpha.value = Math.max(0, Math.min(1, percent))
}

function endAlphaDrag() {
  isDraggingAlpha.value = false
  window.removeEventListener('mousemove', handleAlphaDrag)
  window.removeEventListener('mouseup', endAlphaDrag)
}

// 切换格式
function toggleFormat() {
  const currentFormats = formats.value as readonly FormatType[]
  const currentIndex = currentFormats.indexOf(currentFormat.value)
  const nextIndex = (currentIndex + 1) % currentFormats.length
  const nextFormat = currentFormats[nextIndex]
  if (nextFormat) {
    currentFormat.value = nextFormat
    // 标记用户已手动切换过格式
    userHasSwitchedFormat.value = true
    // 触发 v-model 更新，使新格式的值同步到父组件
    const value = colorValue.value
    if (value !== undefined) {
      emit('update:modelValue', value)
    }
  }
}

// 监听颜色变化和格式变化，触发 v-model 更新
watch([hue, saturation, lightness, alpha, currentFormat], () => {
  const value = colorValue.value
  if (value !== undefined) {
    emit('update:modelValue', value)
  }
})

// 监听 alpha 变化，当从不透明变为透明时，自动切换到带透明度的格式
watch(alpha, (newAlpha, oldAlpha) => {
  // 只有当用户没有指定格式列表，且用户没有手动切换过格式时，才自动切换格式
  if (!props.formats || props.formats.length === 0) {
    // 如果用户已经手动切换过格式，保持用户的选择
    if (userHasSwitchedFormat.value) return

    const currentFormatSupportsAlpha = isAlphaFormat(currentFormat.value)

    // 从不透明变为透明，自动切换到对应的带透明度格式
    if (newAlpha < 1 && oldAlpha === 1 && !currentFormatSupportsAlpha) {
      switch (currentFormat.value) {
        case 'hex':
          currentFormat.value = 'hex8'
          break
        case 'rgb':
          currentFormat.value = 'rgba'
          break
        case 'hsl':
          currentFormat.value = 'hsv' // hsl/hsla 不支持透明度，切换到 hsv
          break
        default:
          currentFormat.value = 'hex8'
          break
      }
    }
    // 从透明变为不透明，自动切换到对应的不带透明度格式
    else if (newAlpha === 1 && oldAlpha < 1 && currentFormatSupportsAlpha) {
      switch (currentFormat.value) {
        case 'hex8':
          currentFormat.value = 'hex'
          break
        case 'rgba':
          currentFormat.value = 'rgb'
          break
        case 'hsv':
          currentFormat.value = 'hsl' // hsv 不支持不透明格式，切换到 hsl
          break
        default:
          currentFormat.value = 'hex'
          break
      }
    }
  }
})

// 监听外部 modelValue 变化
watch(
  () => props.modelValue,
  (newValue, oldValue) => {
    // 只有当值真正改变时才重新解析
    // 这样可以避免格式切换时的循环
    if (newValue && newValue !== oldValue && newValue !== colorValue.value) {
      setColorFromString(newValue)
    }
  },
  { immediate: true },
)

// 监听格式变化，确保 currentFormat 在可用格式列表中
watch(
  formats,
  (newFormats) => {
    // 如果正在设置颜色，跳过格式检查
    if (isSettingColor.value) return

    // 如果用户已经手动切换过格式，保持用户的选择
    if (userHasSwitchedFormat.value) return

    if (newFormats.length > 0 && !newFormats.some((f) => f === currentFormat.value)) {
      // 如果当前格式不在可用格式列表中，切换到第一个可用格式
      currentFormat.value = newFormats[0] as FormatType
    }
  },
  { immediate: true },
)

// 更新游标位置
function updateCursorPositions() {
  if (!spectrumMapRef.value || !hueCanvasRef.value) return

  const spectrumRect = spectrumMapRef.value.getBoundingClientRect()
  const hueRect = hueCanvasRef.value.getBoundingClientRect()

  // 计算色相游标位置
  const hueY = hueRect.height - (hue.value / 360) * hueRect.height
  hueCursorPos.value = { y: Math.max(0, Math.min(hueY, hueRect.height)) }

  // 计算色谱游标位置
  const hsv = hslToHsv(hue.value, saturation.value, lightness.value)
  const x = spectrumRect.width * hsv.s
  const y = spectrumRect.height * (1 - hsv.v)
  spectrumCursorPos.value = {
    x: Math.max(0, Math.min(x, spectrumRect.width)),
    y: Math.max(0, Math.min(y, spectrumRect.height)),
  }

  // 计算透明度游标位置（如果显示透明度条）
  if (alphaCanvasRef.value) {
    const alphaRect = alphaCanvasRef.value.getBoundingClientRect()
    const alphaY = alphaRect.height * (1 - alpha.value)
    alphaCursorPos.value = { y: Math.max(0, Math.min(alphaY, alphaRect.height)) }
  }
}

// 从颜色字符串解析并设置颜色
function setColorFromString(colorStr: string) {
  let detectedFormat: FormatType | null = null

  // 标记正在设置颜色，跳过格式检查
  isSettingColor.value = true

  // 解析 Hex 格式
  if (colorStr.startsWith('#')) {
    let hex = colorStr.slice(1)

    // 处理 hex8 格式 (#RRGGBBAA)
    if (hex.length === 8) {
      detectedFormat = 'hex8'
      const alphaValue = parseInt(hex.slice(6, 8), 16) / 255
      alpha.value = alphaValue
      hex = hex.slice(0, 6)
    }
    // 处理缩写 hex4 格式 (#RGBA)
    else if (hex.length === 4) {
      detectedFormat = 'hex8'
      const lastChar = hex[3]
      if (lastChar) {
        const alphaValue = parseInt(lastChar + lastChar, 16) / 255
        alpha.value = alphaValue
      }
      hex = hex
        .slice(0, 3)
        .split('')
        .map((c) => c + c)
        .join('')
    }
    // 处理缩写 hex 格式 (#RGB)
    else if (hex.length === 3) {
      detectedFormat = 'hex'
      hex = hex
        .split('')
        .map((c) => c + c)
        .join('')
    }
    // 标准格式 (#RRGGBB)
    else if (hex.length === 6) {
      detectedFormat = 'hex'
    }

    const r = parseInt(hex.slice(0, 2), 16)
    const g = parseInt(hex.slice(2, 4), 16)
    const b = parseInt(hex.slice(4, 6), 16)

    const rgbToHsl = (r: number, g: number, b: number) => {
      const r2 = r / 255
      const g2 = g / 255
      const b2 = b / 255

      const max = Math.max(r2, g2, b2)
      const min = Math.min(r2, g2, b2)
      let h = 0
      let s = 0
      const l = (max + min) / 2

      if (max !== min) {
        const d = max - min
        s = l > 0.5 ? d / (2 - max - min) : d / (max + min)

        switch (max) {
          case r2:
            h = ((g2 - b2) / d + (g2 < b2 ? 6 : 0)) / 6
            break
          case g2:
            h = ((b2 - r2) / d + 2) / 6
            break
          case b2:
            h = ((r2 - g2) / d + 4) / 6
            break
        }
      }

      return { h: h * 360, s, l }
    }

    const hsl = rgbToHsl(r, g, b)
    hue.value = hsl.h
    saturation.value = hsl.s
    lightness.value = hsl.l

    // 直接设置检测到的格式，即使它不在当前 formats 列表中
    // 这样可以保证用户输入的格式被正确显示
    if (detectedFormat) {
      currentFormat.value = detectedFormat
    }

    // 更新游标位置
    updateCursorPositions()
  }
  // 解析 rgb/rgba 格式
  else if (colorStr.startsWith('rgb')) {
    const match = colorStr.match(/rgba?\((\d+),\s*(\d+),\s*(\d+)(?:,\s*([\d.]+))?\)/)
    if (match && match[1] && match[2] && match[3]) {
      detectedFormat = match[4] ? 'rgba' : 'rgb'
      const r = parseInt(match[1])
      const g = parseInt(match[2])
      const b = parseInt(match[3])
      const alphaValue = match[4] ? parseFloat(match[4]) : 1

      alpha.value = alphaValue

      // 先设置颜色值，这样 alpha.value 更新后 formats 会重新计算
      const rgbToHsl = (r: number, g: number, b: number) => {
        const r2 = r / 255
        const g2 = g / 255
        const b2 = b / 255

        const max = Math.max(r2, g2, b2)
        const min = Math.min(r2, g2, b2)
        let h = 0
        let s = 0
        const l = (max + min) / 2

        if (max !== min) {
          const d = max - min
          s = l > 0.5 ? d / (2 - max - min) : d / (max + min)

          switch (max) {
            case r2:
              h = ((g2 - b2) / d + (g2 < b2 ? 6 : 0)) / 6
              break
            case g2:
              h = ((b2 - r2) / d + 2) / 6
              break
            case b2:
              h = ((r2 - g2) / d + 4) / 6
              break
          }
        }

        return { h: h * 360, s, l }
      }

      const hsl = rgbToHsl(r, g, b)
      hue.value = hsl.h
      saturation.value = hsl.s
      lightness.value = hsl.l

      // 直接设置检测到的格式，即使它不在当前 formats 列表中
      // 这样可以保证用户输入的格式被正确显示
      if (detectedFormat) {
        currentFormat.value = detectedFormat
      }

      // 更新游标位置
      updateCursorPositions()
    }
  }
  // 解析 hsl/hsla 格式
  else if (colorStr.startsWith('hsl')) {
    const match = colorStr.match(/hsla?\((\d+),\s*(\d+)%,\s*(\d+)%(?:,\s*([\d.]+))?\)/)
    if (match && match[1] && match[2] && match[3]) {
      detectedFormat = 'hsl'
      const h = parseInt(match[1])
      const s = parseInt(match[2]) / 100
      const l = parseInt(match[3]) / 100
      const alphaValue = match[4] ? parseFloat(match[4]) : 1

      hue.value = h
      saturation.value = s
      lightness.value = l
      alpha.value = alphaValue

      // 直接设置检测到的格式，即使它不在当前 formats 列表中
      // 这样可以保证用户输入的格式被正确显示
      if (detectedFormat) {
        currentFormat.value = detectedFormat
      }

      // 更新游标位置
      updateCursorPositions()
    }
  }
  // 解析 hsv 格式
  else if (colorStr.startsWith('hsv')) {
    detectedFormat = 'hsv'
    const match = colorStr.match(/hsv\((\d+),\s*(\d+)%,\s*(\d+)%\)/)
    if (match && match[1] && match[2] && match[3]) {
      const h = parseInt(match[1])
      const s = parseInt(match[2]) / 100
      const v = parseInt(match[3]) / 100

      // HSV 转 HSL
      const l = (v * (2 - s)) / 2
      const s2 = v === 0 ? 0 : (v * s) / (1 - Math.abs(2 * l - 1))

      hue.value = h
      saturation.value = s2
      lightness.value = l

      // 直接设置检测到的格式，即使它不在当前 formats 列表中
      // 这样可以保证用户输入的格式被正确显示
      if (detectedFormat) {
        currentFormat.value = detectedFormat
      }

      // 更新游标位置
      updateCursorPositions()
    }
  }

  // 如果检测到的格式与当前格式不同，说明是新的输入，重置用户切换标志
  if (detectedFormat && detectedFormat !== currentFormat.value) {
    userHasSwitchedFormat.value = false
  }

  // 清除正在设置颜色的标志
  isSettingColor.value = false
}

// 复制颜色值
function copyColorValue() {
  navigator.clipboard
    .writeText(colorValue.value)
    .then(() => {
      copyTitle.value = '已复制!'
      setTimeout(() => {
        copyTitle.value = '点击复制'
      }, 1500)
    })
    .catch(() => {
      copyTitle.value = '复制失败'
      setTimeout(() => {
        copyTitle.value = '点击复制'
      }, 1500)
    })
}

// 监听色相变化，重绘色谱图
watch(hue, () => {
  drawSpectrum()
})

// 组件根元素引用
const panelRef = ref<HTMLElement | null>(null)

// 标记是否忽略下一次点击事件（用于避免刚打开时的点击事件）
let ignoreNextClick = false

// 处理点击外部
function handleClickOutside(event: MouseEvent) {
  if (ignoreNextClick) {
    ignoreNextClick = false
    return
  }
  if (!panelRef.value) return
  if (!panelRef.value.contains(event.target as Node)) {
    emit('close')
  }
}

onMounted(() => {
  initCanvases()
  window.addEventListener('resize', initCanvases)

  // 如果启用了点击外部关闭，添加点击监听
  if (props.closeOnOutsideClick) {
    // 忽略当前正在处理的点击事件（如果是刚打开时触发的事件）
    ignoreNextClick = true
    window.addEventListener('click', handleClickOutside)
  }
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', initCanvases)
  window.removeEventListener('mousemove', handleSpectrumDrag)
  window.removeEventListener('mousemove', handleHueDrag)
  window.removeEventListener('mousemove', handleAlphaDrag)
  window.removeEventListener('click', handleClickOutside)
})
</script>

<style scoped>
.color-picker-panel {
  background: #1f232a;
  width: 280px;
  border-radius: 8px;
  border: 2px solid #364347;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
  padding: 10px;
  user-select: none;
  -ms-user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
}

.panel-row {
  position: relative;
}

.spectrum-map {
  position: relative;
  width: 200px;
  height: 200px;
  overflow: hidden;
  background: #fff;
  border-radius: 4px;
}

.hue-map,
.alpha-map {
  position: absolute;
  top: 5px;
  bottom: 5px;
  width: 16px;
  border-radius: 4px;
  overflow: hidden;
}

.hue-map {
  right: 27px;
}

.alpha-map {
  right: 0px;
}

.alpha-checkboard {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image:
    linear-gradient(45deg, #ccc 25%, transparent 25%),
    linear-gradient(-45deg, #ccc 25%, transparent 25%),
    linear-gradient(45deg, transparent 75%, #ccc 75%),
    linear-gradient(-45deg, transparent 75%, #ccc 75%);
  background-size: 8px 8px;
  background-position:
    0 0,
    0 4px,
    4px -4px,
    -4px 0px;
  z-index: 0;
  border-radius: 4px;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.color-cursor {
  border-radius: 50%;
  background: #ccc;
  box-sizing: border-box;
  position: absolute;
  pointer-events: none;
  z-index: 2;
  border: 2px solid #fff;
  box-shadow: 0 0 2px rgba(0, 0, 0, 0.5);
  transition: transform 0.1s;
}

.spectrum-cursor {
  width: 30px;
  height: 30px;
  margin-left: -15px;
  margin-top: -15px;
}

.hue-cursor,
.alpha-cursor {
  left: 50%;
  width: 20px;
  height: 20px;
  margin-left: -10px;
  margin-top: -10px;
  top: 0;
}

.hue-cursor.dragging,
.alpha-cursor.dragging,
.spectrum-cursor.dragging {
  transition: none;
}

.value-row {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  align-content: center;
  justify-content: space-between;
  align-items: center;
  column-gap: 4px;
  margin-top: 10px;
}

.color-preview-container {
  position: relative;
  width: 40px;
  height: 40px;
  border-radius: 4px;
  border: 2px solid #364347;
  flex-shrink: 0;
  overflow: hidden;
}

.color-preview-checkboard {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image:
    linear-gradient(45deg, #ccc 25%, transparent 25%),
    linear-gradient(-45deg, #ccc 25%, transparent 25%),
    linear-gradient(45deg, transparent 75%, #ccc 75%),
    linear-gradient(-45deg, transparent 75%, #ccc 75%);
  background-size: 10px 10px;
  background-position:
    0 0,
    0 5px,
    5px -5px,
    -5px 0px;
  z-index: 1;
}

.color-preview-color {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
}

.color-value {
  flex: 1;
  min-width: 0;
  background: #15191c;
  border: 1px solid #364347;
  border-radius: 2px;
  line-height: 38px;
  padding: 0 10px;
  text-align: center;
  color: #8b949e;
  font-size: 14px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  cursor: pointer;
  user-select: none;
}

.color-value:hover {
  background: #1a1f24;
  border-color: #4a5960;
}
</style>
