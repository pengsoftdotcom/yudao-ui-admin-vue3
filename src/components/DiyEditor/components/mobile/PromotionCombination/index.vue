<template>
  <el-scrollbar class="z-1 min-h-30px" wrap-class="w-full" ref="containerRef">
    <!-- 商品网格 -->
    <div
      class="grid overflow-x-auto"
      :style="{
        gridGap: `${property.space}px`,
        gridTemplateColumns,
        width: scrollbarWidth
      }"
    >
      <!-- 商品 -->
      <div
        class="relative box-content flex flex-row flex-wrap overflow-hidden bg-white"
        :style="{
          borderTopLeftRadius: `${property.borderRadiusTop}px`,
          borderTopRightRadius: `${property.borderRadiusTop}px`,
          borderBottomLeftRadius: `${property.borderRadiusBottom}px`,
          borderBottomRightRadius: `${property.borderRadiusBottom}px`
        }"
        v-for="(spu, index) in spuList"
        :key="index"
      >
        <!-- 角标 -->
        <div
          v-if="property.badge.show"
          class="absolute left-0 top-0 z-1 items-center justify-center"
        >
          <el-image fit="cover" :src="property.badge.imgUrl" class="h-26px w-38px" />
        </div>
        <!-- 商品封面图 -->
        <el-image fit="cover" :src="spu.picUrl" :style="{ width: imageSize, height: imageSize }" />
        <div
          :class="[
            'flex flex-col gap-8px p-8px box-border',
            {
              'w-[calc(100%-64px)]': columns === 2,
              'w-full': columns === 3
            }
          ]"
        >
          <!-- 商品名称 -->
          <div
            v-if="property.fields.name.show"
            class="truncate text-12px"
            :style="{ color: property.fields.name.color }"
          >
            {{ spu.name }}
          </div>
          <div>
            <!-- 商品价格 -->
            <span
              v-if="property.fields.price.show"
              class="text-12px"
              :style="{ color: property.fields.price.color }"
            >
              ￥{{ fenToYuan(spu.combinationPrice || spu.price || 0) }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </el-scrollbar>
</template>
<script setup lang="ts">
import { PromotionCombinationProperty } from './config'
import * as ProductSpuApi from '@/api/mall/product/spu'
import * as CombinationActivityApi from '@/api/mall/promotion/combination/combinationActivity'
import { Spu } from '@/api/mall/product/spu'
import { CombinationProductVO } from '@/api/mall/promotion/combination/combinationActivity'
import { fenToYuan } from '@/utils'

/** 拼团 */
defineOptions({ name: 'PromotionCombination' })
// 定义属性
const props = defineProps<{ property: PromotionCombinationProperty }>()
// 商品列表
const spuList = ref<ProductSpuApi.Spu[]>([])
watch(
  () => props.property.activityId,
  async () => {
    if (!props.property.activityId) return
    const activity = await CombinationActivityApi.getCombinationActivity(props.property.activityId)
    if (!activity?.spuId) return
    spuList.value = [await ProductSpuApi.getSpu(activity.spuId)]
    // 循环活动信息，赋值拼团价格
    activity.products.forEach((product: CombinationProductVO) => {
      spuList.value.forEach((spu: Spu) => {
        // 商品原售价和拼团价，哪个便宜就赋值哪个
        spu.combinationPrice = Math.min(spu.combinationPrice || Infinity, product.combinationPrice) // 设置 SPU 的最低价格
      })
    })
  },
  {
    immediate: true,
    deep: true
  }
)
// 手机宽度
const phoneWidth = ref(375)
// 容器
const containerRef = ref()
// 商品的列数
const columns = ref(2)
// 滚动条宽度
const scrollbarWidth = ref('100%')
// 商品图大小
const imageSize = ref('0')
// 商品网络列数
const gridTemplateColumns = ref('')
// 计算布局参数
watch(
  () => [props.property, phoneWidth, spuList.value.length],
  () => {
    // 计算列数
    columns.value = props.property.layoutType === 'oneCol' ? 1 : 3
    // 每列的宽度为：（总宽度 - 间距 * (列数 - 1)）/ 列数
    const productWidth =
      (phoneWidth.value - props.property.space * (columns.value - 1)) / columns.value
    // 商品图布局：2列时，左右布局 3列时，上下布局
    imageSize.value = columns.value === 2 ? '64px' : `${productWidth}px`
    // 指定列数
    gridTemplateColumns.value = `repeat(${columns.value}, auto)`
    // 不滚动
    scrollbarWidth.value = '100%'
  },
  { immediate: true, deep: true }
)
onMounted(() => {
  // 提取手机宽度
  phoneWidth.value = containerRef.value?.wrapRef?.offsetWidth || 375
})
</script>

<style scoped lang="scss"></style>
