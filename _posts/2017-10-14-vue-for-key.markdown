---
layout:     post
title:      "v-for加key属性的作用详解"
subtitle:   "一篇文章看懂key属性的影响"
date:       2017-10-14
author:     "Shany"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - JavaScript
    - Vue
---

> This document is not completed and will be updated anytime.
vue 官方文档中说
当 Vue.js 用 v-for 正在更新已渲染过的元素列表时，它默认用“就地复用”策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序， 而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的每个元素。这个类似 Vue 1.x 的 track-by="$index" 。

这个默认的模式是高效的，但是只适用于不依赖<strong>子组件状态</strong>或临时 DOM 状态 (例如：表单输入值) 的列表渲染输出。

什么是子组件状态？ 答案是，子组件本地属性

二话不说上代码

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>key-index</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>

<body>
    <div id="app">
        {{ message }}
        <div class="btn" @click='add2list'>点击加列表</div>
        有key
        <ul>
            <todo-item v-for="item in groceryList" v-bind:todo="item" v-bind:key="item.id">
            </todo-item>
            
        </ul>
        没有key
        <ul>
            <todo-item v-for="item in groceryList" v-bind:todo="item" >
            </todo-item>
        </ul>
    </div>

    <script>
        Vue.component('todo-item', {
            props: ['todo'],
            template: '<li>{{ todo.text }} {{todo.id}}{{localText}}</li>',
            data() {
                return {
                    localText: ''
                }
            },
            created() {
                this.localText = this.todo.text
              console.log('this.localText = ',this.localText)
            }
        })
        var app = new Vue({
            el: '#app',
            data: {
                message: 'Hello Vue!',
                groceryList: [{
                    id: 0,
                    text: '蔬菜'
                }, {
                    id: 1,
                    text: '奶酪'
                }, {
                    id: 2,
                    text: '菠萝'
                }]
            },
            methods: {
                add2list() {
                    this.groceryList.splice(1, 1, {
                        id: 3,
                        text: '香蕉'
                    })
                }
            }
        })
    </script>
</body>

</html>
```

在点击按钮后，列表中第二个item被换成了香蕉，id也变为3，但在没有绑定key值的情况下，会发现第二个的localText变量仍然是初始化时的值。
由此可知，在不绑定key时，被vue的‘就地复用’替换的组件是不会执行create方法的，控制台的输出也证明了这一点