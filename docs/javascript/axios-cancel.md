?> 作者：SunJian ，供稿日期：2020/06/19


## 需求说明
* 在开发中，经常会遇到接口重复请求导致的各种问题。
	1. 对于重复的Get请求，会导致页面更新多次，发生页面抖动的现象，影响用户体验。

	2. 对于重复的Post请求，会导致在服务端生成两次记录。
* 请求的响应时间存在不确定性，请求次数过多时，有可能较早发起的请求会较晚响应。
* 如果当前页面请求还未响应完成，就切换到了下一个路由，那么这些请求直到响应返回才会中止，可能会影响下个页面的显示或者影响页面性能。

无论从用户体验或者从业务严谨方面来说，取消无用的请求确实是很有必要的。

## 使用方式
`axios`是一个主流的http请求库，它提供了两种取消请求的方式，具体可以参考如下代码：

1. 通过`axios.CancelToken.source`生成取消令牌`token`和取消方法`cancel`


```javascript
const source = axios.CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function(thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // handle error
  }
});

axios.post('/user/12345', {
  name: 'new name'
}, {
  cancelToken: source.token
})

// cancel the request (the message parameter is optional)
source.cancel('Operation canceled by the user.');
```

2. 通过`axios.CancelToken`构造函数生成取消函数

```javascript
const CancelToken = axios.CancelToken;
let cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // An executor function receives a cancel function as a parameter
    cancel = c;
  })
});

// cancel the request
cancel();
```

## 源码实现

```javascript
// axios/lib/cancel/CancelToken.js
CancelToken.source = function source() {
  var cancel;
  var token = new CancelToken(function executor(c) {
    cancel = c;
  });
  return {
    token: token,
    cancel: cancel
  };
};
```
`cancelToken.source()`是一个工厂函数，会返回两个属性，一个`token`和`cancel`函数，该函数在被调用时，将执行取消请求操作。

这两个属性由函数`CancelToken `构造而成，可以看看`CancelToken `里面的代码：

```javascript
// axios/lib/cancel/CancelToken.js
function CancelToken(executor) { // 参数executor

  // 参数类型判断为函数
  if (typeof executor !== 'function') {
    throw new TypeError('executor must be a function.');
  }
  
  var resolvePromise;
  // 实例挂载一个promise，这个promise会在变量resolvePromise执行后resolved
  this.promise = new Promise(function promiseExecutor(resolve) {
    resolvePromise = resolve;
  });


  var token = this; // 实例即token
  // 执行器执行，将函数cancel传递到外界
  executor(function cancel(message) {
    if (token.reason) {
      // 如果token挂载了reason属性，说明该token下的请求已被取消
      return;
    }
    // token挂载reason属性
    token.reason = new Cancel(message);
    // 外界可以通过执行resolvePromise来将该token的promise置为resolved
    resolvePromise(token.reason);
  });
}
```

`promise resolved` 做的操作：

```javascript
// axios/lib/adapters/xhr.js
    if (config.cancelToken) {
      // 请求时带上了cancelToken，如果上面token的promise resolve就会执行取消请求的操作
      config.cancelToken.promise.then(function onCanceled(cancel) {
        if (!request) {
          return;
        }

        request.abort();
        reject(cancel);
        // Clean up request
        request = null;
      });
    }

```


## 项目中代码实现
我们这里选择第二种方法实现。

以上所说的取消请求的原因可以概括为两个场景：

* 同一个请求发送了多次，需要取消之前的请求

* 当路由切换时，需要取消上个路由中未完成的请求

### 封装CancelToken

```javascript
import Axios from 'axios'

// 声明一个 Map 用于存储每个请求的标识 和 取消函数
const pending = new Map()

/**
 * 添加请求
 * @param {Object} config
 */
export const addPending = (config) => {
    const { method, url, params, data } = config
    const requestUrl = [method, url, JSON.stringify(params), JSON.stringify(data)].join('&')
    config.cancelToken = config.cancelToken || new Axios.CancelToken(cancel => {
        if (!pending.has(requestUrl)) { // 如果 pending 中不存在当前请求，则添加进去
            pending.set(requestUrl, cancel)
        }
    })
}
/**
 * 移除请求
 * @param {Object} config
 */
export const removePending = (config) => {
    const { method, url, params, data } = config
    const requestUrl = [method, url, JSON.stringify(params), JSON.stringify(data)].join('&')
    if (pending.has(requestUrl)) { // 如果在 pending 中存在当前请求标识，需要取消当前请求，并且移除
        const cancel = pending.get(requestUrl)
        cancel(requestUrl)
        pending.delete(requestUrl)
    }
}
/**
 * 清空 pending 中的请求（在路由跳转时调用）
 */
export const clearPending = () => {
    for (const [url, cancel] of pending) {
        cancel(url)
    }
    pending.clear()
}

```

### axio拦截器中使用

```javascript
axios.interceptors.request.use(config => {
  removePending(config) // 本次请求前，将之前相同的请求取消掉
  addPending(config) // 将当前请求添加到 pending 中
  // ...
  return config
}, error => {
  return Promise.reject(error)
})

axios.interceptors.response.use(response => {
  removePending(response) // 在请求结束后，移除本次请求
  // ...
  return response
}, error => {
  if (axios.isCancel(error)) {
    console.log('repeated request: ' + error.message)
  } else {
    // handle error code
  }
  return Promise.reject(error)
})

```

### 路由跳转处理
```javascript

router.beforeEach((to, from, next) => {
  clearPending()
  // ...
  next()
})

```


	

