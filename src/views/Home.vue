<template>
  <div class="home">
    <HelloWorld msg="图片上传组件demo"/>
    <upload-image></upload-image>
    
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import uploadImage from '@comp/upload-image'
export default {
  name: 'home',
  components: {
    HelloWorld,
    uploadImage
  }
}
</script>
