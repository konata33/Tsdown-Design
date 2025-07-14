
<script setup lang="ts">
import FifthSection from './components/FifthSection.vue';
import FourthSection from './components/FourthSection.vue';
import IntroSection from './components/IntroSection.vue';
import SecondSection from './components/SecondSection.vue';
import ThirdSection from './components/ThirdSection.vue';
</script>

<template>
  <div class="min-h-screen bg-gray-950">
    <!-- <HeadGuide /> -->
    <IntroSection />
    <SecondSection />
    <ThirdSection />
    <FourthSection />
    <FifthSection />
  </div>
</template>

<style>
html, body {
  margin: 0;
  padding: 0;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

* {
  box-sizing: border-box;
}

body {
  background-color: #030712;
  color: #f9fafb;
  line-height: 1.6;
}
</style>
