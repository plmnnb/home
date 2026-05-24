<template>
  <!-- 基本信息 -->
  <div class="message">
    <!-- Logo -->
    <div class="logo">
      <!-- 圆形头像容器 -->
      <div class="avatar-container">
        <img class="logo-img" :src="siteLogo" alt="logo" />
      </div>
      <!-- 自适应字体容器 -->
      <div class="name-wrapper">
        <div class="name-container" ref="nameContainerRef">
          <span class="bg" ref="bgTextRef">{{ siteUrl.main }}</span>
          <span v-if="siteUrl.suffix" class="sm">{{ siteUrl.suffix }}</span>
        </div>
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

const store = mainStore();

// 主页站点logo
const siteLogo = import.meta.env.VITE_SITE_MAIN_LOGO;

// 站点链接 - 修复中文域名问题
const siteUrl = computed(() => {
  const url = import.meta.env.VITE_SITE_URL;
  if (!url) return { main: "imsyy", suffix: "top" };
  
  let cleanUrl = url;
  if (url.startsWith("http://") || url.startsWith("https://")) {
    cleanUrl = url.replace(/^(https?:\/\/)/, "");
  }
  
  const parts = cleanUrl.split(".");
  
  if (parts.length === 1) {
    return { main: parts[0], suffix: null };
  }
  
  return { main: parts[0], suffix: parts.slice(1).join(".") };
});

// 简介区域文字
const descriptionText = reactive({
  hello: import.meta.env.VITE_DESC_HELLO,
  text: import.meta.env.VITE_DESC_TEXT,
});

// 自适应字体相关
const nameContainerRef = ref(null);
const bgTextRef = ref(null);
const resizeObserver = ref(null);
let adjustTimer = null;
let lastFontSize = null;

// 计算并调整字体大小 - 添加缓存防抖
const adjustFontSize = () => {
  if (!nameContainerRef.value || !bgTextRef.value) return;
  
  const container = nameContainerRef.value;
  const textEl = bgTextRef.value;
  
  const suffixExists = siteUrl.value.suffix;
  const containerMaxWidth = container.clientWidth - (suffixExists ? 60 : 20);
  
  let currentFontSize = 80;
  textEl.style.fontSize = `${currentFontSize}px`;
  
  let textWidth = textEl.scrollWidth;
  
  while (textWidth > containerMaxWidth && currentFontSize > 20) {
    currentFontSize -= 2;
    textEl.style.fontSize = `${currentFontSize}px`;
    textWidth = textEl.scrollWidth;
  }
  
  if (textWidth > containerMaxWidth) {
    currentFontSize = Math.max(20, (containerMaxWidth / textWidth) * currentFontSize);
    textEl.style.fontSize = `${currentFontSize}px`;
  }
  
  // 如果字体大小没变化，不触发重新计算
  if (lastFontSize === currentFontSize) return;
  lastFontSize = currentFontSize;
};

// 防抖版本的调整函数
const debouncedAdjustFontSize = () => {
  if (adjustTimer) clearTimeout(adjustTimer);
  adjustTimer = setTimeout(() => {
    adjustFontSize();
  }, 100);
};

// 使用 ResizeObserver 监听容器大小变化
const initResizeObserver = () => {
  if (!nameContainerRef.value) return;
  
  resizeObserver.value = new ResizeObserver(() => {
    debouncedAdjustFontSize();
  });
  
  resizeObserver.value.observe(nameContainerRef.value);
};

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
  }
);

// 监听站点 URL 变化
watch(
  () => siteUrl.value.main,
  () => {
    setTimeout(() => {
      adjustFontSize();
    }, 100);
  }
);

// 监听窗口大小变化 - 使用防抖
let resizeTimer = null;
const handleResize = () => {
  if (resizeTimer) clearTimeout(resizeTimer);
  resizeTimer = setTimeout(() => {
    adjustFontSize();
  }, 150);
};

onMounted(() => {
  setTimeout(() => {
    adjustFontSize();
    initResizeObserver();
  }, 100);
  
  window.addEventListener('resize', handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize);
  if (resizeTimer) clearTimeout(resizeTimer);
  if (adjustTimer) clearTimeout(adjustTimer);
  if (resizeObserver.value) {
    resizeObserver.value.disconnect();
  }
});
</script>

<style lang="scss" scoped>
.message {
  .logo {
    display: flex;
    flex-direction: row;
    align-items: center;
    animation: fade 0.5s;
    max-width: 460px;
    
    .avatar-container {
      width: 120px;
      height: 120px;
      flex-shrink: 0;
      border-radius: 50%;
      overflow: hidden;
      background-color: rgba(0, 0, 0, 0.2);
      
      .logo-img {
        width: 100%;
        height: 100%;
        object-fit: cover;
      }
    }
    
    .name-wrapper {
      flex: 1;
      min-width: 0;
      padding-left: 22px;
    }
    
    .name-container {
      width: 100%;
      display: flex;
      align-items: baseline;
      white-space: nowrap;
      
      .bg {
        font-size: 5rem;
        font-family: "HarmonyOS_Regular", sans-serif;
        display: inline-block;
        transition: font-size 0.1s ease;
        white-space: nowrap;
        max-width: calc(100% - 60px);
        overflow: visible;
      }
      
      .sm {
        margin-left: 6px;
        font-size: 2rem;
        flex-shrink: 0;
        
        @media (min-width: 721px) and (max-width: 789px) {
          display: none;
        }
      }
    }
    
    @media (max-width: 768px) {
      .avatar-container {
        width: 100px;
        height: 100px;
      }
      .name-container {
        .bg {
          font-size: 4.5rem;
        }
      }
    }
    
    @media (max-width: 720px) {
      max-width: 100%;
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
          font-family: "Pacifico-Regular", cursive;
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
