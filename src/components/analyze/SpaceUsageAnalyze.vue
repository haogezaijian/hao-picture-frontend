<template>
  <div class="space-user-analyze">
    <a-flex gap="middle">
      <a-card title="存储空间" style="width: 50%">
        <div style="height: 320px; text-align: center">
          <h3>
            {{ formatSize(data.usedSize) }} /
            {{ data.maxSize ? formatSize(data.maxSize) : '无限制' }}
          </h3>
          <a-progress type="dashboard" :percent="data.sizeUsageRatio ?? 0"/>
        </div>
      </a-card>
      <a-card title="图片数量" style="width: 50%">
        <h3>
          {{ data.usedCount }} /
          {{ data.maxCount ?? '无限制' }}
        </h3>
        <a-progress type="dashboard" :percent="data.sizeUsageRatio ?? 0"/>
      </a-card>

    </a-flex>
  </div>

</template>

<script setup lang="ts">

import VChart from "vue-echarts";
import "echarts"
import {onMounted, ref, watchEffect} from "vue";
import {getSpaceUsageAnalyzeUsingPost} from "@/api/spaceAnalyzeController.ts";
import {message} from "ant-design-vue";
import {formatSize} from "../../utils";

interface Props {
  queryAll?: boolean
  queryPublic?: boolean
  spaceId?: number
}

const props = withDefaults(defineProps<Props>(), {
  queryAll: false,
  queryPublic: false,
})

const data = ref<API.SpaceUsageAnalyzeResponse>({})

const loading = ref(true)

// 获取数据
const fetchData = async () => {
  loading.value = true

  const res = await getSpaceUsageAnalyzeUsingPost({
    queryAll: props.queryAll,
    queryPublic: props.queryPublic,
    spaceId: props.spaceId
  })
  if (res.data.code === 0 && res.data.data) {
    data.value = res.data.data
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
  loading.value = false
}

// 页面加载时获取数据，请求一次
watchEffect(() => {
  fetchData()
})


</script>

<style scoped>

</style>
