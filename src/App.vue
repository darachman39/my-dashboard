<script setup>
import { RouterView } from 'vue-router';
import {
  NLayout,
  NLayoutContent
} from 'naive-ui';

import Header from './layouts/Header.vue';
import Sidebar from './layouts/Sidebar.vue';
import Footer from './layouts/Footer.vue';
</script>

<template>
  <NLayout has-sider>
    <Sidebar />
    <NLayout style="min-height: 100vh;">
      <Header />
      <NLayoutContent 
        content-style="padding: 24px; padding-bottom: 100px">
        <RouterView />
      </NLayoutContent>
      <Footer />
    </NLayout>
  </NLayout>
</template>
