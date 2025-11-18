<script setup>
import { RouterView } from "vue-router";
import { ref, provide } from "vue";

const isSignedIn = ref(false);
provide("is-signed-in", isSignedIn.value);
</script>

<template>
  <RouterView />
</template>
