<template>
  <q-page class="row items-center justify-evenly">
    <q-file color="orange" filled v-model="model" label="Label">
      <template v-slot:prepend>
        <q-icon name="cloud_upload" />
      </template>
    </q-file>
  </q-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';


</script>
