<template>
  <n-card style="margin-top: 1rem" v-show="props.data && props.data.length > 0">
    <n-tree :data="props.data ?? []" key-field="path" label-field="name" :default-expanded-keys="['/Assets']"
      @update:selected-keys="handleSelected" block-line expand-on-click />
  </n-card>
</template>

<script setup lang="ts">
import { ref, defineProps, h } from "vue";
import {
  NCard,
  NTree,
  useNotification,
  TreeOption,
  NotificationReactive,
} from "naive-ui";

const props = defineProps({
  data: Array<TreeOption>
})

interface Assets {
  id?: string;
  path: string;
  isFile: boolean;
  preview?: string;
  name?: string;
  prefix?: any;
  children?: Assets[];
}

const notification = useNotification();
const nRef = ref<NotificationReactive | null>(null);

const handleSelected = (
  _keys: Array<string | number>,
  option: Array<TreeOption | null>
) => {
  const a = (option[0] as unknown as Assets)
  const preview = a?.preview;
  if (!preview) {
    if (nRef.value) nRef.value.destroy()
    return nRef.value = null
  }
  if (!nRef.value) nRef.value = notification.create({
    title: a.name,
    content: () => h('img', { src: preview, style: 'width:100%' }),
    onClose: () => (nRef.value = null),

  })
  else {
    nRef.value.title = a.name
    nRef.value.content = () => h('img', { src: preview, style: 'width:100%' })
  }

};
</script>
