?> 作者：DongChen_Hu ，供稿日期：2019/12/27


# 单元测试

| 环境     | 版本号   |
| -------- | -------- |
| node     | v12.13.1 |
| npm      | 6.13.1   |
| @vue/cli | 4.1.1    |

*[vue-jest-demo](https://gitee.com/hudongchen/vue-jest-demo)*

## 作用

- 正确性
  * 测试可以验证代码的正确性，在上线前做到心里有底
- **自动化**
  * 当然手工也可以测试，通过console可以打印出内部信息，但是这是一次性的事情，下次测试还需要从头来过，效率不能得到保证。通过编写测试用例，可以做到一次编写，多次运行
- 解释性
  * 测试用例用于测试接口、模块的重要性，那么在测试用例中就会涉及如何使用这些API。其他开发人员如果要使用这些API，那阅读测试用例是一种很好地途径，有时比文档说明更清晰
- **驱动开发，指导设计**
  * 代码被测试的前提是代码本身的可测试性，那么要保证代码的可测试性，就需要在开发中注意API的设计，TDD将测试前移就是起到这么一个作用
- 保证重构
  * 互联网行业产品迭代速度很快，迭代后必然存在代码重构的过程，那怎么才能保证重构后代码的质量呢？有测试用例做后盾，就可以大胆的进行重构

## 原则

- 测试代码时，**只考虑测试，不考虑内部实现**
- 数据尽量模拟现实，越靠近现实越好
- 充分考虑数据的边界条件
- 对**重点、复杂、核心代码**，重点测试
- 测试、功能开发相结合，有利于设计和代码重构

## 常用操作

### 搭建工程

```
PS C:\Users\hudc\Workspace\测试\20191203> vue create vue-jest

Vue CLI v4.1.1
? Please pick a preset: Manually select features
? Check the features needed for your project: Babel, TS, Router, Vuex, CSS Pre-processors, Linter, Unit
? Use class-style component syntax? Yes
? Use Babel alongside TypeScript (required for modern mode, auto-detected polyfills, transpiling JSX)? Yes
? Use history mode for router? (Requires proper server setup for index fallback in production) Yes
? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): Sass/SCSS (with node-sass)
? Pick a linter / formatter config: Standard
? Pick additional lint features: (Press <space> to select, <a> to toggle all, <i> to invert selection)Lint on save
? Pick a unit testing solution: Jest
? Where do you prefer placing config for Babel, ESLint, etc.? In dedicated config files
? Save this as a preset for future projects? No
```

### 启动测试

```bash
# 启动所有测试
npm run test:unit
# 指定名称测试
npm run test:unit -t App
```

## 编写测试用例

### 内部状态

- 测试组件

```html
<!-- Counter.vue -->
<template>
  <div class="counter">
    <span>{{ count }}</span>
    <el-button @click="count++" icon="el-icon-plus" type="success">累计加1</el-button>
  </div>
</template>

<script lang="ts">
  import { Component, Vue } from 'vue-property-decorator'
  @Component({ name: 'Counter' })
  export default class Counter extends Vue {
    count: number = 0
  }
</script>
```

- 测试用例

```ts
// Counter.spec.ts
import { mount, createLocalVue } from '@vue/test-utils'
import Counter from '@/components/Counter.vue'
import { Button } from 'element-ui'

const localVue = createLocalVue()
localVue.component(Button.name, Button)
const wrapper = mount(Counter, { localVue })

describe('Counter.vue', () => {
  it('默认状态值为0', () => {
    expect(wrapper.vm.$data.count).toBe(0)
  })

  it('修改count值为20', () => {
    wrapper.setData({ count: 20 })
    expect(wrapper.find('span').text()).toMatch('20')
  })

  it('由子组件触发click事件，更新count为22', () => {
    wrapper.setData({ count: 21 })
    wrapper.find(Button).vm.$emit('click')
    expect(wrapper.vm.$data.count).toBe(22)
  })

  it('由子组件内的button触发点击事件，更新count为11', () => {
    wrapper.setData({ count: 10 })
    const elBtn = wrapper.find(Button)
    elBtn.find('button').trigger('click')
    expect(wrapper.vm.$data.count).toBe(11)
  })
})
```

### 组件通信

- 测试组件

```html
<!-- Counter2.vue -->
<template>
  <div class="counter-2">
    <span>{{ count }}</span>
    <el-button @click="update" icon="el-icon-plus" type="success"
    >累计加1</el-button>
  </div>
</template>

<script lang="ts">
  import { Component, Vue, Prop, Emit } from 'vue-property-decorator'
  @Component({ name: 'Counter2' })
  export default class Counter2 extends Vue {
    @Prop({ default: 10 }) readonly count!: number
    @Emit() update(): number {
      return this.count + 1
    }
  }
</script>
```

- 测试用例

```ts
// Counter2.spec.ts
import { mount, createLocalVue } from '@vue/test-utils'
import Counter2 from '@/components/Counter2.vue'
import { Button } from 'element-ui'

const localVue = createLocalVue()
localVue.component(Button.name, Button)

describe('Counter2.vue', () => {
  it('默认prop值为10', () => {
    const wrapper = mount(Counter2, { localVue })
    expect(wrapper.props().count).toBe(10)
  })

  it('设置propData.count值为11', () => {
    const wrapper = mount(Counter2, { localVue, propsData: { count: 11 } })
    expect(wrapper.props('count')).toBe(11)
  })

  it('修改props.count值为12', () => {
    const wrapper = mount(Counter2, { localVue })
    wrapper.setProps({ count: 12 })
    expect(wrapper.find('span').text()).toMatch('12')
  })

  it('子组件click事件，触发update方法调用', () => {
    const wrapper = mount(Counter2, { localVue })
    const update = jest.fn()
    wrapper.setMethods({ update })
    wrapper.find(Button).vm.$emit('click')
    expect(update).toBeCalled()
  })

  it('连续点击，update方法被调用2次', () => {
    const wrapper = mount(Counter2, { localVue })
    const update = jest.fn()
    wrapper.vm.$on('update', update)
    // 第一次点击
    wrapper.find(Button).vm.$emit('click')
    expect(update).toBeCalledTimes(1)
    expect(update).toBeCalledWith(11)
    // 第二次点击
    wrapper.setProps({ count: 11 })
    wrapper.find(Button).vm.$emit('click')
    expect(update).toBeCalledTimes(2)
    expect(update).toBeCalledWith(12)
  })
})
```

### 异步操作

- （一）测试接口

```ts
// request.ts
import request from 'axios'
export const GET = (url: string, params?: object, others?: object) => {
  return request.get(url, { params, ...others })
}

// api.ts
import { GET } from './request'
export default {
  getUsers: (params?: object) =>
    GET('https://jsonplaceholder.typicode.com/users', params)
}
```

- （一）测试用例

```ts
// index.spec.ts
import Api from '@/api'

describe('@/api/index.ts', () => {
  it('getUsers接口，不接收入参', () => {
    return Api.getUsers().then((res: any) => {
      expect(res.length).toBe(10)
    })
  })

  it('getUsers接口，接收入参', async () => {
    const res: any = await Api.getUsers({ id: 1 })
    expect(res.length).toBe(1)
    expect(res[0].id).toBe(1)
  })
})
```

- （二）测试组件

```html
<!-- FetchData.vue -->
<template>
  <div class="fetch">{{ msg }}</div>
</template>

<script lang="ts">
  import { Component, Vue, Prop } from 'vue-property-decorator'
  import utils from '@/utils'
  import Api from '@/api'

  @Component({ name: 'FetchData' })
  export default class FetchData extends Vue {
    msg = '数据加载中...'
    created() {
      this.getUser()
    }
    getUser() {
      const param = { id: utils.getRandomNum(1, 10) }
      Api.getUsers(param).then(res => {
        if (Array.isArray(res)) {
          this.msg = res[0]!.username
        }
      })
    }
  }
</script>
```

- （二）测试用例

```ts
// FetchData.spec.ts
import { mount } from '@vue/test-utils'
import Vue from 'vue'
import FetchData from '@/components/FetchData.vue'
jest.mock('@/utils', () => ({
  getRandomNum: () => 6
}))
jest.mock('@/api', () => ({
  getUsers: () => Promise.resolve([{ username: '~!@#$%^&*()_+' }])
}))

describe('FetchData.vue', () => {
  it('在created生命周期中调用方法', () => {
    const getUser = jest.fn()
    const options = {
      methods: { getUser }
    }
    mount(FetchData, options)
    expect(getUser).toBeCalled()
  })

  it('接收异步接口的返回值', () => {
    const wrapper = mount(FetchData)
    return Vue.nextTick().then(() => {
      expect(wrapper.vm.$data.msg).toMatch('~!@#$%^&*()_+')
    })
  })
})
```

### 路由切换

- 测试组件

```html
<!-- App.vue -->
<template>
  <div id="app">
    <div id="nav">
      <router-link exact to="/">Home</router-link> |
      <router-link exact to="/about">About</router-link> |
      <router-link to="/page">Page</router-link>
    </div>
    <router-view />
  </div>
</template>
```

- 测试用例

```ts
// App.spec.ts
import {
  mount,
  shallowMount,
  createLocalVue,
  config,
  RouterLinkStub
} from '@vue/test-utils'
import VueRouter, { Route } from 'vue-router'
import routes from '@/router/routes'
import App from '@/App.vue'
import ElementUI from 'element-ui'
import PageA from '@/views/PageA.vue'
import { beforeEachGuard } from '@/router/guards'
// ;(config.stubs as Record<string, any>)['router-link'] = RouterLinkStub

const localVue = createLocalVue()
localVue.use(VueRouter)
localVue.use(ElementUI)

describe('App.vue', () => {
  it('有3个路由', () => {
    const wrapper = shallowMount(App, {
      localVue,
      stubs: { 'router-link': RouterLinkStub, 'router-view': '<span />' }
    })
    expect(wrapper.findAll(RouterLinkStub).length).toBe(3)
  })

  it('第3个路由是`/page`', () => {
    const wrapper = mount(App, {
      localVue,
      stubs: { RouterLink: RouterLinkStub, RouterView: true } // ['router-view']
    })
    const link = wrapper.findAll(RouterLinkStub).at(2)
    expect(link.props().to).toMatch('/page')
  })

  it('路由到About页面（callback/done）', done => {
    const router = new VueRouter({ routes })
    const wrapper = mount(App, { localVue, router })
    router.push('/about').then(() => {
      expect(router.currentRoute.path).toMatch('/about')
      done()
    })
  })

  it('路由到PageA页面（Promise/return）', () => {
    const router = new VueRouter({ routes })
    const wrapper = mount(App, { localVue, router })
    return router.push('/page').then(() => {
      const pa = wrapper.find(PageA)
      expect(pa.exists()).toBe(true)
    })
  })

  it('路由到PageA页面（async/await）', async () => {
    const router = new VueRouter({ routes })
    const wrapper = mount(App, { localVue, router })
    await router.push('/page')
    const pa = wrapper.find({ name: 'PageA' })
    expect(pa.is(PageA)).toBe(true)
  })
})

describe('App.vue', () => {
  it('页面跳转触发全局前置守卫', async () => {
    const router = new VueRouter({ routes })
    const to = { name: 'about' } as Route
    const from = { name: 'home' } as Route
    const next = jest.fn()
    beforeEachGuard(to, from, next)
    router.beforeEach(beforeEachGuard)
    const wrapper = shallowMount(App, { localVue, router })
    expect(router.currentRoute.path).toMatch('/')
    await router.push(to)
    expect(next).toBeCalled()
    expect(router.currentRoute.path).toMatch('/about')
  })
})
```

### 状态管理

- 测试声明

```ts
// root.d.ts
interface IRootState {
  isLoading: boolean
}

// count.d.ts
interface ICountState {
  count: number
}

// store.ts
import Vue from 'vue'
import Vuex from 'vuex'
import state from './state'
import getters from './getters'
import mutations from './mutations'
import count from './count'

Vue.use(Vuex)

export default new Vuex.Store<IRootState>({
  state,
  getters,
  mutations,
  modules: { count }
})

// types.ts
export const SHOW_LOADING = 'SHOW_LOADING'
export const HIDE_LOADING = 'HIDE_LOADING'

// state.ts
export default { isLoading: false } as IRootState

// getters.ts
import { GetterTree } from 'vuex'
export default {
  loading: state => state.isLoading
} as GetterTree<IRootState, any>

// mutations.ts
import { MutationTree } from 'vuex'
import { SHOW_LOADING, HIDE_LOADING } from './types'
export default {
  [SHOW_LOADING](state) {
    state.isLoading = true
  },
  [HIDE_LOADING](state) {
    state.isLoading = false
  }
} as MutationTree<IRootState>

// count/index.ts
import { Module } from 'vuex'
import state from './state'
import getters from './getters'
import mutations from './mutations'
import actions from './actions'
const countModule: Module<ICountState, IRootState> = {
  namespaced: true,
  state,
  mutations,
  actions,
  getters
}
export default countModule

// count/types.ts
export const INCREMENT = 'INCREMENT'

// count/state.ts
const state: ICountState = { count: 0 }
export default state

// count/getters.ts
import { GetterTree } from 'vuex'
const getters: GetterTree<ICountState, IRootState> = {
  iCount: state => state.count
}
export default getters

// count/mutations.ts
import { MutationTree } from 'vuex'
import { INCREMENT } from './types'
const mutations: MutationTree<ICountState> = {
  [INCREMENT](state, payload: number = 0) {
    state.count = payload
  }
}
export default mutations

// count/actions.ts
import { ActionTree } from 'vuex'
import { INCREMENT } from './types'
import { SHOW_LOADING, HIDE_LOADING } from '../types'
const actions: ActionTree<ICountState, IRootState> = {
  [INCREMENT]({ commit }, payload?: number) {
    return new Promise<void>(resolve => {
      commit(SHOW_LOADING, null, { root: true })
      setTimeout(() => {
        commit(INCREMENT, payload)
        commit(HIDE_LOADING, null, { root: true })
        resolve()
      }, 500)
    })
  }
}
export default actions
```

- 测试组件

```html
<!-- Counter3.vue -->
<template>
  <div class="counter">
    <span>{{ iCount }}</span>
    <el-button-group>
      <el-button
        @click="handleClick(iCount + 1)"
        icon="el-icon-plus"
        type="success"
      >加1</el-button>
      <el-button
        @click="handleClick(iCount - 1)"
        icon="el-icon-minus"
        type="danger"
      >减1</el-button>
      <el-button @click="handleClick()" icon="el-icon-refresh" type="primary"
      >Reset</el-button>
    </el-button-group>
  </div>
</template>

<script lang="ts">
  import { Component, Vue } from 'vue-property-decorator'
  import { namespace } from 'vuex-class'
  import { INCREMENT } from '@/store/count/types'

  const countModule = namespace('count')
  @Component({ name: 'Counter3' })
  export default class Counter3 extends Vue {
    @countModule.Getter readonly iCount!: number
    @countModule.Action(INCREMENT) handleClick!: (
      newCount?: number
    ) => Promise<void>
  }
</script>
```

- 测试用例

```ts
// Counter3.spec.ts
import { mount, createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'
import state from '@/store/state'
import getters from '@/store/getters'
import mutations from '@/store/mutations'
import count from '@/store/count'
import Counter3 from '@/components/Counter3.vue'
import { Button, ButtonGroup } from 'element-ui'

describe('Counter3.vue', () => {
  const localVue = createLocalVue()
  localVue.component(Button.name, Button)
  localVue.component(ButtonGroup.name, ButtonGroup)
  localVue.use(Vuex)

  const store = new Vuex.Store<IRootState>({
    state,
    getters,
    mutations,
    modules: { count }
  })
  const wrapper = mount(Counter3, { store, localVue })
  const vm = <any>wrapper.vm

  describe('组件状态（默认）', () => {
    it('默认状态值为0', () => {
      expect((store.state as any).count.count).toBe(0)
      expect(vm.iCount).toBe(0)
    })

    it('修改store状态值为20', () => {
      ;(store.state as any).count.count = 20
      expect(vm.iCount).toBe(20)
      expect(wrapper.find('span').text()).toMatch('20')
    })

    it('有一个Group组件', () => {
      expect(wrapper.contains(ButtonGroup)).toBe(true)
    })

    it('有三个Button按钮', () => {
      const btns = wrapper.findAll(Button)
      expect(btns.length).toBe(3)
    })
  })

  describe('组件状态变化（通过事件触发）', () => {
    beforeEach(() => {
      jest.useFakeTimers()
      ;(store.state as any).count.count = 0
    })

    afterAll(() => {
      jest.useRealTimers()
      ;(store.state as any).count.count = 0
    })

    it('点击加1按钮，状态值为1', () => {
      const btn = wrapper.findAll(Button).at(0)
      expect(btn.props('type')).toMatch(/success/)
      btn.vm.$emit('click')

      expect(vm.iCount).toBe(0)
      expect(setTimeout).toBeCalled()

      jest.runOnlyPendingTimers()

      return localVue.nextTick().then(() => {
        expect(vm.iCount).toBe(1)
      })
    })

    it('点击减1按钮，状态值为-1', () => {
      const btn = wrapper.findAll(Button).at(1)
      expect(btn.props('type')).toMatch(/danger/)
      btn.vm.$emit('click')

      expect(vm.iCount).toBe(0)
      jest.runOnlyPendingTimers()

      return localVue.nextTick().then(() => {
        expect(vm.iCount).toBe(-1)
      })
    })

    it('点击Reset按钮，状态值为0', () => {
      ;(store.state as any).count.count = 20
      const btn = wrapper.findAll(Button).at(2)
      expect(btn.props('type')).toMatch(/primary/)
      btn.vm.$emit('click')

      expect(vm.iCount).toBe(20)
      jest.runOnlyPendingTimers()

      return localVue.nextTick().then(() => {
        expect(vm.iCount).toBe(0)
      })
    })

    it('点击加1按钮，事件方法被调用', () => {
      ;(store.state as any).count.count = 11
      const handleClick = jest.fn(() => {})
      wrapper.setMethods({ handleClick })

      const btn = wrapper.findAll(Button).at(0)
      expect(btn.props('type')).toMatch(/success/)
      btn.vm.$emit('click')

      expect(handleClick).toBeCalled()
      expect(handleClick).toBeCalledWith(12)
    })
  })
})
```

### 常用技巧

- 使用 `shallowMount()` 方法挂载组件时，不渲染其子组件（子组件使用存根代替）
- 使用 `createLocalVue()` 创建一个 `localVue` 来安装全局插件，防止污染全局的 `Vue` 构造函数
- 使用 `stubs` 选项覆写全局或局部注册的组件
- 子组件触发自定义事件 `wrapper.find(子组件).vm.$emit('自定义事件')`
- 触发 dom 事件 `wrapper.trigger('事件')`

### DOM 结构

- `Wrapper.find(选择器)`
  - 返回匹配选择器的第一个 `Wrapper`（DOM 节点或 Vue 组件）
- `Wrapper.findAll(选择器)`
  - 返回一个 `WrapperArray`
- `Wrapper.findAll(选择器).at(序号)`
  - 返回 `WrapperArray` 中的第 `index` 个 `Wrapper`（从 0 开始计数）
- `Wrapper.is(选择器)`
  - 判断 `Wrapper` 是否匹配选择器
- `Wrapper.contains(选择器)`
  - 判断 `Wrapper` 是否包含了一个匹配的选择器
- `Wrapper.exists()`
  - 判断 `Wrapper` 或 `WrapperArray` 是否存在
- `Wrapper.html()`
  - 返回 `Wrapper` DOM 节点的 HTML 字符串

### 选择器

- CSS 选择器
- Vue 组件
- 选项对象
  - `{ name: 'compName' }`
  - `{ ref: 'compRef' }`

### 断言匹配器

- `.toBe` 检查值相等
- `.toEqual` 检查对象的值
- `.toMatch` 检查字符串
- `.toContain` 检查数组
- `.toBeCloseTo` 检查浮点数
- `.toBeCalled` 检查方法被调用（`toHaveBeenCalled` 的别名）
- `.toBeCalledWith` 检查方法被调用时的参数
- `.not` 取反

## Jest

### describe / test(it)

- [describe](https://jestjs.io/docs/zh-Hans/api#describename-fn)

### expect

- [Using Matchers](https://jestjs.io/docs/zh-Hans/using-matchers)
- [expect](https://jestjs.io/docs/zh-Hans/expect#expectvalue)

### js.fn / js.mock

- [Mock Functions](https://jestjs.io/docs/zh-Hans/mock-functions)
- [jest.fn](https://jestjs.io/docs/zh-Hans/jest-object#jestfnimplementation)

### beforeEach / afterEach / ...

- [beforeEach](https://jestjs.io/docs/zh-Hans/api#beforeallfn-timeout)

## 参考

- [Vue Test Utils](https://vue-test-utils.vuejs.org/zh/)
- [Jest 官网](https://jestjs.io/zh-Hans/)
- [Jest 结合 Vue-test-utils 使用的初步实践](https://blog.csdn.net/duola8789/article/details/80434962)
