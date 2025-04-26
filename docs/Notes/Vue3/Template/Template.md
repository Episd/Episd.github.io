## 模板语法

### 文本插值

```vue
<span>Message: {{ msg }}</span>
```

用双大括号绑定属性的值，msg 属性值更改时，标签中的内容随之更改

### Attribute 绑定

响应式地绑定一个 attribute（属性），应该使用 `v-bind` 指令：

```vue
<div v-bind:id="dynamicId"></div>
```

上面的 `div` 标签的 `id` attribute 会与组件的 `dynamicId` 属性保持一致，如果

 `dynamicId` 是 `null` 或者 `undefined`，这个 `div` 标签的 `id` 属性就会被移除

可以用单冒号简写 `v-bind` 指令：

```vue
<div :id="dynamicId"/>
```

HTML 中的布尔属性可以被绑定，当绑定的属性为真时该属性存在，为假时移除该属性

```vue
<!-- isButtonDisabled 为真时 disabled 存在 -->
<button :disabled="isButtonDisabled">Button</button>
```

可以通过不带参数的 `v-bind` 绑定书写了多个 attribute 的 JS 对象

```Javascript
const objectOfAttrs = {
  id: 'container',
  class: 'wrapper',
  style: 'background-color:green'
}
```

```vue
<div v-bind="objectOfAttrs"></div>
```

除了别的属性，attribute 也可以绑定 JS 表达式：

```Javascript
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div :id="`list-${id}`"></div>
```

只要是能单独合法的写在 `return` 之后的 JS 表达式均可以被绑定，被绑定的表达式中也可以调用组件暴露的方法

```vue
<time :title="toTitleDate(date)" :datetime="date">
  {{ formatDate(date) }}
</time>
```

### 指令

指令是标签中带有 `v-` 前缀的特殊 attribute。
