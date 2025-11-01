# mj-choose
### Intro

mj-choose 是一款轻量的 Vue 3 组件，用于展示和管理饮食分类与菜品。支持用户自定义数据源，可灵活配置专属的饮食分类及对应菜品。

### Installation

```
$ npm install mj-choose --save     或       yarn add mj-choose
```

###  Usage

```vue
(1) 基础使用（使用默认数据）
无需传递自定义数据，直接使用组件自带的默认饮食列表：
<template>
<div>
<h2>我的饮食计划</h2>
<MjChoose />
</div>
</template>

<script setup lang="ts">
// 导入组件
import MjChoose from 'mj-choose';
</script>
```

 

```vue



(2) 高级使用（自定义数据源）
    通过 custom-meal-data 属性传递自己定义的饮食分类和菜品：
    <template>
  <div>
    <h2>我的专属饮食计划</h2>
    <!-- 绑定自定义数据源 -->
    <MjChoose :custom-meal-data="myMealData" />
  </div>
</template>

<script setup lang="ts">
import MjChoose from 'mj-choose';

// 定义自定义饮食数据（按要求格式编写）
const myMealData = [
  { 
    name: '早饭', 
    dishes: ['豆浆', '油条', '肉包', '杂粮粥'] 
  },
  { 
    name: '午饭', 
    dishes: ['宫保鸡丁盖饭', '牛肉面', '麻辣烫', '减脂沙拉'] 
  },
  { 
    name: '下午茶', 
    dishes: ['奶茶', '蛋糕', '水果拼盘', '坚果'] 
  },
  { 
    name: '夜宵', 
    dishes: ['烧烤', '螺蛳粉', '泡面', '关东煮'] 
  }
];
</script>
```

### Tests

```
$ make test
```

### Contribution

欢迎用户提交 bug 修复和功能新增。

### License

MIT License

 Copyright (c) [2025] [mj] 

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions: The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software. THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.