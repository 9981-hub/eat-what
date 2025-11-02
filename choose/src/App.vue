<template>
  <div class="container">
    <div ref="tempContainerRef" class="food-tag-container"></div>
    <div class="meal-tip" v-if="showMealTip">点击可以切换饭点~</div>
    <h1 class="change" @click="shakeText" :class="{ shaking: isShaking }">
      今天<span style="cursor: pointer" @click="switchMeal">{{
        currentMeal
      }}</span
      >吃<span style="font-size: 1.6vw; color: #111111; font-weight: bold">{{
        currentDish
      }}</span
      >{{ punctuation }}
    </h1>
    <button class="btn" @click="handleClick">{{ btnText }}</button>
    <div
      v-if="showComment && currentMode === 'abnormal'"
      class="random-comment"
    >
      {{ currentComment }}
    </div>
    <div class="mode-switch" v-show="showModeBar">
      <button
        class="mode-btn"
        style="cursor: pointer"
        @click="switchMode('normal')"
        :class="{
          active: currentMode === 'normal',
          'normal-mode': currentMode === 'normal',
        }"
      >
        正常人类
      </button>
      <button
        class="mode-btn"
        style="cursor: pointer"
        @click="switchMode('abnormal')"
        :class="{
          active: currentMode === 'abnormal',
          'abnormal-mode': currentMode === 'abnormal',
        }"
      >
        非正常人类
      </button>
    </div>
    <div class="tip-container" :class="{ show: showTip }">{{ tipMessage }}</div>
    <div class="modal-overlay" v-if="showModal" @click.self="closeModal">
      <div class="modal-content">
        <p>{{ modalMessage }}</p>
        <button class="modal-button" @click="closeModal">确定</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, defineProps, onMounted, onBeforeUnmount, computed } from "vue";
import { defaultMealDishes, fFoods, randomComments, abnormalMealDishes} from "./data";
import "./style.css";

//状态变量
const currentDish = ref("神马");
const punctuation = ref("?");
const btnText = ref("开始");
const isShaking = ref(false);
const showMealTip = ref(true);
const showModeBar = ref(true);
const showComment = ref(false);
const showModal = ref(false);
const modalMessage = ref("");
const currentComment = ref<string | undefined>("");
const isAnimating = ref(false);
const currentMode = ref("normal");
const clickTimestamps = ref<number[]>([]);

//快速点击的提示信息
const showTip = ref(false);
const tipMessage = ref("");
const QUICK_CLICK_INTERVAL = 5000; //5秒内
const CHECK_POINTS = [2, 5, 10, 20, 30];
const MESSAGES = [
  "我就知道你会换一个😀",
  "说，你是不是天秤座?😱",
  "你是吃了炫迈吗？😥",
  "难道你是处女座？🤖",
  "在换我可要报警了🐷！",
];

//定时器相关变量
let autoChangeTimer: number | null = null;
let tipTimer: number | null = null;
let dishInterval: number | null = null;

//背景菜名动画效果相关变量
const tempContainerRef = ref<HTMLElement>();
const foodTagsIntervalRef = ref<number | null>();
const tagInterval = 100;

//获取当前背景菜名
function createFoodTag() {
  const dishes = getCurrentTimeDishes();
  const tag = document.createElement("div");
  tag.className = "food_tag";
  tag.textContent = dishes[Math.floor(Math.random() * dishes.length)]!;
  tag.style.left = `${Math.random() * 100}%`;
  tag.style.top = `${Math.random() * 100}%`;
  tempContainerRef.value?.appendChild(tag);
  setTimeout(() => {
    tag.remove();
  }, 1500);
}

//文字框抖动
const shakeText = () => {
  if (isShaking.value) return;

  isShaking.value = true;

  setTimeout(() => {
    isShaking.value = false;
  }, 500);
};

//根据当前时间自动显示早，中，晚饭
function getCurrentTimeDishes() {
  const now = new Date().getHours();
  let index = 0;
  if (now >= 9 && now < 13) {
    index = 1;
  } else if (now >= 13) {
    index = 2;
  }
  return currentMealDishes.value[index]!.dishes;
}
//切换早，午，晚饭
const switchMeal = () => {
  currentMealIndex.value =
    (currentMealIndex.value + 1) % currentMealDishes.value.length;
  currentMeal.value = currentMealDishes.value[currentMealIndex.value]!.name;
};
const currentMealIndex = ref(0);

//切换模式栏
const switchMode = (mode: string) => {
  currentMode.value = mode;
  if (mode === "abnormal") {
    modalMessage.value = "注意！前方高能！";
    showModal.value = true;
  } else if (mode === "normal") {
    modalMessage.value = "还是人类好吃呢";
    showModal.value = true;
  }
};
const closeModal = () => {
  showModal.value = false;
};

//非正常模式下的随机评论
const stopRandom = () => {
  if (currentMode.value === "abnormal") {
    const randomIndex = Math.floor(Math.random() * randomComments.length);
    currentComment.value = randomComments[randomIndex];
    showComment.value = true;
  }
};

// //定义用户传参，接受用户自定义的分类数据
const props = defineProps<{
  customMealDishes?: Array<{
    name: string;
    dishes: string[];
  }>;
}>();
//计算属性：根据当前模式动态返回餐点数据
const currentMealDishes = computed(() => {
  return props.customMealDishes || defaultMealDishes;
});
const currentMeal = ref(currentMealDishes.value[0]!.name);
//点击切换菜名逻辑
const handleClick = () => {
  const now = Date.now();
  clickTimestamps.value.push(now);
  clickTimestamps.value = clickTimestamps.value.filter(
    (timestamp: number) => now - timestamp < QUICK_CLICK_INTERVAL
  );
  const currentCount = clickTimestamps.value.length;
  const index = CHECK_POINTS.findIndex((point) => point === currentCount);
  if (index !== -1) {
  clearTimeout(tipTimer!);
  tipMessage.value = MESSAGES[index]!;
  showTip.value = true;
  tipTimer = setTimeout(() => {
    showTip.value = false;
    setTimeout(() => tipMessage.value = "", 300);
  }, 3000);
}
//安全清除定时器
const clearIntervalIfExists = (interval: number | null | undefined) : number | null=> {
  if (interval !== null && interval !== undefined) {
    clearInterval(interval);
    return null;
  }
  return null;
};
// 功能按钮逻辑
const isStarting = btnText.value === "开始" || btnText.value === "换一个";
if (isStarting) {
  btnText.value = "停";
  punctuation.value = "?";
  showModeBar.value = false;
  autoChangeTimer = setInterval(randomCurrentMealDish, 100);
  isAnimating.value = true;
  foodTagsIntervalRef.value = setInterval(createFoodTag, tagInterval);
} else {
  btnText.value = "换一个";
  punctuation.value = "!";
  showModeBar.value = true;
  stopRandom();
  autoChangeTimer = clearIntervalIfExists(autoChangeTimer);
  isAnimating.value = false;
  foodTagsIntervalRef.value = clearIntervalIfExists(foodTagsIntervalRef.value);
}
};
//随机获取当前餐点的菜名
const randomCurrentMealDish = () => {
  const currentDishes = currentMealDishes.value[currentMealIndex.value]!.dishes;
  const randomIndex = Math.floor(Math.random() * currentDishes!.length);
  currentDish.value = currentDishes[randomIndex]!;
};

//在组件卸载前清除定时器
onBeforeUnmount(() => {
  if (autoChangeTimer !== null) {
    clearInterval(autoChangeTimer);
  }
  if (dishInterval !== null) {
    clearInterval(dishInterval);
  }
  if (tipTimer !== null) {
    clearTimeout(tipTimer);
  }
  if (clearClickTimer !== null) {
    clearInterval(clearClickTimer);
  }
  if (foodTagsIntervalRef.value !== null) {
    clearInterval(foodTagsIntervalRef.value);
  }
});

//在组件挂载时添加一个定时器定期清理
let clearClickTimer: number | null = null;

//自动切换到初始页面（页面加载完成后执行）
onMounted(() => {
  setTimeout(() => {
    showMealTip.value = false;
  }, 3000);

  const now = new Date().getHours();
  if (now < 9 || now >= 23) {
    currentMealIndex.value = 0;
  } else if (now < 13) {
    currentMealIndex.value = 1;
  } else {
    currentMealIndex.value = 2;
  }
  currentMeal.value = currentMealDishes.value[currentMealIndex.value]!.name;
  clearClickTimer = setInterval(() => {
    const now = Date.now();
    if (
      clickTimestamps.value.length > 0 &&
      now - clickTimestamps.value[clickTimestamps.value.length - 1]! > 3000
    ) {
      clickTimestamps.value = [];
    }
  }, 1000);
});
</script>
<style></style>
