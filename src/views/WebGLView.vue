<template>
  <div class="WebGLView">
    <div class="menus-left" @click="handleClickMenus">
      <button v-for="item in components" :key="item">{{ item }}</button>
    </div>

    <div class="comps-right">
      <component v-if="component" :is="component"></component>
      <h2 v-else>点击任意按钮查看demo</h2>
    </div>
  </div>
</template>

<script setup>
import { ref, shallowRef, defineAsyncComponent } from 'vue';
const requireComponent = import.meta.glob(['@/components/webgl/*.vue', '!**/GLCanvas.vue', '!**/TheTemplate.vue']);

const component = shallowRef(null);
const components = ref([]);
const componentsMap = {};

Object.entries(requireComponent)
  .forEach(([key, importFn]) => {
    const compName = key.split('/').pop().replace(/\.\w+$/, '');
    components.value.push(compName);
    componentsMap[compName] = defineAsyncComponent(importFn);
  });

function handleClickMenus (ev) {
  if (ev.target.tagName !== "BUTTON") return;
  component.value = componentsMap[ev.target.textContent];
}

</script>

<style lang="scss" scoped>
.WebGLView {
  width: 100vw;
  height: 100vh;
  display: flex;
  padding: 8px;

  .menus-left {
    max-width: 300px;
  }

  .comps-right {
    border: 1px solid gainsboro;
    flex: 1;
    margin-left: 8px;
  }
}
</style>