---
title: vue计算属性
date: 2020-05-17 00:00:00
tags: ["Vue"]
categories: ["Vue"]
---

### 计算属性
vue计算属性学习
<!--more-->
1. 计算属性的setter和getter 

   计算属性一般没有set方法，只是一个只读属性

   计算属性包含get和set的写法

   ```javascript
   fullNameGetAndSet: {
     get: function () {
       return this.first + " " + this.second;
     },
     set: function () {
       console.log("set");
     }
   }
   ```
   
2. 计算属性和methods属性对比
   计算属性具有缓存性，多次循环计算属性只执行一次，调用methods中的方法，执行多次，性能上计算属性优于methods中的方法
   
3. 完整demo
   
   ```vue
   <template>
     <div>
       <div>
         <div>计算属性简写：{{fullName}}</div>
         <div>计算属性完整写法：{{fullNameGetAndSet}}</div>
         <div>计算属性demo:{{total}}</div>
       </div>
     </div>
   </template>
   
   <script>
     export default {
       name: "ComputedLearn",
       data() {
         return {
           first: "first",
           second: "second",
           books: [
             {name: "book1", value: 20},
             {name: "book2", value: 30},
             {name: "book3", value: 40},
             {name: "book4", value: 50},
             {name: "book5", value: 60},
           ]
         };
       },
       computed: {
         fullName() {
           return this.first + " " + this.second
         },
         total() {
           return this.books.reduce((pre, current) => current.value + pre, 0);
         },
         fullNameGetAndSet: {
           get: function () {
             return this.first + " " + this.second;
           },
           set: function () {
             console.log("set");
           }
         }
       }
     }
   </script>
   
   <style scoped>
   
   </style>
   ```
   
   
   
