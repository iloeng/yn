<template>
  <FixedFloat
    class="quick-filter"
    :top="props.top"
    :right="props.right"
    :bottom="props.bottom"
    :left="props.left"
    :disable-auto-focus="!props.filterInputHidden"
    @close="close"
    @keypress.enter.stop.exact="chooseItem()"
    @keydown.up.stop.exact="selectItem(-1)"
    @keydown.down.stop.exact="selectItem(1)"
  >
    <div class="input">
      <input
        v-auto-focus
        type="text"
        :placeholder="placeholder"
        v-model="keyword"
        v-if="!props.filterInputHidden"
        @blur.stop="close"
        @keydown.esc="close"
      />
    </div>
    <div ref="refList" class="list" :style="{width: fixedWidth}">
      <div
        v-for="item in list"
        :key="item.key"
        :class="{
          item: true,
          selected: selected && selected.key === item.key,
        }"
        @mousedown="chooseItem(item)"
        @mouseover="updateSelected(item)"
      >
        <svg-icon class="checked-icon" v-if="item.key === props.current" name="check-solid" />
        <span>{{ item.label }}</span>
      </div>
    </div>
  </FixedFloat>
</template>

<script lang="ts" setup>
import { Components } from '@fe/types'
import { computed, nextTick, onMounted, ref, watch } from 'vue'
import SvgIcon from './SvgIcon.vue'
import FixedFloat from './FixedFloat.vue'

type Item = Components.QuickFilter.Item
type Props = Components.QuickFilter.Props

const props = defineProps<Props>()

const refList = ref<HTMLElement | null>(null)
const emit = defineEmits(['close', 'choose', 'input'])
const fixedWidth = ref(undefined as string | undefined)
const keyword = ref('')
const selected = ref<Item | null>(null)
const list = computed(() => props.list.filter(
  item => item.label.toLowerCase().includes(keyword.value.toLowerCase())
))

function close () {
  nextTick(() => {
    emit('close')
  })
}

onMounted(() => {
  updateSelected()
})

function updateSelected (item: Item | null = null) {
  if (item) {
    selected.value = item
  } else {
    selected.value = list.value.find(x => x.key === props.current) || list.value.length > 0 ? list.value[0] : null
  }

  nextTick(() => {
    if (refList.value) {
      const el = refList.value.querySelector<any>('.selected')
      el?.scrollIntoViewIfNeeded()
    }
  })
}

function selectItem (inc: number) {
  if (list.value.length < 1) {
    updateSelected()
    return
  }

  const currentIndex = list.value.findIndex(x => selected.value?.key === x.key)

  let index = currentIndex + inc
  if (index > list.value.length - 1) {
    index = 0
  } else if (index < 0) {
    index = list.value.length - 1
  }

  updateSelected(list.value[index])
}

function chooseItem (item: Item | null = null) {
  if (item) {
    emit('choose', item)
  } else if (selected.value) {
    emit('choose', selected.value)
  }

  close()
}

watch(() => keyword.value, (val) => {
  emit('input', val)
  updateSelected()

  if (val) {
    fixedWidth.value = refList.value?.offsetWidth + 'px'
  } else {
    fixedWidth.value = undefined
  }
})

</script>

<style lang="scss" scoped>
.quick-filter {
  input[type="text"] {
    border-bottom-left-radius: 0;
    border-bottom-right-radius: 0;
    font-size: 14px;
    padding: 4px 8px;
    box-shadow: rgba(0, 0, 0, 0.3) 0px 0px 2px;

    &:focus {
      background: rgba(var(--g-color-82-rgb), 0.5);
    }
  }

  .list {
    width: 100%;
    max-height: 70vh;
    overflow-y: auto;
    user-select: none;
    padding: 3px 1px;
    box-sizing: border-box;

    .item {
      padding: 0 20px;
      line-height: 1.9;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      color: var(--g-color-20);
      border-radius: var(--g-border-radius);
      font-size: 14px;
      font-variant-numeric: tabular-nums;

      &.selected {
        background: var(--g-color-active-a);
        color: var(--g-color-5);
      }

      .checked-icon {
        position: absolute;
        width: 10px;
        height: 10px;
        transform: translateX(-14px) translateY(10px) scaleX(0.8);
      }
    }
  }
}
</style>
