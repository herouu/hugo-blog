---
title: "Vue监听属性"
subtitle: ""
date: 2020-05-21T11:45:12+08:00
lastmod: 2020-05-21T11:45:12+08:00
draft: false
author: ""
authorLink: ""
description: ""

tags: ["Vue"]
categories: ["Vue"]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---
vue监听属性学习
<!--more-->

```vue
<template>
  <div>
    <button @click="nameChange">改变</button>
    <h2>name: {{this.name}}</h2>
    <h2>watch:{{this.nameWatch}}</h2>

    <button @click="promiseLearn">Promise</button>
  </div>
</template>

<script>
  export default {
    name: "WatchLearn",
    data() {
      return {
        name: "初始值",
        nameWatch: "",
      }
    },
    watch: {
      name(val) {
        this.nameWatch = val;
      }
    },
    methods: {
      nameChange() {
        return this.name = "变化值";
      }
    }
  }
</script>

<style scoped>

</style>
```
