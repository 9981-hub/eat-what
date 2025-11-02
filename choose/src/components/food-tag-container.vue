<template>
    <div ref="tempContainerRef" class="food-tag-container"></div>
  </template>
  
  <script setup lang="ts">
  import { ref, onMounted, onBeforeUnmount, watch } from 'vue'
  import { defaultMealDishes } from '../data'
  
  // Props
  const props = defineProps<{
    isAnimating: boolean
    tagInterval?: number
  }>()
  
  // 容器引用和定时器
  const tempContainerRef = ref<HTMLElement>()
  const foodTagsInterval = ref<number | null>(null)
  const maxTags = 20
  
  // 创建食物标签函数
  const createFoodTag = () => {
    if (!tempContainerRef.value) return
    
    const container = tempContainerRef.value
    const allFoods = defaultMealDishes.flatMap(cat => cat.items)
    
    // 限制最大标签数量
    if (container.children.length >= maxTags) {
      const firstChild = container.firstChild
      if (firstChild) container.removeChild(firstChild)
    }
    
    // 创建标签元素
    const tag = document.createElement('div')
    tag.className = 'food_tag'
    tag.textContent = allFoods[Math.floor(Math.random() * allFoods.length)]
    
    // 设置随机样式
    tag.style.left = `${Math.random() * 90}%`
    tag.style.top = `${Math.random() * 90}%`
    tag.style.opacity = '0'
    tag.style.fontSize = `${Math.floor(Math.random() * 10) + 14}px`
    
    container.appendChild(tag)
    
    // 动画效果
    setTimeout(() => {
      tag.style.opacity = '0.5'
    }, 10)
    
    // 自动移除
    setTimeout(() => {
      tag.style.opacity = '0'
      setTimeout(() => {
        if (container.contains(tag)) container.removeChild(tag)
      }, 300)
    }, 3000 + Math.random() * 2000)
  }
  
  // 监听动画状态变化
  watch(() => props.isAnimating, (newVal) => {
    if (newVal) {
      // 启动动画
      foodTagsInterval.value = setInterval(createFoodTag, props.tagInterval || 100)
    } else {
      // 停止动画
      if (foodTagsInterval.value !== null) {
        clearInterval(foodTagsInterval.value)
        foodTagsInterval.value = null
      }
      
      // 清空标签
      if (tempContainerRef.value) {
        tempContainerRef.value.innerHTML = ''
      }
    }
  })
  
  // 清理定时器
  onBeforeUnmount(() => {
    if (foodTagsInterval.value !== null) {
      clearInterval(foodTagsInterval.value)
    }
  })
  </script>
  
  <style scoped>
  .food-tag-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 1;
    overflow: hidden;
  }
  
  .food_tag {
    position: absolute;
    color: #000;
    white-space: nowrap;
    transition: opacity 0.3s ease, transform 20s linear;
    transform: translateY(-100px) rotate(0deg);
    animation: floatUp 20s linear forwards;
  }
  
  @keyframes floatUp {
    from {
      transform: translateY(0) rotate(0deg);
    }
    to {
      transform: translateY(-100vh) rotate(360deg);
    }
  }
  </style>