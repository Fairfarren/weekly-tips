?> 作者：Lucas ，供稿日期：2019/08/16


## 概要

**props down && events up**

## 静态props和动态props
type常见类型: `String`, `Number`, `Boolean`, `Function`, `Object`, `Array`, `Symbol`

```html
<!-- parent -->
<template>
  <child-component
    one=1
    :two="2"
    three=true
    :four="true"
    five
  ></child-component>
</template>

<!-- child -->
<script>
  export default {
    props: ['one', 'two', 'three', 'four', 'five'],
    mounted() {
      console.log(typeof this.one, typeof this.two, typeof this.three, typeof this.four)
      console.log(this.five, typeof this.five)
    }
  }
</script>
```

或者:

```html

<!-- parent -->
<template>
  <child-component
    six="6"
    seven
  ></child-component>
</template>

<!-- child -->
<script>
  export default {
    props: {
      seven: {
        // type: Boolean
        // type: [Boolean, String]
        type: [String, Boolean]
      },
      eight: {
        // type: Boolean
        type: String
      }
    },
    mounted() {
      console.log(this.six, this.seven, this.eight)
    }
  }
</script>
```

## 单向数据流

```javascript
<!-- parent -->
<template>
  <child-component
    :detailData="detailData"
  ></child-component>
</template>
<script>
  export default {
    data() {
      return {
        detailData: {
          firstName: 'hello',
          lastName: 'world'
        }
      }
    }
  }
</script>

<!-- child -->
<script>
  export default {
    data() {
      return {
        detailData: {
          formData: this.detailData
        }
      }
    },
    props: {
      detailData: {
        type: Object,
        required: true
      }
    },
    computed: {
      fullName() {
        return this.detailData.firstName + this.detailData.lastName
      }
    }
  }
</script>
```

- 在子组件中修改值类型的prop时，浏览器console中会报错，且父组件中对应的prop字段值不会改变
- 在子组件中修改引用类型的prop时，浏览器console中不会报错，父组件中对应的prop字段值发生改变

## props之default

```html
<!-- parent -->
<template>
  <child-component></child-component>
</template>

<!-- child -->
<script>
  export default {
    data() {
      return {
        detailData: {
          formData: this.detailData
        }
      }
    },
    props: {
      detailData: {
        type: Object,
        default: () => ({
          pageId: '',
          pageName: ''',
          pageStatus: 1
        })
      }
    }
  }
</script>
```

或者：

```html
<!-- parent -->
<template>
  <child-component :detailData="detailData"></child-component>
</template>
<script>
  export default {
    data() {
      return {
        detailData: undefined
      }
    }
  }
</script>

<!-- child -->
<script>
  export default {
    data() {
      return {
        detailData: {
          formData: this.detailData
        }
      }
    },
    props: {
      detailData: {
        type: Object,
        default: () => ({
          pageId: '',
          pageName: ''',
          pageStatus: 1
        })
      }
    }
  }
</script>
```

?> 只有设置 `detailData`为`undefined`, 在子组件中访问`this.detailData`才可以获取到默认值。

## props源码

查看Vue源码[src\core\util\props.js](https://github.com/vuejs/vue/blob/dev/src/core/util/props.js#L45)，可以看到如下代码:

```javascript
 if (value === undefined) {
    value = getPropDefaultValue(vm, prop, key)
    // since the default value is a fresh copy,
    // make sure to observe it.
    const prevShouldObserve = shouldObserve
    toggleObserving(true)
    observe(value)
    toggleObserving(prevShouldObserve)

```

可见当`value`为`undefined`时才会去获取props的默认值。
