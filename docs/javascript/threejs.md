?> 作者：DongChen_Hu ，供稿日期：2020/08/10

可以查看 [在线演示(基于 three:v0.119.1 和 vue:v3.0.0-rc.5)](https://hdcljt.github.io/three-demo) ，代码在 [演示代码仓库](https://github.com/hdcljt/three-demo.git)

> 以下内容是辅助演示的简介，在演示中都有用到，且演示更加详细

## 场景 / 相机 / 渲染器 / 物件

-   场景（容器）
-   相机（取景）
-   渲染器（输出）
-   物件
    -   形状/结构
    -   材质
    -   纹理
    -   光源

## HelloWorld

```ts
// HelloWorld.ts
import { defineComponent, ref, h, onMounted } from 'vue'
import * as THREE from 'three'

export default defineComponent({
    setup() {
        const cvs = ref<HTMLCanvasElement>(null)

        onMounted(() => {
            const scene = new THREE.Scene()

            const camera = new THREE.PerspectiveCamera(75, 2, 0.1, 5)
            camera.position.z = 2

            const geometry = new THREE.BoxBufferGeometry()
            const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 })

            const cube = new THREE.Mesh(geometry, material)

            scene.add(cube)

            const renderer = new THREE.WebGLRenderer({ canvas: cvs.value })

            const animate = () => {
                cube.rotation.x += 0.01
                cube.rotation.y += 0.01

                renderer.render(scene, camera)
            }

            renderer.setAnimationLoop(animate)
        })

        return () => h('canvas', { ref: cvs })
    }
})
```

## 相机 - 透视

模拟人眼所看到的景象

-   构造器

```js
const fov = 45   // 摄像机视锥体垂直视野角度，从视图的底部到顶部，以角度来表示。默认值是50
const aspect = 2 // 摄像机视锥体宽高比，通常是使用画布的宽/画布的高。默认值是1（正方形画布）
const near = 0.1 // 摄像机视锥体近截面（距离），默认值是0.1（个单位）
const far = 5    // 摄像机视锥体远截面（距离），默认值是2000（个单位）
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far)
```

-   更新相机

```js
// 如果改变 fov/aspect/near/far ，需要重新计算投影矩阵（而相机的位置和目标会自动更新）
camera.aspect = canvas.clientWidth / canvas.clientHeight // window.innerWidth / window.innerHeight
// 更新摄像机投影矩阵。在参数被改变以后必须被调用
camera.updateProjectionMatrix()
```

-   将大画布拆分成多个小视口

```js
/**
 *  +---+---+---+
 *  | A | B | C |
 *  +---+---+---+
 *  | D | E | F |
 *  +---+---+---+
 **/
// E
const width = 1920       // 副摄像机的宽度（每个视图的宽）
const height = 1080      // 副摄像机的高度（每个视图的高）
const x = 1 * width      // 副摄像机的水平偏移（视图的起点）
const y = 1 * height     // 副摄像机的垂直偏移（视图的起点）
const fullWidth = w * 3  // 多视图的全宽设置（画布宽）
const fullHeight = h * 2 // 多视图的全高设置（画布高）
// 设置视图偏移量
camera.setViewOffset(fullWidth, fullHeight, x, y, width, height)
// 渲染视图
renderer.render(scene, camera)
// 清除任何由 .setViewOffset 设置的视图偏移量（使渲染恢复正常）
camera.clearViewOffset()
```

## 场景

放置物件、灯光和摄像机的地方

-   初始化

```js
const scene = new THREE.Scene()
// 设置背景（可以是 Color 或 Texture），默认值为null
scene.background = new THREE.Color('white')
const color = 'lightblue' // 雾的颜色（16进制整型、css风格字符串）
const near = 1  // 开始应用雾的最小距离，距离活动相机小于“near”个单位的物体不会受到雾的影响。默认值为1
const far = 2   // 计算并应用雾化停止的最大距离。距离活动相机超过“far”个单位的物体不会受到雾的影响。默认值为1000
// 影响场景中的每个物体的雾效果，默认值为null
scene.fog = new THREE.Fog(color, near, far)
```

-   场景中添加物体、灯光、相机

```js
scene.add(mesh)
scene.add(light)
scene.add(camera)
```

## 物体 - 几何体

```js
// 形状-立方体
const width = 1  // X轴上面的宽度，默认值为1
const height = 1 // Y轴上面的高度，默认值为1
const depth = 1  // Z轴上面的深度，默认值为1
const widthSegments = 1  // （可选）宽度的分段数，默认值是1
const heightSegments = 1 // （可选）宽度的分段数，默认值是1
const depthSegments = 1  // （可选）宽度的分段数，默认值是1
const geometry = new THREE.BoxBufferGeometry(
    width,
    height,
    depth,
    widthSegments,
    heightSegments,
    depthSegments
)
// 材质-不受光照的影响
const color = 0xffffff // 材质的颜色，默认值为白色 (0xffffff)
const material = new THREE.MeshBasicMaterial({ color })
// 网格-基于三角形组成的多边形网格的物体。
// Mesh: geometry-默认值是一个新的BufferGeometry，material-默认是一个新的MeshBasicMaterial(颜色随机)
const mesh = new THREE.Mesh(geometry, material)
```

## 物体 - 3D 模型

```js
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
const gltfLoader = new GLTFLoader()
gltfLoader.load('models/mountain_landscape/scene.gltf', gltf => {
    scene.add(gltf.scene)
})
```

## 纹理 - 图片

```js
const loader = new THREE.TextureLoader()
const material = new THREE.MeshBasicMaterial({
    map: loader.load('textures/cube/neg-x.jpg')
})
const geometry = new THREE.BoxBufferGeometry()
const mesh = new THREE.Mesh(geometry, material)
```

```js
const loader = new THREE.CubeTextureLoader()
const texture = loader.setPath('textures/cube/').load([
    'pos-x.jpg', // +x
    'neg-x.jpg', // -x
    'pos-y.jpg', // +y
    'neg-y.jpg', // -y
    'pos-z.jpg', // +z
    'neg-z.jpg'  // -z
])
const scene = new THREE.Scene()
scene.background = texture
const material = new THREE.MeshBasicMaterial({
    envMap: texture
})
const mesh = new THREE.Mesh(
    new THREE.SphereBufferGeometry(0.4, 32, 16),
    material
)
scene.add(mesh)
```

## 纹理 - 视频

```js
const geometry = new THREE.SphereBufferGeometry(500, 60, 40)
const texture = new THREE.VideoTexture(video.value)
const material = new THREE.MeshBasicMaterial({ map: texture })
const mesh = new THREE.Mesh(geometry, material)
const camera = new THREE.PerspectiveCamera(75, 2, 1, 1100)
camera.position.set(0, 10, 20)
```

## 光源 - 平行光

用平行光模拟太阳光的效果

```js
const color = 0xffffff // 用16进制表示光的颜色。 默认值为 0xffffff (白色)
const intensity = 1    // 光照的强度。默认值为1
const light = new THREE.DirectionalLight(color, intensity)
```

投射阴影会增加渲染次数。一种常见的解决方案是使用多个灯光，但只有一个平行光产生阴影

```js
renderer.shadowMap.enabled = true // 开启阴影贴图
light.castShadow = true   // 光源投射阴影
mesh.castShadow = true    // 物体投射阴影
mesh.receiveShadow = true // 物体接收阴影
```

## 渲染器

-   初始化

```js
const canvas = document.querySelector('canvas')
const renderer = new THREE.WebGLRenderer({
    canvas,          // 一个供渲染器绘制其输出的canvas，和domElement属性对应。如果没有传会创建一个新canvas
    alpha: false,    // canvas是否包含alpha (透明度)。默认为 false
    antialias: false // 是否执行抗锯齿。默认为 false
})
renderer.domElement // 一个canvas，渲染器在其上绘制输出（对应于构造器传入的canvas）
renderer.shadowMap.enabled = false   // 如果设置开启，允许在场景中使用阴影贴图。默认是 false
renderer.xr.enabled = false // 该标志通知渲染器准备进行XR渲染。默认值为 false
renderer.setClearColor(0xffffff, 1.0) // 设置颜色及其透明度
```

-   渲染场景

```js
const render = () => {
    // 用相机(camera)渲染一个场景(scene)
    renderer.render(scene, camera)
}
```

-   循环渲染

```js
// 可用来代替requestAnimationFrame的内置函数。对于WebXR项目，必须使用此函数（不能使用raf）
renderer.setAnimationLoop(render)
// 停止所有正在进行的动画
renderer.setAnimationLoop(null)
```

-   调整输出画布的大小

```js
const canvas = renderer.domElement
const pixelRatio = window.devicePixelRatio
const width = (canvas.clientWidth * pixelRatio) | 0
const height = (canvas.clientHeight * pixelRatio) | 0
const needResize = canvas.width !== width || canvas.height !== height
if (needResize) {
    // 将输出画布的大小调整为给定的宽高度，同时考虑设备像素比率，并将视口从(0, 0)开始调整为适合的大小
    // 将updateStyle设置为false，可防止对输出画布进行任何样式更改
    renderer.setSize(width, height, false)
}
```

-   渲染一个目标缓冲

```js
const rtWidth = 512  // 渲染目标宽度
const rtHeight = 512 // 渲染目标高度
const renderTarget = new THREE.WebGLRenderTarget(rtWidth, rtHeight, {
    depthBuffer: false,  // 渲染到深度缓冲区。默认是 true
    stencilBuffer: false // 渲染到模板缓冲区。默认是 true
})
// 设置需要被激活的渲染目标为renderTarget
renderer.setRenderTarget(renderTarget)
// 渲染renderTarget
renderer.render(rtScene, rtCamera)
// 将画布设置成活动渲染目标（参数为空）
renderer.setRenderTarget(null)
// 渲染画布
renderer.render(scene, camera)
```

-   剪裁视口

```js
// 禁用
renderer.setScissorTest(false)
// 告诉渲染器清除颜色、深度或模板缓存。此方法将颜色缓存初始化为当前颜色。默认都是true
const color = true
const depth = true
const stencil = true
renderer.clear(color, depth, stencil)
// 启用剪裁检测（只有在所定义的裁剪区域内的像素才会受之后的渲染器影响）
renderer.setScissorTest(true)
// 将剪裁区域设为(x, y)到(x + width, y + height)，(x, y)是剪裁区域的左下角
renderer.setScissor(x, y, width, height)
// 将视口大小设为(x, y)到(x + width, y + height)，(x, y)是视口区域的左下角
renderer.setViewport(x, y, width, height)
// 渲染场景
renderer.render(scene, camera)
```

## 后处理

EffectComposer 类管理了产生最终视觉效果的后期处理过程链。  
后期处理过程根据它们添加/插入的顺序来执行，最后一个过程会被自动渲染到屏幕上

-   初始化

```js
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer'
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass'
import { BloomPass } from 'three/examples/jsm/postprocessing/BloomPass'
import { FilmPass } from 'three/examples/jsm/postprocessing/FilmPass'
// 每一个Pass都有继承了一些公共属性，例如：
// enabled 是否启用该过程
// renderToScreen 是否渲染到屏幕（画布）
const bloomPass = new BloomPass(
    1,  // strength
    25, // kernel size
    4,  // sigma
    256 // blur render target resolution
)
// bloomPass.enabled = false // 是否使用此处理过程
const filmPass = new FilmPass(
    0.35,  // noise intensity
    0.025, // scanline intensity
    648,   // scanline count
    false  // grayscale
)
// filmPass.renderToScreen = true // 后处理结束
```

-   过程链

```js
// 效果合成器将创建两个渲染目标（rtA,rtB），可以添加多个过程对象
const composer = new EffectComposer(renderer)
// 首先，将原相机和场景作为第一个渲染目标
composer.addPass(new RenderPass(scene, camera))
// 添加模糊效果
composer.addPass(bloomPass)
// 绘制噪音和扫描线
composer.addPass(filmPass)
```

-   渲染

```js
// 执行所有启用的后期处理过程
composer.render(deltaTime)
```

## 拾取对象

-   初始化

```js
const raycaster = new THREE.Raycaster()
const coords = new THREE.Vector2()
```

-   设备坐标

```js
const onMouseMove = event => {
    const canvas = event.target
    const { left, top, width, height } = canvas.getBoundingClientRect()
    // 将鼠标位置归一化为设备坐标。x 和 y 方向的取值范围是 (-1 to +1)
    coords.x = ((event.clientX - left) / width) * 2 - 1
    coords.y = -((event.clientY - top) / height) * 2 + 1
}
```

-   计算焦点

```js
// coords 在标准化设备坐标中鼠标的二维坐标（X分量与Y分量应当在-1到1之间）
// camera 射线所来源的摄像机
raycaster.setFromCamera(coords, camera)
// scene.children 检测和射线相交的一组物体
const intersects = raycaster.intersectObjects(scene.children)
if (intersects.length) {
    // 突出显示第一个对象（最近的一个）
    intersects[0].object.material.emissive.setHex(0xff0000)
}
```

## WebVR

```js
import { VRButton } from 'three/examples/jsm/webxr/VRButton'
// 初始化
const renderer = new THREE.WebGLRenderer({ canvas: cvs.value })
renderer.xr.enabled = true
vrBtn = VRButton.createButton(renderer)
document.body.appendChild(vrBtn)
// 渲染
const render = () => {
    renderer.render(scene, camera)
}
renderer.setAnimationLoop(render)
// 销毁
renderer.setAnimationLoop(null)
document.body.removeChild(vrBtn)
```

## 控制器

-   OrbitControls 轨道控制器
-   TrackballControls 轨迹球控制器
-   DragControls 拖放控制器
-   DeviceOrientationControls （移动）设备方向控制器

## 辅助参考系

-   AxesHelper 坐标轴
-   GridHelper 网格
-   CameraHelper 相机
-   DirectionalLightHelper 平行光

## 补充

-   角度与弧度的转换

```
弧度 = 角度 * Math.PI / 180
```

-   根据距离求可见高度

```
tanα = ∠α 对边 / ∠α 邻边

可见高度 = Math.tan(相机角度 / 2 * Math.PI / 180) * 与相机的距离 * 2

相机距离 = 高度 / 2 / Math.tan(相机角度 / 2 * Math.PI / 180)
```

## 参考

1.  [three.js（r119）](https://threejs.org/)
2.  [Threejs Fundamentals（r115）](https://threejsfundamentals.org/)
3.  [Threejs 中文网（r92）](http://www.webgl3d.cn/Three.js/)
4.  [WebGL 中文网（r73）](http://www.hewebgl.com/article/articledir/1)
