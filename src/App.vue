<template>
  <GojuonQuiz/>

  <component :is="'script'">
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());

    gtag('config', 'G-FNEHZZBCRR');
  </component>
</template>

<script>
import GojuonQuiz from './components/GojuonQuiz.vue'

export default {
  name: 'App',
  components: {
    GojuonQuiz,
  },
  mounted() {
    let gaScript = document.createElement('script')
    gaScript.setAttribute('src', 'https://www.googletagmanager.com/gtag/js?id=G-FNEHZZBCRR')
    document.head.appendChild(gaScript)
  },
}
</script>

<style>

</style>
