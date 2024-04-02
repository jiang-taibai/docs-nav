<script setup>
import {NNotificationProvider} from 'naive-ui'
import MainView from "./view/MainView.vue";
</script>

<template>
  <n-notification-provider>
    <main-view/>
  </n-notification-provider>
</template>

<style scoped>
</style>
