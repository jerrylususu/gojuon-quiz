<template>
  <GojuonQuiz/>

  <div v-if="updateExists">
    {{ t('update_available') }}
    <button @click="refreshApp">
      {{ t('update_button') }}
    </button>
  </div>

  <component :is="'script'">
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());

    gtag('config', 'G-FNEHZZBCRR');
  </component>
</template>

<script>
import GojuonQuiz from './components/GojuonQuiz.vue'
import update from './mixins/update'
import { useI18n } from 'vue-i18n'


export default {
  name: 'App',
  components: {
    GojuonQuiz,
  },
  mixins: [update],
  mounted() {
    let gaScript = document.createElement('script')
    gaScript.setAttribute('src', 'https://www.googletagmanager.com/gtag/js?id=G-FNEHZZBCRR')
    document.head.appendChild(gaScript)
  },
  setup() {
    const { t } = useI18n({
        inheritLocale: true,
        missingWarn: false,
        fallbackWarn: false,
        // useScope: 'local',
    })

    return { t }
  }
}
</script>

<style>

</style>
