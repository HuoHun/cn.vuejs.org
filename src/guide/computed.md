---
title: 计算属性
type: guide
order: 5
---

在模板中表达式非常便利，但是它们实际上只用于简单的操作。模板是为了描述视图的结构。在模板中放入太多的逻辑会让模板过重且难以维护。这就是为什么 Vue.js 将绑定表达式限制为一个表达式。如果需要多于一个表达式的逻辑，应当使用**计算属性**。

### 基础例子

``` html
<div id="example">
  a={{ a }}, b={{ b }}
</div>
```

``` js
var vm = new Vue({
  el: '#example',
  data: {
    a: 1
  },
  computed: {
    // getter
    b: function () {
      // `this` 指向 vm 实例
      return this.a + 1
    }
  }
})
```

结果：

{% raw %}
<div id="example" class="demo">
  a={{ a }}, b={{ b }}
</div>
<script>
var vm = new Vue({
  el: '#example',
  data: {
    a: 1
  },
  computed: {
    b: function () {
      return this.a + 1
    }
  }
})
</script>
{% endraw %}

这里我们声明了一个计算属性 `b`。我们提供的函数将用作属性 `vm.b`的读取器函数（getter）。

``` js
console.log(vm.b) // -> 2
vm.a = 2
console.log(vm.b) // -> 3
```

你可以打开浏览器的控制台，修改例子的 vm。`vm.b` 的值始终取决于 `vm.a` 的值。

在模板中可以像普通属性一样将数据绑定到计算属性。Vue 知道 `vm.b` 依赖于 `vm.a`，因此在 `vm.a` 改变时它将更新任何依赖于 `vm.b` 的绑定。而且最妙的是我们是声明式地创建这种依赖关系：计算读取器函数是纯粹的且没有副效应，这让它易于测试与理解。

### 计算属性 vs. $watch

Vue.js 提供了一个方法 `$watch`，它用于观察 Vue 实例上的数据变动。当一些数据需要根据其它数据变化时， `$watch` 很诱人 —— 特别是如果你来自 AngularJS。不过，通常更好的办法是使用计算属性而不是一个命令式的 `$watch` 回调。考虑下面例子：

``` html
<div id="demo">{{fullName}}</div>
```

``` js
var vm = new Vue({
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  }
})

vm.$watch('firstName', function (val) {
  this.fullName = val + ' ' + this.lastName
})

vm.$watch('lastName', function (val) {
  this.fullName = this.firstName + ' ' + val
})
```

上面代码是命令式的重复的。跟计算属性对比：

``` js
var vm = new Vue({
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```

更好，不是吗？

### 计算存储器

计算属性默认只是读取器，不过在需要时你也可以提供一个存储器：

``` js
// ...
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...
```

现在在调用 `vm.fullName = 'John Doe'` 时，存储器被调用，`vm.firstName` 和 `vm.lastName` 将相应地更新。

关于计算属性是如何更新的技术细节见[响应系统](reactivity.html#Inside_Computed_Properties)。
