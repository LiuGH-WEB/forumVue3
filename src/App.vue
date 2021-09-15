<template>
  <div>
    <div class="header-bg">
      <div class="container">
        <global-header :user="currentUser"></global-header>
      </div>
    </div>
    <div class="container">
      <loader
        v-if="isLoading"
        text="拼命加载中"
      ></loader>
      <router-view></router-view>
    </div>
    <div class="footer-bg">
      <div class="container">
        <footer class="text-center py-4 text-secondary bg-light mt-6">
        <small>
          <ul class="list-inline mb-0">
            <li class="list-inline-item">© 2020 者也专栏</li>
            <li class="list-inline-item">课程</li>
            <li class="list-inline-item">文档</li>
            <li class="list-inline-item">联系</li>
            <li class="list-inline-item">更多</li>
          </ul>
        </small>
      </footer>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, watch } from 'vue'
import { useStore } from 'vuex'
import 'bootstrap/dist/css/bootstrap.min.css'
import GlobalHeader from './components/GlobalHeader.vue'
import Loader from './components/Loader.vue'
import { GlobalDataProps } from './store'
import createMessage from './components/createMessage'

export default defineComponent({
  name: 'App',
  components: {
    GlobalHeader,
    Loader
  },
  setup() {
    const store = useStore<GlobalDataProps>()
    const currentUser = computed(() => store.state.user)
    const isLoading = computed(() => store.state.loading)
    const error = computed(() => store.state.error)
    watch(() => error.value.status, () => {
      const { status, message } = error.value
      if (status && message) {
        createMessage(message, 'error')
      }
    })
    return {
      currentUser,
      isLoading,
      error
    }
  }
})
</script>

<style scoped>
  .header-bg{
    background: #0d6efd;
  }
  .footer-bg{
    background: #f8f9fa;
  }
</style>
