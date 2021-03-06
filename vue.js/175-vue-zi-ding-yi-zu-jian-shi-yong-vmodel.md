# 175 vue-自定义组件使用v-model

## 官方教程

[自定义组件的 v-model](https://cn.vuejs.org/v2/guide/components-custom-events.html#自定义组件的-v-model)

> **只需要记住：一个组件上的 v-model 默认会利用名为 `value` 的 prop 和名为`input` 的事件。**

## 小示例

> 自定义一个custom-input组件。[点击查看在线示例](https://codesandbox.io/s/vue-demo-g3vjr)

```text
<template>
  <div class="custom-input">
    <span>我的输入框：</span>
    <input type="text" :value="value" @input="valueChanged">
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: String,
      default: ""
    }
  },
  created() {},
  methods: {
    valueChanged(e) {
      this.$emit("input", e.target.value);
    }
  }
};
</script>
```

父组件可以直接调用：

```text
<template>
  <div>
    <custom-input v-model="inputvalue"></custom-input>
    <span>{{inputvalue}}</span>
  </div>
</template>

<script>
import CustomInput from "./components/custom-input.vue";

export default {
  name: "App",
  components: {
    CustomInput
  },
  data: function() {
    return {
      inputvalue: "A"
    };
  }
};
</script>
```

当`custom-input` 的输入框的值变化的时候，可以看到父组件的`inputvalue` 的值跟着变化了。

这就是自定义组件中v-model最简单的使用。

