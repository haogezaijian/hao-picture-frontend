<template>
  <div id="mySpacePage">
    <p>请稍后</p>
  </div>
</template>

<script setup lang="ts">

import {useRouter} from "vue-router";
import {useLoginUserStore} from "@/stores/useLoginUserStore.ts";
import {message} from "ant-design-vue";
import {onMounted} from "vue";
import {listSpaceVoByPageUsingPost} from "@/api/spaceController.ts";

const router = useRouter()
const loginUserStore = useLoginUserStore();

const checkUserSpace = async () => {
  const loginUser = loginUserStore.loginUser

  if (!loginUser?.id) {
    router.replace('/user/login');
    return
  }

  const res = await listSpaceVoByPageUsingPost({
    userId: loginUser.id,
    current: 1,
    pageSize: 1,
  })

  if (res.data.code === 0) {
    if (res.data.data?.records?.length > 0) {
      const space = res.data.data.records[0]
      router.replace(`/space/${space.id}`)
    }else {
      router.replace('/add_space')
      message.warn('您还没有创建空间，请先创建空间')
    }
  }else {
    message.error('获取空间失败，'+res.data.message)
  }

}

onMounted(() => {
  checkUserSpace()
})

</script>

<style scoped>

</style>
