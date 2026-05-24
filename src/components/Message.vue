<template>
  <!-- 基本信息 -->
  <div class="message">
    <!-- Logo -->
    <div class="logo">
      <img class="logo-img" :src="siteLogo" alt="logo" />
      <div :class="{ name: true, 'text-hidden': true, long: siteUrl[0].length >= 6 }">
        <span class="bg regular">{{ siteUrl[0] }}</span>
        <span class="sm regular">{{ siteUrl[1] }}</span>
      </div>
    </div>
    <!-- 简介 -->
    <div class="description cards" @click="changeBox">
      <div class="content">
        <Icon size="16">
          <QuoteLeft />
        </Icon>
        <Transition name="fade" mode="out-in">
          <div :key="descriptionText.hello + descriptionText.text" class="text">
            <p>{{ descriptionText.hello }}</p>
            <p>{{ descriptionText.text }}</p>
          </div>
        </Transition>
        <Icon size="16">
          <QuoteRight />
        </Icon>
      </div>
    </div>
  </div>
</template>

<script setup>
import { Icon } from "@vicons/utils";
import { QuoteLeft, QuoteRight } from "@vicons/fa";
import { Error } from "@icon-park/vue-next";
import { mainStore } from "@/store";
import { computed, reactive, watch, h } from "vue";
import { ElMessage } from "element-plus";
const store = mainStore();

// 主页站点logo
const siteLogo = import.meta.env.VITE_SITE_MAIN_LOGO;
// 站点链接
const siteUrl = computed(() => {
  const url = import.meta.env.VITE_SITE_URL;
  if (!url) return "草帽是赖".split("");
  if (url.startsWith("http://") || url.startsWith("https://")) {
    const urlFormat = url.replace(/^(https?:\/\/)/, "");
    return urlFormat.split(".");
  }
  return url.split(".");
});

// 简介区域文字
const descriptionText = reactive({
  hello: import.meta.env.VITE_DESC_HELLO,
  text: import.meta.env.VITE_DESC_TEXT,
});

// 切换右侧功能区
const changeBox = () => {
  if (store.getInnerWidth >= 721) {
    store.boxOpenState = !store.boxOpenState;
  } else {
    ElMessage({
      message: "当前页面宽度不足以开启盒子",
      grouping: true,
      icon: h(Error, {
        theme: "filled",
        fill: "#efefef",
      }),
    });
  }
};

// 监听状态变化
watch(
  () => store.boxOpenState,
  (value) => {
    if (value) {
      descriptionText.hello = import.meta.env.VITE_DESC_HELLO_OTHER;
      descriptionText.text = import.meta.env.VITE_DESC_TEXT_OTHER;
    } else {
      descriptionText.hello = import.meta.env.VITE_DESC_HELLO;
      descriptionText.text = import.meta.env.VITE_DESC_TEXT;
    }
  },
);
</script>

<style lang="scss" scoped>
// 引入全局正体字体
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;500;700&display=swap');

.message {
  .logo {
    display: flex;
    flex-direction: row;
    align-items: center;
    animation: fade 0.5s;
    max-width: 460px;
    .logo-img {
      border-radius: 50%;
      width: 120px;
    }
    .name {
      width: 100%;
      padding-left: 22px;
      transform: translateY(-8px);
      // 强制标准正体（思源黑体）
      font-family: 'Noto Sans SC', sans-serif;
      // 固定宽度边界，超过自动缩小
      max-width: 280px;
      display: flex;
      align-items: baseline;
      flex-wrap: wrap; // 关键：中文自动换行，彻底解决裁切
      gap: 6px;

      .regular {
        font-family: 'Noto Sans SC', sans-serif;
        font-weight: 500;
      }

      .bg {
        // 动态字体：宽度越小，字号越小
        font-size: clamp(2rem, 4vw, 4.5rem);
        white-space: normal; // 取消强制不换行
        word-break: break-all; // 中文自动换行
      }

      .sm {
        font-size: clamp(1rem, 1.8vw, 2rem);
        white-space: normal;
        @media (min-width: 721px) and (max-width: 789px) {
          display: none;
        }
      }
    }
    @media (max-width: 768px) {
      .logo-img {
        width: 100px;
      }
      .name {
        height: auto;
      }
    }

    @media (max-width: 720px) {
      max-width: 100%;
      .name {
        max-width: unset;
      }
    }
  }

  .description {
    padding: 1rem;
    margin-top: 3.5rem;
    max-width: 460px;
    animation: fade 0.5s;

    .content {
      display: flex;
      justify-content: space-between;

      .text {
        margin: 0.75rem 1rem;
        line-height: 2rem;
        margin-right: auto;
        transition: opacity 0.2s;

        p {
          &:nth-of-type(1) {
            font-family: "Pacifico-Regular";
          }
        }
      }

      .xicon:nth-of-type(2) {
        align-self: flex-end;
      }
    }
    @media (max-width: 720px) {
      max-width: 100%;
      pointer-events: none;
    }
  }
}
</style>
