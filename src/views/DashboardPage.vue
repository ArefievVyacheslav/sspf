<template lang="pug">
  div
    b-icon#logout(icon="power" aria-hidden="true" font-scale="3rem" @click="logout")
    b-tooltip(target="logout" triggers="hover") logout

    b-icon#start(icon="play-fill" aria-hidden="true" font-scale="4rem" @click="startParsing(null)")
    b-tooltip(target="start" triggers="hover") start parsing

    b-icon#proxy(icon="list" aria-hidden="true" font-scale="4rem" v-b-modal.modal-proxy)
    b-tooltip(target="proxy" triggers="hover") proxy, активны до 29-ого

    b-jumbotron.parser-card(v-for="shop in data" :key="shop.name" :header="shop.name.charAt(0).toUpperCase() + shop.name.slice(1)")
      h2(v-if="shop.first" v-text="shop.first")
      h3(v-if="shop.second" v-text="shop.second")
      h4(v-if="shop.third" v-text="shop.third")
      h5(v-if="shop.fourth" v-text="shop.fourth")
      p(v-if="shop.total" v-text="shop.total")

      b-button.d-inline-flex.pt-2.pr-3.pl-2.pb-2(v-if="shop.total" variant="primary" @click="startParsing(shop.name)")
        b-icon(icon="arrow-clockwise" animation="spin" font-scale="1")
        .ml-2 Перезапуск

    b-modal#modal-proxy(scrollable ok-only title="Текущие прокси" ok-title="Сохранить" @ok="save")
      div.cp(v-if="!isEditProxies")
        p.m-0(v-for="proxy in proxies" :key="proxy" @click="isEditProxies = true") {{ proxy }}

      textarea.textarea(v-if="isEditProxies" v-model="proxiesTextarea")

</template>

<script>
export default {
  name: 'DashboardPage',
  data: () => ({
    data: null,
    proxies: null,
    proxiesTextarea: null,
    isEditProxies: false,
    intervalId: null
  }),
  computed: {
    proxiesReq () {
      return this.proxiesTextarea.split('\n').filter(proxy => !!proxy)
    }
  },
  methods: {
    logout () {
      window.localStorage.setItem('accessToken', null)
      this.$router.push('/login')
    },
    async startParsing (shop) {
      if (shop) {
        setTimeout(await this.getStatus, 1000)
        const { data } = await this.$axios.post('/start_parser', { shop: 'stockmann' })
        this.$bvToast.toast(data, { autoHideDelay: 5 * 1000, variant: 'success', noCloseButton: true })
      } else {
        setTimeout(await this.getStatus, 1000)
        const { data } = await this.$axios.post('/start_parser', { all: true })
        this.$bvToast.toast(data, { autoHideDelay: 5 * 1000, variant: 'success', noCloseButton: true })
      }
    },
    async getStatus () {
      const { data } = await this.$axios.get('/status')
      this.data = data
    },
    async getProxies () {
      try {
        const { data } = await this.$axios.get('/proxy')
        this.proxies = data
        this.$bvToast.toast('Прокси получены', { autoHideDelay: 5000, variant: 'success', noCloseButton: true })
      } catch (e) {
        this.$bvToast.toast(e, { autoHideDelay: 5000, variant: 'danger', noCloseButton: true })
      }
    },
    async save () {
      try {
        await this.$axios.put('/proxy', this.proxiesReq)
        this.$bvToast.toast('Прокси сохранены', { autoHideDelay: 5000, variant: 'success', noCloseButton: true })
        this.proxies = this.proxiesReq
      } catch (e) {
        this.$bvToast.toast(e, { autoHideDelay: 5000, variant: 'danger', noCloseButton: true })
      }
    }
  },
  async created () {
    await this.getStatus()
    await this.getProxies()
    this.proxiesTextarea = this.proxies.reduce((acc, proxy) => acc += proxy + '\n', '')
  },
  async mounted () {
    this.intervalId = setInterval(await this.getStatus, 3 * 1000)
  },
  beforeDestroy () {
    clearInterval(this.intervalId)
  }
}
</script>

<style lang="sass">
.parser-card
  max-width: 80%
  margin: 40px auto
  padding: 2rem !important
  border-radius: 18px

#logout,
#start,
#proxy
  position: absolute
  cursor: pointer

#logout
  top: 15px
  right: 15px

#start
  top: 7px
  right: 50px

#proxy
  top: 7px
  right: 100px
  outline: none

.textarea
  width: 100%
  min-height: 1000px

.cp
  cursor: pointer

</style>
