<template>
  <div class="picture-search-form">
    <!-- 搜索表单 -->
    <a-form layout="inline" :model="searchParams" @finish="doSearch">
      <a-form-item label="关键词">
        <a-input
          v-model:value="searchParams.searchText"
          placeholder="从名称和简介搜索"
          allow-clear
        />
      </a-form-item>
      <a-form-item name="category" label="分类">
        <a-auto-complete
          style="min-width: 180px"
          v-model:value="searchParams.category"
          placeholder="请输入分类"
          :options="categoryOptions"
          allow-clear
        />
      </a-form-item>
      <a-form-item name="tags" label="标签">
        <a-select
          style="min-width: 180px"
          v-model:value="searchParams.tags"
          mode="tags"
          placeholder="请输入标签"
          :options="tagOptions"
          allow-clear
        />
      </a-form-item>
      <a-form-item label="日期" name="dateRange">
        <a-range-picker
          style="width: 400px"
          show-time
          v-model:value="dateRange"
          :placeholder="['编辑开始时间','编辑结束时间']"
          format="YYYY/MM/DD HH:mm:ss"
          :presets="rangePresets"
          @change="onRangeChange"
        />
      </a-form-item>
      <a-form-item label="名称" name="name">
        <a-input v-model:value="searchParams.name" placeholder="请输入名称" allow-clear/>
      </a-form-item>
      <a-form-item label="简介" name="introduction">
        <a-input v-model:value="searchParams.introduction" placeholder="请输入简介" allow-clear/>
      </a-form-item>
      <a-form-item label="宽度" name="picWidth">
        <a-input-number v-model:value="searchParams.picWidth"/>
      </a-form-item>
      <a-form-item label="高度" name="picHeight">
        <a-input-number v-model:value="searchParams.picHeight"/>
      </a-form-item>
      <a-form-item label="格式" name="picFormat">
        <a-input v-model:value="searchParams.picFormat" placeholder="请输入格式" allow-clear/>
      </a-form-item>
      <a-form-item>
        <a-space>
          <a-button type="primary" html-type="submit" style="width: 96px">搜索</a-button>
          <a-button html-type="reset" style="width: 96px" @click="doClear">重置</a-button>
        </a-space>
      </a-form-item>
    </a-form>
    <div style="margin-bottom: 16px"/>
  </div>
</template>
<script lang="ts" setup>
import {reactive, ref,onMounted} from 'vue'
import dayjs from "dayjs";
import {listPictureTagCategoryUsingGet} from "@/api/pictureController.ts";
import {message} from "ant-design-vue";

interface Props {
  onSearch?: (searchParams: API.PictureQueryRequest) => void
}

const props = defineProps<Props>()

// 搜索条件
const searchParams = reactive<API.PictureQueryRequest>({})

// 搜索数据
const doSearch = () => {
  props.onSearch(searchParams)
}

const dateRange = ref<[]>([])

const onRangeChange = (dates: any[], dateStrings: string[]) => {
  if (dates.length >= 2) {
    searchParams.startEditTime = undefined;
    searchParams.endEditTime = undefined
  }else {
    searchParams.startEditTime = dates[0].toDate();
    searchParams.endEditTime = dates[1].toDate()
  }
};

const rangePresets = ref([
  {label: 'Last 7 Days', value: [dayjs().add(-7, 'd'), dayjs()]},
  {label: 'Last 14 Days', value: [dayjs().add(-14, 'd'), dayjs()]},
  {label: 'Last 30 Days', value: [dayjs().add(-30, 'd'), dayjs()]},
  {label: 'Last 90 Days', value: [dayjs().add(-90, 'd'), dayjs()]},
]);


const categoryOptions = ref<string[]>([])
const tagOptions = ref<string[]>([])

/**
 * 获取标签和分类选项
 * @param values
 */
const getTagCategoryOptions = async () => {
  const res = await listPictureTagCategoryUsingGet()
  if (res.data.code === 0 && res.data.data) {
    tagOptions.value = (res.data.data.tagList ?? []).map((data: string) => {
      return {
        value: data,
        label: data,
      }
    })
    categoryOptions.value = (res.data.data.categoryList ?? []).map((data: string) => {
      return {
        value: data,
        label: data,
      }
    })
  } else {
    message.error('获取标签分类列表失败，' + res.data.message)
  }
}

onMounted(() => {
  getTagCategoryOptions()
})

const doClear = () => {
  dateRange.value = []
  Object.keys(searchParams).forEach((key) => {
    searchParams[key] = undefined
  })
  props.onSearch?.(searchParams)
}


</script>

<style scoped>

.picture-search-form .ant-form-item {
  margin-top: 16px;
}
</style>
