<script setup>
import { RouterView } from 'vue-router'
import Loading from 'vue-loading-overlay'
import { ref, onMounted } from 'vue';

const loadingView = ref({
  color: 'green',
  fullPage: true,
})

const isLoading = ref(false);
const loadingEffect = ()=> {
  isLoading.value = true
  setTimeout(() => {
    isLoading.value = false
  }, 1000)
}

onMounted(()=>{
  loadingEffect()
});
</script>

<template>
  <Loading :is-full-page="loadingView.fullPage" v-model:active="isLoading" :color="loadingView.color"/>
  <RouterView />
</template>

<style scoped>

</style>
