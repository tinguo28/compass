<script setup lang="ts">
// elementplus中文包
import zhCn from 'element-plus/lib/locale/lang/zh-cn'
import Default from '~/layouts/default.vue'
useHead({
  title: '罗盘',
})
const locale = $ref(zhCn)

onMounted(() => {
  document.body.style.setProperty('--el-color-primary', '#00bfbf')
  document.body.style.setProperty('--el-color-primary-light-9', '#eaf9f5')
  document.body.style.setProperty('--el-color-primary-light-3', '#03d5d5')
})
</script>

<template>
  <el-config-provider :locale="locale">
    <Default style="font-family: auto;" />
  </el-config-provider>
</template>

