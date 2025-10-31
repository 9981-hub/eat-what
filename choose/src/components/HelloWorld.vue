<template>
  <div class="container">
    <div class="dish-container">
      <div
        v-for="dish in backgroundDishes"
        :key="dish.id"
        class="background-dish"
        :style="{
          left: dish.x + '%',
          top: dish.y + '%',
          fontSize: dish.size + 'px',
          opacity: dish.opacity
        }"
      >
        {{ dish.name }}
      </div>
    </div>
    <div class="meal-tip" v-if="showMealTip">点击可以切换饭点~</div>
    <h1 class="change" @click="shakeText" :class="{'shaking':isShaking}">
 今天<span  style="cursor: pointer" @click="switchMeal">{{ currentMeal }}</span>吃<span style="font-size: 1.6vw; color: #111111; font-weight: bold;">{{ currentDish }}</span>{{ punctuation }}
    </h1>
    <button class="btn" @click="handleClick">{{btnText}}</button>
    <div v-if="showComment && currentMode === 'abnormal'" class="random-comment">{{ currentComment }}</div>
    <div class="mode-switch" v-show="showModeBar">
      <button
        class="mode-btn"
        style="cursor: pointer"
        @click="switchMode('normal')"
        :class="{active: currentMode === 'normal', 'normal-mode':currentMode === 'normal'}"
      >正常人类</button>
      <button
        class="mode-btn"
        style="cursor: pointer"
        @click="switchMode('abnormal')"
        :class="{active: currentMode === 'abnormal', 'abnormal-mode':currentMode === 'abnormal'}"
      >非正常人类</button>
    </div>
      <div class="tip-container" :class="{show:showTip}">{{ tipMessage }}</div>
      <div class="modal-overlay" v-if="showModal" @click.self="closeModal">
        <div class="modal-content">
          <p>{{ modalMessage }}</p>
          <button class="modal-button" @click="closeModal">确定</button>
        </div>
      </div>
  </div>
</template>
<script setup lang="ts">
import {ref, defineProps, onMounted, onBeforeUnmount, initCustomFormatter, computed} from 'vue';

const currentComment = ref<string | undefined>('');
const showComment = ref(false);

const randomComments = [
'大哥，饶命啊大哥', '吃吃吃，就知道吃', '壮士，干了这碗热翔', '就这，还不够我塞牙缝儿', '莫慌，抱紧我', '吃一个，长一斤', '你帅你先吃 你胖你先吃', '听说吃这玩意吃不胖 你先吃','我不饿 不吃不是中国人', '配上鸡汤，口味更佳', '我仿佛看到了盐水瓶', '嗯，好吃么？', '饭后注意漱口哦', '这菜红烧味道如何', '饭后百步走，活到九十九', '分享页面到朋友圈，可以获得30个QQ太阳', '据说吃完99%都哭了', '惊天内幕！这网页是逗你玩的', '为了身边的朋友！！转！！！！', '我也是醉了', '我想静静，不要问我静静是谁', '解决吃什么难题哪家强？',' 我就笑笑不说话 转发过100'
];

const stopRandom = () => {
  if (currentMode.value === 'abnormal') {
    const randomIndex = Math.floor(Math.random() * randomComments.length);
    currentComment.value = randomComments[randomIndex];
    showComment.value = true;
  }
}

interface BackgroundDish {
  id: number;
  name: string;
  x: number;
  y: number;
  size: number;
  opacity: number;
}

const currentMode = ref('normal');

const showMealTip = ref(true);

const isShaking = ref(false);

const backgroundDishes = ref<BackgroundDish[]>([]);

const currentDish = ref('神马');

const punctuation = ref('?');

const showModeBar = ref(true);

const btnText = ref('开始');
let autoChangeTimer: number | null = null;//定时器

const isAnimating = ref(false);
let dishInterval: number | null = null;

const showModal = ref(false);
const modalMessage = ref('');

const closeModal = () => {
  showModal.value = false;
};



const clickTimestamps = ref<number[]>([]);
const showTip = ref(false);
const tipMessage = ref('');
let tipTimer: number | null = null;
const QUICK_CLICK_INTERVAL = 5000;//5秒内
const CHECK_POINTS = [2,5,10,20,30];
const MESSAGES = [
    '我就知道你会换一个😀',
    '说，你是不是天秤座?😱',
    '你是吃了炫迈吗？😥',
    '难道你是处女座？🤖',
    '在换我可要报警了🐷！'
];

const shakeText = () => {
  if (isShaking.value) return ;

  isShaking.value = true;

  setTimeout(() => {
    isShaking.value = false;
  },500);
};

//定义用户传参，接受用户自定义的早午晚饭分类数据
const props = defineProps<{
  //用户传参格式示例：[{name:'早饭',dishes:['豆浆','油条']},...]
  customMealDishes?:Array<{
    name:string;
    dishes: string[];
  }>;
}>();

//默认餐点分类数据（用户没传参时用这个）
const defaultMealDishes = [
  {name:'早饭',dishes: [ '面包','蛋糕', '荷包蛋', '烧饼', '饽', '肉夹馍', '油条', '馄饨', '火腿', '面条', '小笼包', '玉米粥', '肉包', '煎饼果子', '饺子', '煎蛋', '烧卖', '生煎', '锅贴', '包子', '酸奶', '苹果', '梨', '香蕉', '皮蛋瘦肉粥', '蛋挞', '南瓜粥', '煎饼', '玉米糊', '泡面', '粥', '馒头', '燕麦片', '水煮蛋', '米粉', '豆浆', '牛奶', '花卷', '豆腐脑', '煎饼果子', '小米粥', '黑米糕', '鸡蛋饼', '牛奶布丁', '水果沙拉', '鸡蛋羹', '南瓜馅饼', '鸡蛋灌饼', '奶香小馒头', '汉堡包', '披萨', '八宝粥', '三明治', '蛋包饭', '豆沙红薯饼', '驴肉火烧', '粥', '粢饭糕', '蒸饺', '白粥']},
  {name:'午饭',dishes:[ '麻辣烫','面条', '卤面', '麻辣香锅', '麻辣拌', '米线', '砂锅', '香辣面', '香辣蟹柳拌饭', '淮南牛肉汤', '螺狮粉', '炝锅面', '盖浇饭', '鸡排饭', '东北盒饭', '西红柿鸡蛋拌饭', '鸡丁米线', '地三鲜', '糖醋排骨饭', '辣椒炒肉拌面', '红烧茄子盖饭', '锅包肉', '汉堡', '披萨', '水煮鱼', '炒鸡', '炒虾仁', '鱼面', '金汤鱼粉', '烧鹅', '烤鸭', '家常豆腐', '四喜丸子', '肉末凉粉', '豆角茄子', '烤鱼', '火锅']},
  {name:'晚饭',dishes:['盖浇饭', '砂锅', '大排档', '米线', '满汉全席', '西餐', '麻辣烫', '自助餐', '炒面', '快餐', '水果', '西北风', '馄饨', '火锅', '烧烤', '泡面', '水饺', '日本料理', '涮羊肉', '味千拉面', '面包', '扬州炒饭', '自助餐', '菜饭骨头汤', '茶餐厅', '海底捞', '西贝莜面村', '披萨', '麦当劳', 'KFC', '汉堡王', '卡乐星', '兰州拉面', '沙县小吃', '烤鱼', '烤肉', '海鲜', '铁板烧', '韩国料理', '粥', '快餐', '萨莉亚', '桂林米粉', '东南亚菜', '甜点', '农家菜', '川菜', '粤菜', '湘菜', '本帮菜', '生活', '全家便当']}
]

const fFoods = [
  '时空扭曲汉堡','量子态披萨','黑洞炒饭','虫洞意面','星际能量棒','超新星烧烤','平行宇宙火锅','多维空间寿司','反物质甜点','中子星烤肉','宇宙射线饮料','暗物质冰激凌','冰箱','书桌','电扇','空调','鼠标','键盘','电视','台灯','手机','餐巾纸','椅子','窗户','纸箱','别针','毛线','假发','发箍']

  const abnormalMealDishes = [
    {name: '早饭', dishes: fFoods},
    {name: '午饭', dishes: fFoods},
    {name: '晚饭', dishes: fFoods}
  ]

  const getCurrentModeDishes = () => {
    return currentMode.value === 'normal' ? (props.customMealDishes || defaultMealDishes) : abnormalMealDishes;
  };

  //计算属性：根据当前模式动态返回餐点数据
  const currentMealDishes = computed(() => {
    return currentMode.value === 'normal' ? (props.customMealDishes || defaultMealDishes) : abnormalMealDishes;
  });

  const currentMeal = ref(currentMealDishes.value[0]!.name);

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

const switchMeal =() => {
  currentMealIndex.value = (currentMealIndex.value + 1) % currentMealDishes.value.length;
  currentMeal.value = currentMealDishes.value[currentMealIndex.value]!.name;
};

// 响应式变量
const currentMealIndex = ref(0);


//切换菜名逻辑
const handleClick = () => {
  const now = Date.now();
  //添加当前点击时间戳
  clickTimestamps.value.push(now);
  //移除超过时间窗口的点击记录
  clickTimestamps.value = clickTimestamps.value.filter(timestamp => now - timestamp < QUICK_CLICK_INTERVAL);
  //检查快速点击次数是否达到触发点
  const currentCount = clickTimestamps.value.length;
  const index = CHECK_POINTS.findIndex(point => point === currentCount);

  if (index !== -1) {
    //显示提示
    tipMessage.value = MESSAGES[index]!;
    showTip.value = true;
    //清除之前的定时器
    if (tipTimer) {
      clearTimeout(tipTimer);
    }
    //设置提示内容并显示
    tipMessage.value = MESSAGES[index]!;
    showTip.value = true;

    //设置3秒后隐藏提示
    tipTimer = setTimeout(() => {
      showTip.value = false;
      setTimeout(() => {
        tipMessage.value = '';
      }, 300);
    },3000);
  }

  if (btnText.value === '开始' || btnText.value === '换一个') {
    btnText.value = '停';
    punctuation.value = '?';
    showModeBar.value = false;
    autoChangeTimer = setInterval(() => {
      randomCurrentMealDish();
    }, 100);
    //启动背景动画
    isAnimating.value = true;
    dishInterval = setInterval(addRandomDish, 100);
  } else if (btnText.value === '停') {
    btnText.value = '换一个';
    punctuation.value = '!';
    showModeBar.value = true;
    stopRandom();
    //清除定时器，停止切换
    if (autoChangeTimer !== null) {
      clearInterval(autoChangeTimer);
      autoChangeTimer = null;
    }

    //停止背景动画
    isAnimating.value = false;
    if (dishInterval !== null) {
      clearInterval(dishInterval);
      dishInterval = null;
    }
    
    //让现有菜名统一淡出，而不是直接清空
    const fadeOutTimer = setInterval(() => {
      if (backgroundDishes.value.length > 0) {
        backgroundDishes.value.forEach((dish,index) => {
          if (backgroundDishes.value[index]) {
            backgroundDishes.value[index]!.opacity -= 0.05;

            //当某个菜名透明度降到0时移除它
            if (backgroundDishes.value[index]!.opacity <= 0) {
              backgroundDishes.value.splice(index,1);
            }
          }
        });
      } else {
        //所有菜名都消失后清除定时器
        clearInterval(fadeOutTimer);
      }
    }, 30);
  } 
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
});

//随机获取当前餐点的菜名
const randomCurrentMealDish = () => {
  const currentDishes = currentMealDishes.value[currentMealIndex.value]!.dishes;
  const randomIndex = Math.floor(Math.random()*currentDishes!.length);
  currentDish.value = currentDishes[randomIndex]!;
};

//添加随机菜名动画方法
function addRandomDish() {
  const dishes = getCurrentTimeDishes();
  const randomDish = dishes[Math.floor(Math.random() * dishes.length)]!;
  const dish = {
    id: Date.now() + Math.random(),
    name: randomDish,
    x: Math.random() * 80 +  10,
    y: Math.random() * 80 + 10,
    size: 25,
    opacity: 0.7
  };

  backgroundDishes.value.push(dish);

  //动画处理，加快速度
  const timer = setInterval(() => {
    const index = backgroundDishes.value.findIndex(d => d.id === dish.id);
    if (index !== -1 && backgroundDishes.value[index]) {
      backgroundDishes.value[index]!.size -= 0.1;
      backgroundDishes.value[index]!.opacity -= 0.01;

      if (backgroundDishes.value[index]!.opacity <= 0) {
        clearInterval(timer);
        backgroundDishes.value.splice(index,1);
      }
    }
  }, 30);
}

const switchMode = (mode: string) => {
  currentMode.value = mode;
  //重置餐点索引，确保在新模式中有有效的索引
  if (currentMealIndex.value >= currentMealDishes.value.length) {
    currentMealIndex.value = 0;
  }
  //更新当前餐点
  currentMeal.value = currentMealDishes.value
  [currentMealIndex.value]!.name;

  //如果当前正在展示菜品，也更新当前菜品
  if (btnText.value === '换一个') {
    randomCurrentMealDish();
  }

  //显示提示信息
  if (mode === 'abnormal') {
    modalMessage.value = '注意！前方高能！';
    showModal.value = true;
  } else if(mode === 'normal') {
    modalMessage.value = '还是人类好吃呢';
    showModal.value = true;
  }
};

//在组件挂载时添加一个定时器定期清理
let clearClickTimer:number | null = null;

//自动切换逻辑（页面加载完成后执行）
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
  //根据自动切换的索引更新餐点和菜名
  currentMeal.value = currentMealDishes.value[currentMealIndex.value]!.name;

  //定期清理长时间没有点击的记录（比如3秒后）
  clearClickTimer = setInterval(() => {
    const now = Date.now();
    if (clickTimestamps.value.length > 0 && now - clickTimestamps.value[clickTimestamps.value.length-1]! > 3000) {
      clickTimestamps.value = [];
    }
  }, 1000);
})
</script>
<style>
*{
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Microsoft YaHei', sans-serif;
}

html,body {
  height: 100%;
  width: 100%;
}

body {
  background-image: url('../assets/bg.jpg');
  background-repeat: repeat;
  background-size: auto;
}

.container {
  padding: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-image: url('/src/assets/bg.jpg');
  background-repeat: repeat;
  background-size: auto;
  background-position: 0 0;
  position: relative;
  z-index: 1;
  animation: scrollBackgroundUp 0.1s linear infinite !important;
  margin-top: -40px;
}

@keyframes scrollBackgroundUp {
  0% {background-position: 0 0;}
  100% { background-position: 0 -1000px; }
}

.change {
  position: relative;
  font-size: 1.6vw; 
  font-weight: normal; 
  color: #333333;
  z-index: 10;
}

.random-comment {
  font-size: 12px;
  color: #676565;
  margin-top: 50px;
}

.meal-tip {
  position: absolute;
  top: -60px;
  left: 50%;
  transform: translateX(-50%) rotate(0deg);
  background-color: #5D5D5D;
  color: white;
  padding: 8px 16px;
  border-radius: 15px;
  font-size: 14px;
  white-space: nowrap;
  animation: rotateIn 0.5s ease-out forwards;
}

@keyframes rotateIn {
  0% {
    transform: translateX(-50%) rotate(-30deg) scale(0.8);
    opacity: 0;
  }
  70% {
    transform: translateX(-50%) rotate(10deg) scale(1.1);
    opacity: 0.8;
  }
  100% {
    transform: translateX(-50%) rotate(0deg) scale(1);
    opacity: 1;
  }
}

.meal-tip::after {
  content: '';
  position: absolute;
  bottom: -6px;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 6px solid transparent;
  border-right: 6px solid transparent;
  border-top: 6px solid #5D5D5D;
  animation: fadeInArrow 0.3s ease-out 0.2s forwards;
  opacity: 0;
}

@keyframes fadeInArrow {
  0% {
    opacity: 0;
    transform: translateX(-50%) translateY(-5px);
  }
  100% {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

.shaking {
  animation: shake 0.5s ease-in-out;
}

@keyframes shake {
  0%, 100% {transform: translateX(0);}
  10%, 30%, 50%, 70%, 90% {transform: translateX(-5px);}
  20%, 40%, 60%, 80% {transform: translateX(5px);}
}

h1 {
  margin: 0px 0px 30px;
  letter-spacing: 0;
  word-spacing: 0;
}

h1 span {
  display: inline;
}

.btn {
  box-shadow: inset 0 2px 3px rgba(0, 0, 0, 0.07), 0 1px rgba(255, 255, 255, 0.5);
  cursor: pointer;
  font-size: 28px;
  color: rgb(255, 255, 255);
  background: linear-gradient(90deg, #FF7B2E, #FF9B23, #FF921E);
  border-radius: 40px;
  width: 170px;
  padding: 9px 15px;
  border: none;
  outline: none;
  box-shadow: rgba(0, 0, 0, 0.2) 0px 0px 20px;
  z-index: 10;
  border: 5px solid rgb(194, 190, 190);
  outline: none !important;
  box-shadow: none !important;
}

.btn:hover {
  background: #ffba30;
  border: 5px solid rgb(194, 190, 190);
  outline: none !important;
  box-shadow: none !important;
}

.mode-switch {
  position: fixed;
  bottom: 5%;
  left: 50%;
  transform: translateX(-50%);
  font-size: 0.6vw;
  border: 1px solid rgb(212, 206, 206);
  border-radius: 20px;
  background-color: #FFFFFF99;
}

.mode-switch button {
  border: none;
  margin: 1px;
  padding: 2px 7px;
  color: rgb(173, 166, 166);
  background-color: aliceblue;
}

.mode-btn {
  background-color: aliceblue;
}

.mode-btn.normal-mode {
background-color: #00bdd6;
color: #ffffff;
}

.mode-btn.abnormal-mode {
  background-color: #e80773;
  color: #ffffff;
}

/* 在现有样式后添加 */
.dish-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none; /* 不影响其他交互 */
  z-index: 0; /* 放在底层 */
}

.background-dish {
  position: absolute;
  color: rgba(0, 0, 0, 0.7); /* 半透明颜色 */
  white-space: nowrap;
  font-weight: bold;
  text-shadow: 1px 1px 2px rgba(255, 255, 255, 0.5);
  z-index: 1;
}

@media (max-width:480px) {
  .btn {font-size: 24px;}
  .change {font-size: 24px;}
}

.tip-container {
  position: fixed;
  bottom: 80px;
  left: 50%;
  transform: translateX(-50%);
  padding: 2px 6px;
  background-color: rgba(200, 200, 200, 0.8);
  color: #333;
  border-radius: 8px;
  z-index: 1000;
  transition: opacity 0.3s ease-in-out;
  opacity: 0;
}

.tip-container.show {
  opacity: 1;
}

.tip-message {
  background-color: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 10px 20px;
  border-radius: 20px;
  font-size: 14px;
  animation: fadeInOut 3s ease-in-out;
}

@keyframes fadeInOut {
  0% { opacity: 0; transform: translateY(20px); }
  10%, 90% { opacity: 1; transform: translateY(0); }
  100% { opacity: 0; transform: translateY(-20px); }
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: flex-start;
  justify-content: center;
  z-index: 2000;
  padding-top: 10px;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
  width: 400px;
  max-width: 90%;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.modal-content p {
  margin: 0 0 20px 0;
  font-size: 12px;
  text-align: left;
  align-self: flex-start;
  color: #333;
  display: flex;
}

.modal-button {
  background-color: #2169EB;
  color: white;
  border: none;
  padding: 6px 15px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  transition: background-color 0.3s;
  align-self: flex-end;
  margin-left: 300px;
  margin-bottom: -10px;
}

.modal-button:hover {
  background-color: #1D5DD1;
}
</style>