<script setup>
import { defineComponent } from "@vue/runtime-core";
import Canvas3DView from "@/components/Canvas3DView.vue";

defineComponent({
  Canvas3DView,
});
</script>

<template>
  <Canvas3DView />
</template>

<style>
@import "./assets/base.css";
#app {
  width: 100vw;
  height: 100vh;
  margin: 0 auto;
  padding: 2rem;
  font-weight: normal;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}
</style>
