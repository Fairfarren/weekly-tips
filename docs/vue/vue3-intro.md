?> 作者：石头，供稿日期：2020/06/05


## 为什么需要 Composition API ?

- 当要去理解一个组件时，我们更加关心的是“这个组件是要干什么” (即代码背后的意图)，而不是“这个组件用到了什么选项”。

- 基于选项的 API 撰写出来的代码自然采用了后者的表述方式，然而对前者的表述并不好。

- 一个[现实例子](https://github.com/vuejs/vue-cli/blob/a09407dd5b9f18ace7501ddb603b95e31d6d93c0/packages/@vue/cli-ui/src/components/folder/FolderExplorer.vue#L198-L404)


![图片](https://gitee.com/zhaolei/pictures/raw/master/uPic/5Ocu0g-1591324828461.jpg)



## Composition API 样例

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

如果基于 Composition API， 可以这么写

```html
<template>
    <span>count is {{ count }}</span>
    <span>plusOne is {{ plusOne }}</span>
    <button @click="increment">count++</button>
</template>

<script>
    import { ref, computed } from 'vue'
    export default {
        setup() {
            const count = ref(0)
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


##  Composition API 函数组合

Vue Composition API 可以使用组合函数（composition function）来进行代码复用

```html
<template>
    <div class="mouse">{{ x }} {{ y }} {{ z }}</div>
</template>

<script>
    import { ref, onMounted, onUnmounted } from 'vue'

    const useMouse = () => {
        const x = ref(0)
        const y = ref(0)
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
        const z = ref(0)
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

?> 由于 Vue 3.x 还为发布正式版，官方提供了[ @vue/composition-api](https://github.com/vuejs/composition-api)来模拟 Vue 3 Composition API，并保证和 Vue 3.x 兼容。

使用[ @vue/composition-api](https://github.com/vuejs/composition-api)之前，需要先注册一下，比如：

```js
import Vue from 'vue'
import VueCompositionApi from '@vue/composition-api'

Vue.use(VueCompositionApi)
```

具体使用请参考官方文档。

## Composition API 函数

Vue Composition API 支持如下函数：

-   setup
-   ref
-   reactive
-   computed
-   watch
-   lifeCycle 系列（包含onBeforeMount、onMounted、onBeforeUpdate、onUpdated、onActivated、onDeactivated、onBeforeUnmount、onUnmounted、onErrorCaptured等）
-   provide / inject
-   readonly

### setup

`setup`是 Vue hooks 的入口， 整个生命周期只执行一次，性能较好。`setup()`中不可以使用 this 访问当前组件实例, 我们可以通过 setup 的第二个参数 context 来访问 vue2.x API 中实例上的属性。

```js
export default {
    props: {
        name: String
    },
    setup(props, context) {
        console.log(props.name)
        // context.attrs
        // context.slots
        // context.emit

        onMounted(() => {
            console.log('dom is mounted')
        })
    }
}
```

### ref

接受一个参数值并返回一个响应式且可改变的`ref`对象。`ref`对象拥有一个指向内部值的单一属性 `.value`。

```html

<template>
  <div>{{ count }}</div>
</template>

<script>
import { ref } from 'vue'
export default {
    setup() {
        const count = ref(0)
        console.log(count.value) // 0
        count.value++
        console.log(count.value) // 1
        return {
            count
        }
    }
}
</script>
```

### reactive

接收一个普通对象然后返回该普通对象的响应式代理。等同于 2.x 的 `Vue.observable()`

```js
import { reactive } from 'vue'

const useMouseWithReactive = () => {
  const pos = reactive({
    x: 0,
    y: 0
  })
  const update = e => {
    pos.x = e.pageX
    pos.y = e.pageY
  }
  onMounted(() => {
    window.addEventListener('mousemove', update, false)
  })

  onUnmounted(() => {
    window.removeEventListener('mouseleave', update, false)
  })
  return pos
}

```


### computed

传入一个 `getter` 函数，返回一个默认不可手动修改的 `ref` 对象。

```javacript
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // 错误！

```

或者传入一个拥有 `get` 和 `set` 函数的对象，创建一个可手动修改的计算状态。

```js
const count = ref(1)
const plusOne = computed({
  get: () => count.value + 1,
  set: (val) => {
    count.value = val - 1
  },
})

plusOne.value = 1
console.log(count.value) // 0
```


### watch函数说明

```html

<template>
    <span ref="dom">count is {{ count }}</span> <br />
    <span>plusOne is {{ plusOne }}</span> <br />
    <button @click="increment">count++</button>
</template>

<script>
import { ref, computed, watch, onBeforeUpdate, onUpdated } from 'vue'
export default {
  name: 'Count',
  setup (props, context) {
    const count = ref(0)
    const dom = ref(null)
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
    
    const stopWatcher = watch(count, (value, oldValue) => {
        console.log(`currentValue: ${count.value}, dom: ${dom.value.textContent}`)
    }, { flush: 'post', immediate: false })

    watch( [plusOne, () => count.value + 2], ([v1, v2]) => {
      console.log(`plusOne is ${v1}, pluseTwo is ${v2}`)

    })

    setTimeout(() => {
      // watch会返回一个函数，执行后会停止监听watch
        stopWatcher()
    }, 5000)

    return {
      dom,
      count,
      plusOne,
      increment
    }
  }
}
</script>
```

?> 注意`watch`函数的第个三个参数，它是对象。`immediate`默认为false，表示是否立即执行`watch`里面的处理函数。`flush`有`post`、`pre`和`sync`三个可选值，默认为`post`,标识在DOM更新完毕后执行。

- 默认`flush`为`post`，输出结果为：

```bash
before update....
currentValue: 1, dom: count is 1
updated...
```

- 如果`flush`为`pre`，输出结果为：
```bash
before update....
currentValue: 1, dom: count is 1
updated...
```

- 如果`flush`为`sync`，输出结果为：
```bash
currentValue: 1, dom: count is 0
before update....
updated...
```


### provide/inject

`provide` 和 `inject` 提供依赖注入，功能类似 2.x 的 `provide/inject`。两者都只能在当前活动组件实例的 setup() 中调用。

```js
const ThemeSymbol = symbol()
// Parent
export default {
    setup() {
        const theme = ref('light')
        // const copyTheme = readonly(theme)
        provide(ThemeSymbol, theme)
    }
}

// Child 

export default {
    setup() {
        //  theme 是响应式的
        const theme = inject(ThemeSymbol)
        return {
            theme
        }

    }
}

```

### readonly

传入一个对象（响应式或普通）或 `ref`，返回一个原始对象的只读代理。一个只读的代理是“深层的”，对象内部任何嵌套的属性也都是只读的。

```js
const theme = ref('light')
const copyTheme = readonly(theme)
setTimeout(() => {
    // 控制台会显示警告：Set operation on key "value" failed: target is readonly.
    copyTheme.value = 'dark'
}, 2000)
```



## `ref` vs `reactive`

`ref`和 `reactive`对可以将参数值变为可响应式对象，前者需要 进行`.value`才可以访问到实际值，后者可以直接访问到。但是`reactive`返回的值在结构后会丢失响应式。


```js
import { ref, onMounted, onUnmounted, reactive, toRefs, onBeforeMount, toRefs} from 'vue'

const useMouse = () => {
    const x = ref(0)
    const y = ref(0)
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

const useMouseWithReactive = () => {
  const pos = reactive({
    x: 0,
    y: 0
  })
  const update = e => {
    pos.x = e.pageX
    pos.y = e.pageY
  }
  onMounted(() => {
    window.addEventListener('mousemove', update, false)
  })

  onUnmounted(() => {
    window.removeEventListener('mouseleave', update, false)
  })
  // return toRefs(pos)
  return pos
}

export  { useMouse, useMouseWithReactive }
```

```html

<template>
    {{x}} / {{y}}
</template>

<script>
import { useMouseWithReactive, toRefs }  from './composition/useMouse'

  setup() {
     const pos = useMouseWithReactive()
     
     return  {
       ...pos
     }
  }

</script>

```

此时， 发发现x、y一直为0， 可以使用`toRefs`将其转让基于`ref`的可响应式对象。 比如 `toRefs(pos)`，这样才解构后依然拥有响应式功能。



## 环境准备

使用Vue 3可以参考如下3种方式

- 基于vue-cli升级到vue 3， 使用[vue-cli-plugin-vue-next](https://github.com/vuejs/vue-cli-plugin-vue-next)插件
- 基于[vue-next-webpack-preview](https://github.com/vuejs/vue-next-webpack-preview) webpack样例
- 基于[Vite](https://github.com/vitejs/vite)， 推荐使用，比较简单快捷


## 参考资料 

-  [@vue/composition-api](https://github.com/vuejs/composition-api/blob/master/README.zh-CN.md)
-  [Composition API](https://vue-composition-api-rfc.netlify.app/zh/api.html)
-  [Composition API RFC](https://vue-composition-api-rfc.netlify.app/zh/)
-  [New features in Vue 3 and how to use them](https://blog.logrocket.com/new-features-in-vue-3-and-how-to-use-them/)