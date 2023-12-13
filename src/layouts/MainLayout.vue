
<template>
  <q-layout view="hHh lpR fFf">

    <q-header elevated class="t-bg-orange-400 text-white">
      <q-toolbar>
        <q-toolbar-title>
          <q-avatar>
            <q-icon :name="fasHatWizard" class="t-w-6 t-h-8" />
          </q-avatar>
          Create your imagination
        </q-toolbar-title>

        <q-btn dense flat round icon="menu" @click="toggleRightDrawer" />
      </q-toolbar>
    </q-header>

    <q-drawer v-model="rightDrawerOpen" side="right" bordered>
      <div class="t-flex t-justify-center t-items-center t-h-full">
        <div class="t-text-gray-600 t-text-xl">
          Maybe future...
        </div>
      </div>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>

  </q-layout>
</template>

<script>
import { ref } from 'vue'
import { fasHatWizard } from '@quasar/extras/fontawesome-v6'

export default {
  setup () {
    const rightDrawerOpen = ref(false)   
    return {
      fasHatWizard,
      rightDrawerOpen,
      toggleRightDrawer () {
        rightDrawerOpen.value = !rightDrawerOpen.value
      }
    }
  }
}
</script>