<template>
  <main>
    <form>
      <CustomSelect v-model="value" :items="data" item-title="title" item-value="value"
        placeholder="Выберите значение" />
    </form>
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import CustomSelect from '@/components/CustomSelect.vue'

const value = ref(0)

// const data = ref([1, 2, 3])
// const data = ref(["Option 1", "Option 2", "Option 3"])
// const data = ref([{ title: "Option 1", value: 1 }, { title: "Option 2", value: 2 }])
// const data = ref([{ name: "Option 1", id: 1 }, { name: "Option 2", id: 2 }])

// const data = ref({ value1: "Option 1", value2: "Option 2" })
type ValueType = number | string | Record<string, string | number>
const data = ref<Record<string, ValueType>>({
  value1: { title: 'Option 1', value: '1' },
  value2: { title: 'Option 2', value: 2 },
  value3: { title: 'Option 3' }
})
</script>

<style scoped></style>
