<template>
  <div class="custom-select" v-on-key-stroke:ArrowUp="goPrevValue" v-on-key-stroke:ArrowDown="goNextValue"
    v-on-click-outside="() => (opened = false)" v-on-key-stroke:Enter="release">
    <select class="select select--default" v-model="modelValue" v-show="false" :disabled="disabled">
      <option v-for="{ title, value } in currentItems" :key="value" :value="value">{{ title }}</option>
    </select>
    <button type="button" v-if="selected" @click="release" :disabled="disabled"
      class="custom-select__option custom-select__option--value">
      {{ selected.title }}
    </button>
    <slot v-else name="placeholder" :opened="opened">
      <button type="button" @click="release" :disabled="disabled"
        class="custom-select__option custom-select__option--value">
        {{ placeholder }}
      </button>
    </slot>
    <div class="custom-select__dropdown" v-show="!disabled && opened">
      <slot name="item" v-for="{ title, value } in currentItems" :key="value" :title="title" :value="value"
        :select="select">
        <button type="button" :value="value" :disabled="disabled" @click="select(value)" class="custom-select__option">
          {{ title }}
        </button>
      </slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { vOnClickOutside, vOnKeyStroke } from '@vueuse/components'

type ValueType = number | string | Record<string, string | number>

const modelValue = defineModel()
const props = withDefaults(defineProps<{
  items: Record<string, ValueType> | ValueType[]
  itemTitle: string
  itemValue: string
  disabled: boolean
  placeholder: string
}>(), {
  items: () => [],
  itemTitle: 'title',
  itemValue: 'value',
  disabled: false,
  placeholder: ''
})

const currentItems = computed(() =>
  (Array.isArray(props.items)
    ? props.items.map((item, index) => [index, item])
    : props.items instanceof Object
      ? Object.entries(props.items)
      : []
  ).map(([key, item], index) => (
    item instanceof Object
      ? Array.isArray(props.items)
        ? { title: item[props.itemTitle], value: item[props.itemValue] as string | number, index }
        : {
          title: item[props.itemTitle],
          value: props.itemValue in item ? item[props.itemValue] : key as string | number,
          index
        }
      : Array.isArray(props.items)
        ? { title: item, value: item as string | number, index } // 
        : { title: item, value: key as string, index }
  ))
)

const selected = computed(() => currentItems.value.find(({ value }) => value === modelValue.value))

const prevValue = computed(
  () =>
    currentItems.value[
      (selected.value?.index ?? 0) > 0
        ? (selected.value?.index ?? 1) - 1
        : currentItems.value.length - 1
    ].value
)
const nextValue = computed(
  () =>
    currentItems.value[
      (selected.value?.index ?? 0) >= currentItems.value.length - 1
        ? 0
        : (selected.value?.index ?? 0) + 1
    ].value
)

const opened = ref(false)

const goPrevValue = () => {
  if (opened.value) {
    modelValue.value = prevValue.value
  }
}

const goNextValue = () => {
  if (opened.value) {
    modelValue.value = nextValue.value
  }
}

const release = (e: Event) => {
  opened.value = !opened.value
  e.preventDefault()
}

const select = (value: ValueType) => {
  modelValue.value = value
  opened.value = false
}
</script>

<style scoped lang="scss">
.custom-select {
  position: relative;

  &__option {
    overflow: hidden;
    box-sizing: border-box;
    display: block;
    width: 100%;
    background-color: transparent;
    border: 0;
    border-radius: 0;
    font-family: inherit;
    white-space: nowrap;
    text-align: left;
    text-overflow: ellipsis;
    cursor: pointer;
    user-select: none;
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    position: relative;
    padding: 0.375rem 0.75rem;
    font-size: 1rem;
    color: #212121;
    line-height: 1.5;

    &:focus {
      outline: none;
    }

    &:hover,
    &:focus {
      background-color: darken(#fff, 2.5%);
    }

    &--value {
      padding-right: 0.75rem * 2 + 0.375rem;
      background-color: #fff;
      border: 1px solid darken(#fff, 10%);
      border-radius: 0.25rem;

      &::after {
        position: absolute;
        box-sizing: border-box;
        width: 0;
        height: 0;
        top: 50%;
        right: 0.75rem;
        border-width: 0.3rem;
        border-bottom-width: 0;
        border-style: solid;
        border-color: currentColor transparent;
        transform: translateY(-50%);
        content: '';

        .custom-select--active & {
          transform: translateY(-50%) rotate(-180deg);
        }
      }

      .custom-select--active & {
        border-bottom-color: transparent;
        border-radius: 0.25rem 0.25rem 0 0;

        &:hover,
        &:focus {
          background-color: #fff;
        }

        @at-root {
          .custom-select--dropup#{&} {
            border-top-color: transparent;
            border-bottom-color: darken(#fff, 10%);
            border-radius: 0 0 0.25rem 0.25rem;
          }
        }
      }
    }

    &--selected {
      background-color: darken(#fff, 1.25%);
    }

    &[disabled] {
      color: lighten(#212121, 50%);
      cursor: default;

      &:hover,
      &:focus {
        background-color: transparent;
      }
    }
  }

  &__option-wrap {
    position: relative;
    overflow-y: auto;
    max-height: (0.375rem * 2 + 1rem * 1.5) * 5;

    &::-webkit-scrollbar {
      width: 16px;
    }

    &::-webkit-scrollbar-thumb {
      background-color: darken(#fff, 10%);
      background-clip: padding-box;
      border-width: 0 4px;
      border-style: solid;
      border-color: transparent;
    }
  }

  &__input {
    box-sizing: border-box;
    display: block;
    width: 100%;
    padding: 0;
    border-width: 1px 0;
    border-style: solid;
    border-radius: 0;
    font-family: inherit;
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    position: relative;
    z-index: 1;
    height: 2.25rem;
    margin-top: -1px;
    padding: 0 0.75rem;
    border-color: darken(#fff, 10%);
    transform: translateY(1px);
    font-size: 1rem;
    color: #212121;

    &:focus {
      outline: none;
    }

    .custom-select--dropup & {
      border-top-width: 0;
      margin-top: 0;
      transform: translateY(0);
    }
  }

  &__dropdown {
    position: absolute;
    box-sizing: border-box;
    width: 100%;
    top: 100%;
    left: 0;
    overflow: hidden;
    z-index: 1;
    top: calc(100% - 1px);
    background-color: #fff;
    border-width: 0 1px 1px;
    border-style: solid;
    border-color: darken(#fff, 10%);
    border-radius: 0 0 0.25rem 0.25rem;

    .custom-select--dropup & {
      top: auto;
      bottom: calc(100% - 1px);
      border-width: 1px 1px 0;
      border-radius: 0.25rem 0.25rem 0 0;
    }
  }
}
</style>
