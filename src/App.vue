<script setup lang="ts">
import Countries from "./components/Countries.vue";
</script>

<template>
  <v-layout class="rounded rounded-md">

    <v-app-bar title="Country Service" color="grey" class="text-center"></v-app-bar>

    <v-main class="d-flex align-center justify-center">
      <Countries />
    </v-main>
  </v-layout>
</template>

