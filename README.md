# fed-e-task-03-01

# 1、当我们点击按钮的时候动态给 data 增加的成员是否是响应式数据，如果不是的话，如何把新增成员设置成响应式数据，它的内部原理是什么。

```js
let vm = new Vue({
 el: '#el'
 data: {
  o: 'object',
  dog: {}
 },
 method: {
  clickHandler () {
   // 该 name 属性是否是响应式的
   this.dog.name = 'Trump'
  }
 }
})
```

 不是响应式数据

因为Vue设置响应式数据是在初始化Vue实例的时候进行的，通过method方法给data对象添加新的属性，不会给这个属性添加getter/setter

可以通过Vue.set或this.$set方法将一个对象的属性设置成响应式，例如：

```js
this.$set(this.dog, 'name', 'TRUMP')
```

set方法的原理是调用Observer中的defineReactive方法，给传入的对象和对象的属性手动添加getter/setter

# 2、请简述 Diff 算法的执行过程

Diff算法的过程主要是patch函数执行的过程

patch函数比较新节点和旧节点，如果新节点和旧节点是相同的节点（使用sameVNode判断），使用patchVnode函数更新两个节点的差异，如果新节点与旧节点不同，那么将新节点转化成真实DOM，插入旧节点的父元素中，并将旧节点删除

patchVnode的比较过程：

- 如果新节点与旧节点指向同一个对象，直接返回
- 如果新节点有text，且不等于旧节点的text
  - 如果旧节点有children，则移除旧节点children的DOM元素
  - 如果旧节点没有children，则将最终的DOM的textContent设置成新节点的text
- 如果新节点有子节点，而旧节点没有，则将新节点转换为真实DOM添加到el
- 如果只有旧节点有children，则移除所有的旧节点
- 如果新旧节点都有children且不相等，则调用updateChildren函数更新子节点的差异

updateChildren是Diff算法的核心，会对新旧节点的children进行比较，两者的children都是数组，有多种比较条件

updateChildren的过程：

1. 新旧开始节点相同（key和sel相同）
   1. 调用patchVnode对比和更新节点
   2. 将旧开始和新开始索引往后移动1位
2. 新旧开始节点不相同，会从后往前比较，调用patchVnode对比和更新节点，将旧结束和新结束索引往前移动1位
3. 旧开始节点和新结束节点使用patchVnode更新差异，将旧开始节点移动到旧结束节点的右侧，更新索引，下一次对比，则是旧开始节点++，新结束节点--
4. 旧结束节点和新开始节点使用patchVnode更新差异，将旧结束节点移动到旧开始节点的左侧，更新索引，下一次对比，则是旧结束节点--，新开始节点++
5. 如果非以上四种情况，则先遍历新开始节点
   1. 从旧节点数组中查找是否有和新开始节点相同的节点，如果没找到，则说明新开始的节点是完全新的，将其插入到旧节点数组之前
   2. 如果在旧节点数组中找到了与新开始节点相同的节点，key相同，但sel不相同，将这个节点通过patchVnode更新之后，插入到旧节点数组之前
   3. 如果key和sel都相同，说明新节点和旧节点相同，则将其复制给elmToMove对象，在这里更新差异过后，再将其插入到旧节点数组之前