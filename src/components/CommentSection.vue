<script setup lang="ts">
import { Edit } from "@element-plus/icons-vue"
import { useSubjectStore } from "@/stores/subjectStore"
import { storeToRefs } from "pinia"
import { getFullReviewAPI } from "@/axios/detailAPI.js"
import { ref, watch } from "vue"
import { useElementBounding } from "@vueuse/core"
import { useWindowSize } from "@vueuse/core"
import { useWindowScroll } from "@vueuse/core"

const { y } = useWindowScroll()
const { height } = useWindowSize()
const subjectStore = useSubjectStore()
const { videoInfo } = storeToRefs(subjectStore)
// 影评内容区
const filmReview = ref<HTMLElement[] | null>(null)
// 完整评价替换区域
const fullReview = ref<HTMLElement[] | null>(null)
// 影评总结
const summaryReview = ref<HTMLElement[] | null>(null)
// 收起按钮
const retract = ref<HTMLElement[] | null>(null)
// 互动区
const interactiveZone = ref<HTMLElement[] | null>(null)
// interface ElementBounding {
//   top: Ref<number>
// }
// let elementBounding: ElementBounding = {
//   top: ref<number>(0),
// }
const unwatchList = ref<Array<() => void>>([])
// 点击展开，获取完整评价
const getFullReview = async (id: string, index: number) => {
  const res = await getFullReviewAPI(id)
  summaryReview.value![index].style.display = "none"
  fullReview.value![index].innerHTML = res.data.body
    .replace("main-title-tip", "hidden")
    .replace("copyright", "copyright text-[#999] my-[20px]")
  fullReview.value![index].style.display = "block"
  retract.value![index].style.display = "block"
  const { top, bottom, left } = useElementBounding(filmReview.value![index])
  const unwatch = watch(
    () => top.value,
    () => {
      console.log(top.value, bottom.value, left.value)
      console.log(interactiveZone.value![index].style.position)
      if (bottom.value >= height.value && top.value <= height.value - 60) {
        interactiveZone.value![index].style.position = "fixed"
        interactiveZone.value![index].style.top = `${height.value - 81}px`
        interactiveZone.value![index].style.left = `${left.value}px`
        interactiveZone.value![index].classList.add("fixedStyle")
      } else {
        interactiveZone.value![index].style.position = "static"
        interactiveZone.value![index].classList.remove("fixedStyle")
      }
    },
    { immediate: true }
  )
  unwatchList.value[index] = unwatch
}
// 收起完整评价
const retractReview = (index: number) => {
  const { top } = useElementBounding(filmReview.value![index])
  y.value = y.value + top.value - 100
  summaryReview.value![index].style.display = "block"
  fullReview.value![index].style.display = "none"
  retract.value![index].style.display = "none"
  interactiveZone.value![index].style.position = "static"
  interactiveZone.value![index].classList.remove("fixedStyle")
  unwatchList.value[index]()
  unwatchList.value[index] = () => {}
}
// 点击展开
// 第一次调用函数
// 函数内包含三种情况：
// 1.top值大于等于视口高度减点赞区高度，点赞区在文档流
// 2.bottom值小于等于视口高度减点赞区高度，点赞区在文档流
// 3.其余情况，点赞区固定在视口底部。top值应为视口高度减点赞区高度，left值应为元素left值。
// 此后每次滚动都调用一次该函数。
</script>

<template>
  <!-- 短评 -->
  <div>
    <!-- 头部 -->
    <div class="h-[24px] mt-[16px] mb-[12px] flex items-center justify-between">
      <h2 class="text-[#007722] text-[16px]">
        {{ videoInfo.title }}的短评 · · · · · ·
        <span class="text-[13px] text-[#666]"
          >(
          <a href="" class="text-[#37A] hover:text-white hover:bg-[#37A]"
            >全部 {{ videoInfo.comments_count }} 条</a
          >
          )</span
        >
      </h2>
      <el-button
        :icon="Edit"
        style="
          height: 24px;
          color: #ca6445;
          width: 106px;
          font-size: 13px;
          border-radius: 2px;
        "
        color="#fae9da"
        >我要写短评</el-button
      >
    </div>
    <!-- 分页标签 -->
    <div class="pb-[15px] text-[13px] text-[#37A]">
      <a href="" class="text-[#111]">热门</a>
      /
      <a href="" class="hover:text-white hover:bg-[#37a]">最新</a>
      /
      <a href="" class="hover:text-white hover:bg-[#37a]">好友</a>
    </div>
    <!-- 内容 -->
    <ul>
      <li class="py-[14px] border-t-[1px]" v-for="item in 5">
        <!-- 第一行 -->
        <div class="text-[13px] flex items-center justify-between">
          <div class="flex items-center">
            <a
              href=""
              class="mr-[3px] text-[#37A] hover:text-white hover:bg-[#37a]"
              >kakakarl</a
            >
            看过
            <el-rate
              :model-value="4"
              disabled
              style="
                height: 11px;
                --el-rate-icon-margin: -1px;
                margin-left: 3px;
                --el-rate-icon-size: 14px;
              "
            />
            <span class="ml-[6px] text-[#aaa]">2023-07-21 21:31:57 湖南</span>
          </div>
          <div>
            212
            <span
              class="text-[#37a] hover:text-white hover:bg-[#37a] hover:cursor-pointer"
              >有用</span
            >
          </div>
        </div>
        <!-- 内容 -->
        <p class="text-[13px] text-[#666]">
          3.5，还蛮简单纯净的，还是很传统的“洞中一日，世上千年”的概念。在中国会成为神仙，在美国会去到末世，在日本谈了个恋爱。
        </p>
      </li>
    </ul>
    <div class="text-[13px]">
      >
      <a href="" class="text-[#37a] hover:text-white hover:bg-[#37a]"
        >更多短评 {{ videoInfo.comments_count }}条</a
      >
    </div>
  </div>

  <!-- 影评 -->
  <div class="mb-[25px]">
    <!-- 头部 -->
    <div class="h-[24px] mt-[16px] mb-[12px] flex items-center justify-between">
      <h2 class="text-[#007722] text-[16px]">
        {{ videoInfo.title }}的影评 · · · · · ·
        <span class="text-[13px] text-[#666]"
          >(
          <a href="" class="text-[#37A] hover:text-white hover:bg-[#37A]"
            >全部 {{ videoInfo.reviews_count }} 条</a
          >
          )</span
        >
      </h2>
      <el-button
        :icon="Edit"
        style="
          height: 24px;
          color: #ca6445;
          width: 106px;
          font-size: 13px;
          border-radius: 2px;
        "
        color="#fae9da"
        >我要写影评</el-button
      >
    </div>
    <!-- 分页标签 -->
    <div class="pb-[15px] text-[13px] text-[#37A]">
      <a href="" class="text-[#111]">热门</a>
      /
      <a href="" class="hover:text-white hover:bg-[#37a]">最新</a>
      /
      <a href="" class="hover:text-white hover:bg-[#37a]">好友</a>
    </div>
    <!-- 内容 -->
    <ul>
      <li
        class="py-[14px] border-t-[1px]"
        v-for="(item, index) in videoInfo.reviews.slice(0, 10)"
        ref="filmReview"
      >
        <!-- 第一行 -->
        <div class="text-[13px] flex items-center justify-start">
          <div class="flex items-center">
            <img
              class="w-[24px] h-[24px] rounded-full mr-[10px]"
              :src="item.author.avatar"
              alt=""
            />
            <a
              href=""
              class="mr-[3px] text-[#37A] hover:text-white hover:bg-[#37a]"
              >{{ item.author.name }}</a
            >
            <el-rate
              :model-value="item.rating.value"
              disabled
              style="
                height: 11px;
                --el-rate-icon-margin: -1px;
                margin-left: 3px;
                --el-rate-icon-size: 14px;
              "
            />
            <span class="ml-[6px] text-[#aaa]">{{ item.created_at }}</span>
          </div>
        </div>
        <!-- 内容 -->
        <div class="mt-[10px] text-[13px]">
          <!-- 标题 -->
          <h2 class="mb-[10px]">
            <a
              href=""
              class="text-[14px] text-[#37a] hover:text-white hover:bg-[#37a]"
              >{{ item.title }}</a
            >
          </h2>
          <!-- 剧透提醒 -->
          <div
            class="mb-[5px] pl-[8px] border-l-[1px] border-[#999] text-[#9b9b9b] h-[13px] leading-[13px]"
          >
            这篇影评可能有剧透
          </div>
          <!-- 影评总结 -->
          <p class="text-[#494949]" ref="summaryReview">
            {{ item.summary }}(<span
              class="text-[#37A] hover:text-white hover:bg-[#37a] hover:cursor-pointer"
              @click="getFullReview(item.id, index)"
              >展开</span
            >)
          </p>
          <div ref="fullReview" class="hidden"></div>
        </div>
        <!-- 互动区 -->
        <div
          class="flex mt-[18px] justify-between bg-white w-[675px]"
          ref="interactiveZone"
        >
          <div class="flex items-center">
            <el-button
              style="
                height: 21px;
                padding: 4px 6px;
                font-size: 12px;
                color: #37a;
                border: 0;
                border-radius: 2px;
              "
              color="#f0f7f9"
              >△ {{ item.useful_count }}</el-button
            >
            <el-button
              style="
                height: 21px;
                padding: 4px 6px;
                font-size: 12px;
                color: #37a;
                border: 0;
                border-radius: 2px;
              "
              color="#f0f7f9"
              >▽ {{ item.useless_count }}</el-button
            >
            <span
              class="text-[13px] text-[#37a] hover:text-white hover:bg-[#37a] ml-[10px] hover:cursor-pointer"
              >{{ item.comments_count }}回应</span
            >
          </div>
          <div
            class="hidden text-[13px] text-[#999] hover:cursor-pointer"
            ref="retract"
            @click="retractReview(index)"
          >
            收起
          </div>
        </div>
      </li>
    </ul>
    <div class="text-[13px]">
      >
      <a href="" class="text-[#37a] hover:text-white hover:bg-[#37a]"
        >更多影评 {{ videoInfo.reviews_count }}篇</a
      >
    </div>
  </div>
  <!-- 讨论区 -->
  <div>
    <div
      class="h-[36px] mt-[80px] pb-[12px] flex items-center justify-between border-b-[1px] border-dashed"
    >
      <h2 class="text-[#007722] text-[16px]">讨论区 · · · · · ·</h2>
      <el-button
        :icon="Edit"
        style="
          height: 24px;
          color: #ca6445;
          width: 106px;
          font-size: 13px;
          border-radius: 2px;
        "
        color="#fae9da"
        >添加新讨论</el-button
      >
    </div>
    <div
      class="py-[8px] text-[13px] text-[#666] flex border-b-[1px] border-dashed"
      v-for="item in 5"
    >
      <span class="w-[350px] pr-[3px]">
        <a href="" class="text-[#37a] hover:text-white hover:bg-[#37a] truncate"
          >有没有画风类似的番！</a
        >
      </span>
      <span class="pr-[3px] w-[141px]"
        >来自<a
          href=""
          class="text-[#37a] hover:text-white hover:bg-[#37a] truncate"
          >豆友wzrafz0wZU</a
        ></span
      >
      <span class="pr-[3px] w-[45px]">4 回应</span>
      <span>2024-09-09 00:53:21</span>
    </div>
    <div class="flex items-center justify-end my-[13px]">
      <a
        href=""
        class="text-[13px] text-[#37a] hover:text-white hover:bg-[#37a]"
      >
        > 去这部影片的讨论区（全部42条）</a
      >
    </div>
  </div>
</template>

<style scoped>
.fixedStyle {
  padding: 20px 0;
  border-top: 1px solid #ddd;
  border-bottom: 1px solid #ddd;
}
</style>
