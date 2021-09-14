<template>
  <div class="post-detail-page">
    <modal
      title="删除文章" :visible="modalIsVisible"
      @modal-on-close="modalIsVisible = false"
      @modal-on-confirm="hideAndDete"
    >
      <p>你确定要删除这篇文章</p>
    </modal>
    <article class="w-75 mx-auto mb-5 pb-3" v-if="currentPost">
      <img :src="currentImageUrl" alt="currentPost.title" class="rounded-lg img-fluid my-4" v-if="currentImageUrl">
      <h2 class="mb-4">{{currentPost.title}}</h2>
      <div class="user-profile-component border-top border-bottom py-3 mb-5 align-items-center row g-0">
        <div class="col">
          <!-- 用户头像组件 -->
          <user-profile :user="currentPost.author" v-if="typeof currentPost.author === 'object'"></user-profile>
        </div>
        <!-- 作品创建时间 -->
        <span class="text-muted col text-right font-italic">发表于：{{currentPost.createdAt}}</span>
      </div>
      <!-- 内容解析markdown内容 -->
      <div v-html="currentHTML"></div>
      <div v-if="showEditArea" class="btn-group mt-5">
        <router-link :to="{name: 'create', query: { id: currentPost._id}}" class="btn btn-success">编辑</router-link>
        <button class="btn btn-danger" @click.prevent="modalIsVisible = true">删除</button>
      </div>
    </article>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, computed, ref } from 'vue'
import MarkdownIt from 'markdown-it'
import { useStore } from 'vuex'
import { useRoute, useRouter } from 'vue-router'
import { GlobalDataProps, PostProps, ImageProps, UserProps, ResponseType } from '../store'
import UserProfile from '../components/UserProfile.vue'
import Modal from '../components/Modal.vue'
import createMessage from '../components/createMessage'

export default defineComponent({
  name: 'post-detail',
  components: {
    UserProfile,
    Modal
  },
  setup() {
    const store = useStore<GlobalDataProps>()
    const route = useRoute()
    const router = useRouter()
    /* modal是否显示变量 */
    const modalIsVisible = ref(false)
    const currentId = route.params.id
    const md = new MarkdownIt()
    onMounted(() => {
      store.dispatch('fetchPost', currentId)
    })
    const currentPost = computed<PostProps>(() => store.getters.getCurrentPost(currentId))
    /* markdown文件解析 */
    const currentHTML = computed(() => {
      if (currentPost.value && currentPost.value.content) {
        const { isHTML, content } = currentPost.value
        return isHTML ? content : md.render(content)
      } else {
        return ''
      }
    })
    /* 当前用户登陆才能显示编辑和修改按钮 */
    const showEditArea = computed(() => {
      const { isLogin, _id } = store.state.user
      if (currentPost.value && currentPost.value.author && isLogin) {
        const postAuthor = currentPost.value.author as UserProps
        return postAuthor._id === _id
      } else {
        return false
      }
    })
    /* 图片修饰 */
    const currentImageUrl = computed(() => {
      if (currentPost.value && currentPost.value.image) {
        const { image } = currentPost.value
        return (image as ImageProps).url + '?x-oss-process=image/resize,w_850'
      } else {
        return null
      }
    })
    /* 删除提示框确认按钮事件 */
    const hideAndDete = () => {
      modalIsVisible.value = false
      store.dispatch('deletePost', currentId).then((rawdata: ResponseType<PostProps>) => {
        createMessage('删除成功，立即跳转到个人专栏首页', 'success', 1000)
        setTimeout(() => {
          router.push({ name: 'column', params: { id: rawdata.data.column } })
        }, 1050)
      })
    }
    return {
      currentPost,
      currentImageUrl,
      currentHTML,
      showEditArea,
      modalIsVisible,
      hideAndDete
    }
  }
})
</script>
