?> 作者：石头，供稿日期：2019/08/09


## 为什么需要 Hooks API ?


组件 API 设计所面对的核心问题之一就是如何组织逻辑，以及如何在多个组件之间抽取和复用逻辑。基于 Vue 2.x 目前的 API 我们有一些常见的逻辑复用模式，但都或多或少存在一些问题。这些模式包括：

-   Mixins
-   高阶组件 (Higher-order Components, 也叫 HOCs)
-   Renderless Components (基于 scoped slots / 作用域插槽封装逻辑的组件）

总体来说，以上这些模式存在以下问题：

-   模版中的数据来源不清晰。举例来说，当一个组件中使用了多个 mixin 的时候，光看模版会很难分清一个属性到底是来自哪一个 mixin。HOC 也有类似的问题。
-   命名空间冲突。由不同开发者开发的 mixin 无法保证不会正好用到一样的属性或是方法名。HOC 在注入的 props 中也存在类似问题。
-   性能。HOC 和 Renderless Components 都需要额外的组件实例嵌套来封装逻辑，导致无谓的性能开销。

## Hooks API 样例

首先看一个标准的 SFC Vue 组件

```html
<template>
    <div>
        <span ref="dom">Count is {{ count }}</span>
        <span>plusOne is {{ plusOne }}</span>
        <button @click="increment">count++</button>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                count: 0
            }
        },

        computed: {
            plusOne() {
                return this.count + 1
            }
        },

        methods: {
            increment() {
                this.count++
            }
        }
    }
</script>
```

如果基于 Hooks API， 可以这么写

```html
<template>
    <div>
        <span>count is {{ count }}</span>
        <span>plusOne is {{ plusOne }}</span>
        <button @click="increment">count++</button>
    </div>
</template>

<script>
    // import { value, computed } from 'vue'
    import { value, computed } from 'vue-function-api'
    export default {
        setup() {
            const count = value(0)
            const plusOne = computed(() => count.value + 1)
            const increment = () => {
                count.value++
            }
            return {
                count,
                plusOne,
                increment
            }
        }
    }
</script>
```

Vue hooks 可以使用组合函数（composition function）来进行代码复用

```html
<template>
    <div class="mouse">{{ x }} {{ y }} {{ z }}</div>
</template>

<script>
    import { value, onMounted, onUnmounted } from 'vue-function-api'

    const useMouse = () => {
        const x = value(0)
        const y = value(0)
        const update = e => {
            x.value = e.pageX
            y.value = e.pageY
        }
        onMounted(() => {
            window.addEventListener('mousemove', update, false)
        })

        onUnmounted(() => {
            window.removeEventListener('mouseleave', update, false)
        })
        return { x, y }
    }

    const useOtherLogic = () => {
        const z = value(0)
        const randomZ = () => {
            setTimeout(() => {
                z.value = Math.floor(Math.random() * 100)
                randomZ()
            }, 1000)
        }
        randomZ()
        return { z }
    }

    export default {
        name: 'MouseComponent',
        setup(props, context) {
            const { x, y } = useMouse()
            const { z } = useOtherLogic()
            return { x, y, z }
        }
    }
</script>
```

?> 由于 Vue 3.x 还没发布，官方提供了[vue-function-api](https://github.com/vuejs/vue-function-api)来模拟 Hooks API，并保证和 Vue 3.x 兼容。

使用[vue-function-api](https://github.com/vuejs/vue-function-api)之前，需要先注册一下，比如：

```javascript
import Vue from 'vue'
import { plugin } from 'vue-function-api'

Vue.use(plugin)
```

## Hooks API 函数

Vue Hooks API 支持如下函数：

-   setup
-   value
-   state
-   computed
-   watch
-   lifeCycle 系列（包含 onCreated、onBeforeMount、onMounted、onBeforeUpdate、onUpdated、onActivated、onDeactivated、onBeforeDestroy、onDestroyed、onErrorCaptured、onUnmounted 等）
-   provide / inject

具体可以参考[vue-function-api](https://github.com/vuejs/vue-function-api)

`setup`是 Vue hooks 的入口， 整个生命周期只执行一次，性能较好。`setup()`中不可以使用 this 访问当前组件实例, 我们可以通过 setup 的第二个参数 context 来访问 vue2.x API 中实例上的属性。

```javascript
export default {
    props: {
        name: String
    },
    setup(props, context) {
        console.log(props.name)
        // context.attrs
        // context.slots
        // context.refs
        // context.emit
        // context.parent
        // context.root

        onMounted(() => {
            console.log('dom is mounted')
        })
    }
}
```


### watch函数说明




```javascript
<script>
import { value, computed, watch, onBeforeUpdate, onUpdated } from 'vue-function-api'
export default {
  name: 'Count',
  setup (props, context) {
    const count = value(0)
    const plusOne = computed(() => count.value + 1)
    const increment = () => {
      count.value++
    }

    onBeforeUpdate(() => {
      console.log(`before update....`)
    })

    onUpdated(() => {
      console.log(`updated...`)
    })
    
    const stopWatcher = watch(() => count.value * 2, (value, oldValue) => {
      // console.log(`count * 2 =`, value)
      context.refs.dom && console.log('dom: ', context.refs.dom.textContent)
    }, { flush: 'post', lazy: false }))

    watch([plusOne, () => count.value + 2], ([v1, v2]) => {
      console.log(`plusOne is ${v1}, pluseTwo is ${v2}`)
    })

    setTimeout(() => {
      // watch会返回一个函数，执行后会停止监听watch
      stopWatcher()
    }, 5000)
    return {
      count,
      plusOne,
      increment
    }
  }
}
</script>
```

?> 注意`watch`函数的第个三个参数，它是对象。`lazy`和vue 2.x中的`immediate`相反，默认为false，表示立即执行`watch`里面的处理函数。`flush`有`post`、`pre`和`sync`三个可选知，默认为`post`,标识在DOM更新完毕后执行。

- 默认`flush`为`post`，输入结果为：

```bash
before update....
updated...
dom:  count is 1
```

- 如果`flush`为`pre`，输入结果为：
```bash
before update....
dom:  count is 0
updated...
```

- 如果`flush`为`sync`，输入结果为：
```bash
dom:  count is 0
before update....
updated...
```