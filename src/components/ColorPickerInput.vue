<template>
  <div class="color-picker-input" ref="containerRef">
    <input ref="inputRef" :value="modelValue" @click="handleClick" readonly class="color-input" />

    <Teleport to="body">
      <Transition name="fade">
        <div v-if="isOpen" class="color-picker-dropdown" ref="dropdownRef" :style="dropdownStyle">
          <ColorPicker v-model="localValue" />
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'
import ColorPicker from './ColorPicker.vue'

// Props
interface Props {
  modelValue: string
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: '',
})

// Emits
const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()

// 状态
const isOpen = ref(false)
const localValue = ref(props.modelValue)
const inputRef = ref<HTMLInputElement | null>(null)
const dropdownRef = ref<HTMLDivElement | null>(null)
const containerRef = ref<HTMLDivElement | null>(null)
const dropdownStyle = ref<Record<string, string>>({ position: 'absolute', display: 'none' })

// 处理点击
function handleClick() {
  if (!inputRef.value) return

  const inputRect = inputRef.value.getBoundingClientRect()
  const viewportWidth = window.innerWidth
  const viewportHeight = window.innerHeight

  // 面板宽度约 280px，高度约 240px
  const panelWidth = 300
  const panelHeight = 260
  const offset = 4

  // 计算水平位置
  let left = inputRect.left
  if (left + panelWidth > viewportWidth) {
    // 如果右侧空间不足，显示在左侧
    left = inputRect.right - panelWidth
  }
  // 确保不超出左边界
  if (left < 0) {
    left = 0
  }

  // 计算垂直位置
  let top = inputRect.bottom + offset
  if (top + panelHeight > viewportHeight) {
    // 如果下方空间不足，显示在上方
    top = inputRect.top - panelHeight - offset
  }
  // 确保不超出上边界
  if (top < 0) {
    top = 0
  }

  dropdownStyle.value = {
    position: 'absolute',
    left: `${left}px`,
    top: `${top}px`,
    display: 'block',
  }

  isOpen.value = true
}

// 监听外部 modelValue 变化
watch(
  () => props.modelValue,
  (newValue) => {
    if (newValue !== localValue.value) {
      localValue.value = newValue
    }
  },
  { immediate: true },
)

// 监听内部颜色值变化，同步到外部
watch(localValue, (newValue) => {
  emit('update:modelValue', newValue)
})

// 监听窗口滚动，更新面板位置
function updateDropdownPosition() {
  if (isOpen.value && inputRef.value) {
    handleClick()
  }
}

watch(isOpen, (newValue) => {
  if (newValue) {
    window.addEventListener('scroll', updateDropdownPosition, true)
    window.addEventListener('resize', updateDropdownPosition)
  } else {
    window.removeEventListener('scroll', updateDropdownPosition, true)
    window.removeEventListener('resize', updateDropdownPosition)
  }
})

// 点击外部关闭
function handleClickOutside(event: MouseEvent) {
  if (
    isOpen.value &&
    dropdownRef.value &&
    !dropdownRef.value.contains(event.target as Node) &&
    !inputRef.value?.contains(event.target as Node)
  ) {
    isOpen.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
})

onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside)
})
</script>

<style scoped>
.color-picker-input {
  position: relative;
  display: inline-block;
}

.color-input {
  padding: 8px 12px;
  border: 1px solid #364347;
  border-radius: 4px;
  background: #15191c;
  color: #8b949e;
  font-size: 14px;
  cursor: pointer;
  min-width: 120px;
}

.color-input:focus {
  outline: none;
  border-color: #58a6ff;
}

.color-input::selection {
  background: rgba(88, 166, 255, 0.2);
}

.color-picker-dropdown {
  z-index: 9999;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
}

.fade-enter-active,
.fade-leave-active {
  transition:
    opacity 0.2s ease,
    transform 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

.fade-enter-to,
.fade-leave-from {
  opacity: 1;
  transform: translateY(0);
}
</style>
