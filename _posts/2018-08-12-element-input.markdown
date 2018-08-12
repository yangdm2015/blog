---
layout:     post
title:      "从element input组件看怎么在设计组件时防止v-model绑定的属性陷入无限循环"
subtitle:   "vue组件的嵌套v-model绑定属性设计"
date:       2018-08-12
author:     "Shany"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Vue
    - Javascript 
    - element-ui
---

# 目录

# 准备工作

话不多说，先来一个 element-starter，然后组件中用

```js
import elInput from 'element-ui/packages/input/src/input';
```

这样的方式直接饮用 element 源码，直接饮用

```html
<template>
    <div>
        <el-input v-model='innerValue'></el-input>
        {{innerValue}}
    </div>
</template>
<script>
import elInput from 'element-ui/packages/input/src/input'
export default {
    props:{
        value:[String,Number]
    },
    data(){
        return {
            innerValue:''
        }
    },
    watch:{
        value(v){
            debugger
            this.innerValue = v
        },
        innerValue(v){
            debugger
            this.$emit('input',v)
        }
    },
    components:{
        elInput
    }

}
</script>
```

直接在 element input 组件中打断点，可以看到，在 elemet 源码中，使用 handleInput 函数监听底层 input 元素的 input 事件

```js
handleInput(event) {
          debugger
        const value = event.target.value;
        this.setCurrentValue(value);
        if (this.isOnComposition) return;
        this.$emit('input', value);
      },
```

同时对传入的 value 属性进行监控

```js
watch: {
      'value'(val, oldValue) {
        this.setCurrentValue(val);
      }
    },
```

这两个函数都调用了 setCurrentValue 函数

```js
setCurrentValue(value) {
          debugger
        if (this.isOnComposition && value === this.valueBeforeComposition) return;
        this.currentValue = value;
        if (this.isOnComposition) return;
        this.$nextTick(_ => {
          this.resizeTextarea();
        });
        if (this.validateEvent) {
          this.dispatch('ElFormItem', 'el.form.change', [value]);
        }
      },
```

最终是 currentValue 属性绑定到了底层的 input 元素。无关代码已省略

```html
<input
        :value="currentValue"
        @input="handleInput"
      >
```

通过以上代码，可以看出，在用户使用 v-model 指令绑定某一属性到 el-input 组件的情况下，当，数据流向是

el-input 中引用的底层的 input 元素发出 input 事件 --> handleInput --> setCurrentValue； emit( input) -->绑定 el-input 的 value 改变 --> setCurrentValue ,这时只改变了 currentValue 的值，而不会触发 emit 事件。

总结，通过观察 el-input 的源码，以下三个步骤可以防止 v-model 绑定的属性陷入循环

1.  在监听底层元素的 v-model 绑定属性的方法上，选择使用方法监听 input 事件，而不是给绑定的中间变量上 watcher。
2.  监听 input 事件的方法，做两件事，改变当前值，发出 input 事件。
3.  传入的 value 需要上 watcher 监听它的值发生改变，监听函数只要做一件事，就是改变本地的当前值。
