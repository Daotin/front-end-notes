# 79 vue-vue改变数组排序，视图未更新？

现在vue中有一个数组在页面使用v-for显示，然后给数组排序操作，但是视图未更新。

## 尝试方法

因为直接**采用下标赋值的方式修改数组的某一项不会触发视图更新**（但修改对象的某个属性是可以触发视图更新的），于是尝试使用 `push`，`Vue.set` 也无效的。

### _\(added in 20200421\)_

通过直接赋值的方式修改data中对象的某个属性是可以触发视图更新的，但是**如果子组件的视图直接渲染的父组件传过来的props对象，然后通过直接赋值的方式修改props的对象的属性，视图是不会更新的。必须采用`set`的方式才可以。**

> 1. 使用`push`、`pop`、`shift`、`unshift`、`splice`、`sort`和`reverse`会改变原数组的操作，VUE能正常检测到数组变化而更新视图。
> 2. 使用`filter`、`concat`、`slice`这几种方法操作数组，并不会改变原数组，VUE自然不会检测到数据变化也不会去更新视图。
> 3. 使用下标方式`number[0] = x`、或直接修改数组长度 `number.length = 1` 这两种方式虽然改变了原数组，但此时VUE检测不到数组变化，也不会更新视图。
> 4. Vue 可以检测到**对象属性的添加，修改或删除**，但是不能检测到对**对象的直接修改删除操作**。

## 解决方法

**在v-for循环的每个节点上面加上`key`属性即可。**
