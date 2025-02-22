<template>
  <n-dropdown
    :show="dropdownShow"
    :options="options"
    :x="x"
    :y="y"
    placement="bottom-start"
    @clickoutside="handleHideDropdown"
    @select="handleSelect"
  />
</template>

<script setup>
import { computed } from 'vue'
import { useTagsStore } from '@/store/modules/tags'
import { IconRefresh, IconClose, IconExpand, IconExpandLeft, IconExpandRight } from '@/components/AppIcons'
import { renderIcon } from '@/utils/icon'
import { useAppStore } from '@/store/modules/app'

const props = defineProps({
  show: {
    type: Boolean,
    default: false,
  },
  currentPath: {
    type: String,
    default: '',
  },
  x: {
    type: Number,
    default: 0,
  },
  y: {
    type: Number,
    default: 0,
  },
})

const emit = defineEmits(['update:show'])

const tagsStore = useTagsStore()
const appStore = useAppStore()

const options = computed(() => [
  {
    label: '重新加载',
    key: 'reload',
    disabled: props.currentPath !== tagsStore.activeTag,
    icon: renderIcon(IconRefresh, { size: '14px' }),
  },
  {
    label: '关闭',
    key: 'close',
    disabled: tagsStore.tags.length <= 1,
    icon: renderIcon(IconClose, { size: '14px' }),
  },
  {
    label: '关闭其他',
    key: 'close-other',
    disabled: tagsStore.tags.length <= 1,
    icon: renderIcon(IconExpand, { size: '14px' }),
  },
  {
    label: '关闭左侧',
    key: 'close-left',
    disabled: tagsStore.tags.length <= 1 || props.currentPath === tagsStore.tags[0].path,
    icon: renderIcon(IconExpandLeft, { size: '14px' }),
  },
  {
    label: '关闭右侧',
    key: 'close-right',
    disabled: tagsStore.tags.length <= 1 || props.currentPath === tagsStore.tags[tagsStore.tags.length - 1].path,
    icon: renderIcon(IconExpandRight, { size: '14px' }),
  },
])

const dropdownShow = computed({
  get() {
    return props.show
  },
  set(show) {
    emit('update:show', show)
  },
})

const actionMap = new Map([
  [
    'reload',
    () => {
      appStore.reloadPage()
    },
  ],
  [
    'close',
    () => {
      tagsStore.removeTag(props.currentPath)
    },
  ],
  [
    'close-other',
    () => {
      tagsStore.removeOther(props.currentPath)
    },
  ],
  [
    'close-left',
    () => {
      tagsStore.removeLeft(props.currentPath)
    },
  ],
  [
    'close-right',
    () => {
      tagsStore.removeRight(props.currentPath)
    },
  ],
])

function handleHideDropdown() {
  dropdownShow.value = false
}

function handleSelect(key) {
  const actionFn = actionMap.get(key)
  actionFn && actionFn()
  handleHideDropdown()
}
</script>
