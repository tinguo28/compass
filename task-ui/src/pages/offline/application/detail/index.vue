<script setup lang="ts">
import ItemWrapper from './components/ItemWrapper.vue'
import BaseInfo from './components/BaseInfo.vue'
import ErrorTable from './components/ErrorTable.vue'
import Chart from './components/AnalyzeChart.vue'
import CpuChart from './components/CpuChart.vue'
import { get } from '~/utils/request'
import { cloudTheme } from '~/utils/setting'
let detailInfo: any = $ref({
  runTimeAnalyze: [],
})
const route = useRoute()
const memoryData = computed(() => {
  return detailInfo?.resourcesAnalyze.find(item => item.type === 'memoryChart' && item.item)
})

const cpuData = computed(() => {
  return detailInfo?.resourcesAnalyze.find(item => item.type === 'cpuChart')
})
onMounted(async () => {
  const res = await Promise.all([
    get(`/api/v1/app/report/runError?applicationId=${route.query.applicationId}`),
    get(`/api/v1/app/report/runInfo?applicationId=${route.query.applicationId}`),
    get(`/api/v1/app/report/runResource?applicationId=${route.query.applicationId}`),
    get(`/api/v1/app/report/runTime?applicationId=${route.query.applicationId}`),
  ])
  detailInfo = {
    runErrorAnalyze: res[0].filter((item: any) => item.item),
    runInfo: res[1],
    resourcesAnalyze: res[2],
    runTimeAnalyze: res[3].filter((item: any) => item.item),
  }
})
</script>

<template>
  <el-card>
    <p class="detail-title">
      Application诊断报告详情
    </p>
    <div v-if="detailInfo.runInfo">
      <ItemWrapper title="诊断类型" class="m-b-5">
        <div>
          <span v-for="(item, index) in detailInfo.runInfo.taskInfo.categories" :key="item" class="category-card"
            :title="item" :style="{ 'border-left': `3px solid ${cloudTheme[index]}` }">
            {{ item }}
          </span>
        </div>
      </ItemWrapper>
      <BaseInfo :info="detailInfo.runInfo" />
      <ItemWrapper v-for="(item, index) in detailInfo.runErrorAnalyze" :key="item.name" :title="item.name"
        :conclusion="item.conclusion">
        <ErrorTable :data="item.item.table" :width-list="[120, 180, 180, '', 270]" />
      </ItemWrapper>
      <ItemWrapper v-if="memoryData && memoryData.item" :title="memoryData.name" :conclusion="memoryData.conclusion"
        :gc-info="memoryData.item?.computeNodeList">
        <Chart v-for="chartList in memoryData.item.chartList" :data="chartList" />
      </ItemWrapper>
      <div v-if="cpuData && cpuData.conclusion">
        <ItemWrapper :title="cpuData.name" :conclusion="cpuData.conclusion">
          <CpuChart :data="cpuData.item" />
        </ItemWrapper>
      </div>
      <ItemWrapper v-for="(item, index) in detailInfo.runTimeAnalyze" :key="item.name" :title="item.name"
        :conclusion="item.conclusion">
        <div v-if="item.type === 'chart'">
          <Chart v-for="(itemC, indexC) in item.item.chartList" :key="indexC" :index="`${index}-${indexC}`"
            :data="itemC" />
        </div>
        <div v-if="item.type === 'table'">
          <ErrorTable :data="item.item.table" />
        </div>
      </ItemWrapper>
    </div>
  </el-card>
</template>

<style lang="scss" scoped>
.detail-title {
  font-size: 22px;
  font-weight: bold;
  text-align: center;
  margin-top: 10px;
  margin-bottom: 30px;
}
</style>

<route lang="yaml">
meta:
  name: application
name: appDetail
</route>
