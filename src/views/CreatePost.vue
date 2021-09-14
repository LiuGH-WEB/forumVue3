<template>
  <div class="create-post-page">
    <h4>{{isEiditMode ? '编辑文章' : '新建文章'}}</h4>
    <uploader
      :uploaded="uploadData"
      action="/upload"
      :beforeUpload="uploadCheck"
      @file-uploaded="handleFileUploaded"
      class="d-flex align-items-center justify-content-center bg-light text-secondary w-100 my-4"
    >
      <h2>点击上传头图（仅支持jpg和png格式图片）</h2>
      <template #loading>
        <div class="d-flex">
          <div class="spinner-border text-secondary"></div>
          <h2>正在上传</h2>
        </div>
      </template>
      <template #uploaded="dataProps">
        <img :src="dataProps.uploadedData.data.url">
      </template>
    </uploader>
    <h2></h2>
    <validate-form @form-submit="onFormSubmit">
      <div class="mb-3">
        <label class="form-label">文章标题</label>
        <validate-input
          type="text"
          placeholder="请输入文章标题"
          :rules="titleRules"
          v-model="titleVal"
        ></validate-input>
      </div>
      <div class="mb-3">
        <label class="form-label">文章详情</label>
        <validate-input
          rows="10"
          type="text"
          tag="textarea"
          placeholder="请输入文章标题"
          :rules="contentRules"
          v-model="contentVal"
        ></validate-input>
      </div>
      <template #submit>
        <button class="btn btn-primary btn-large">{{isEiditMode ? '更新文章' : '发表文章'}}</button>
      </template>
    </validate-form>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue'
import { useStore } from 'vuex'
import { GlobalDataProps, PostProps, ResponseType, ImageProps } from '../store'
import { useRouter, useRoute } from 'vue-router'
import ValidateInput, { RulesProp } from '../components/ValidateInput.vue'
import ValidateForm from '../components/ValidateForm.vue'
import Uploader from '../components/Uploader.vue'
import createMessage from '../components/createMessage'
import { beforeUploadCheck } from '../helper'

export default defineComponent({
  name: 'login',
  components: {
    ValidateInput,
    ValidateForm,
    Uploader
  },
  setup() {
    const uploadData = ref()
    const titleVal = ref('')
    const contentVal = ref('')
    /* 页面跳转 */
    const router = useRouter()
    /* 获取路由地址信息 */
    const route = useRoute()
    /* 将URL的queryID值转为boolean对象 */
    const isEiditMode = !!route.query.id
    const store = useStore<GlobalDataProps>()
    let imageId = ''
    const titleRules: RulesProp = [
      { type: 'required', message: '文章标题不能为空' }
    ]
    const contentRules = [
      { type: 'required', message: '文章详情不能为空' }
    ]
    onMounted(() => {
      if (isEiditMode) {
        store.dispatch('fetchPost', route.query.id).then((rawData: ResponseType<PostProps>) => {
          const currentPost = rawData.data
          if (rawData.data.image) {
            uploadData.value = { data: currentPost.image }
          }
          titleVal.value = currentPost.title
          contentVal.value = currentPost.content || ''
        })
      }
    })
    const handleFileUploaded = (rawData: ResponseType<ImageProps>) => {
      if (rawData.data._id) {
        imageId = rawData.data._id
      }
    }
    const onFormSubmit = (result: boolean) => {
      if (result) {
        const { column, _id } = store.state.user
        if (column) {
          const newPost: PostProps = {
            title: titleVal.value,
            content: contentVal.value,
            column,
            author: _id
          }
          if (imageId) {
            newPost.image = imageId
          }
          const actionName = isEiditMode ? 'updatePost' : 'createPost'
          const setData = isEiditMode ? {
            id: route.query.id,
            payload: newPost
          } : newPost
          store.dispatch(actionName, setData).then(() => {
            createMessage('发表成功，2秒后跳转到文章', 'success', 2000)
            setTimeout(() => {
              router.push({ name: 'column', params: { id: column } })
            }, 2000)
          })
        }
      }
    }
    const uploadCheck = (file: File) => {
      const result = beforeUploadCheck(file, { format: ['image/jpeg', 'image/png'], size: 1 })
      const { passed, error } = result
      if (error === 'format') {
        createMessage('上传图片只能是 JPG/PNG 格式', 'error')
      }
      if (error === 'size') {
        createMessage('上传图片大小不能超过 1Mb', 'error')
      }
      return passed
    }
    return {
      titleVal,
      contentVal,
      titleRules,
      contentRules,
      onFormSubmit,
      uploadCheck,
      handleFileUploaded,
      uploadData,
      isEiditMode
    }
  }
})
</script>

<style>
  .create-post-page .file-upload-container{
    height: 200px;
    cursor: pointer;
  }
  .create-post-page .file-upload-container img{
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
</style>
