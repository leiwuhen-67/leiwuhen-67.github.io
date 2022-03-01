---
title: vue3.2 setup语法糖
date: 2022-02-28 10:42:15
categories: JavaScript
tags:
---
vue3.0与vue3.2对比：
1）、vue3.0：
①、变量、方法必须return出来，才能在template中使用。
②、引入自定义组件时，还必须在components中进行注册后才能使用。
③、props与emit通过setup函数来获取。

2）、vue3.2：
①、只需在script标签中添加setup，变量和方法不需要return就能使用。
<!-- more -->
②、组件只需要引入，不用在components中注册就能够使用。
③、子组件中props通过defineProps来获取：
例如：

```
<template>
  <div>
    子组件{{name}} // 心欲无痕========
  </div>
</template>

<script setup>
  import {defineProps} from 'vue'

  defineProps({
   name:{
     type:String,
     default:'默认值'
   }
 })
</script>
```

④、子组件中emit通过defineEmits来获取：
例如：

```
<template>
  <div>
    子组件{{name}}
    <button @click="ziupdata">按钮</button>
  </div>
</template>

<script setup>
  import {defineEmits} from 'vue'

  //自定义函数，父组件可以触发
  const em=defineEmits(['updata'])
  const ziupdata=()=>{
    em("updata",'我是子组件的值')
  }

</script>
```