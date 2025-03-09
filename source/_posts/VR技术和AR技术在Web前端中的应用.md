---
title: VR技术和AR技术在Web前端中的应用
cover_image: /img/cover.png
abbrlink: ecce45ea
date: 2023-04-06 11:46:12
---

虚拟现实（Virtual Reality, VR）技术和增强现实（Augmented Reality, AR）技术在近年来受到越来越多人的关注，它们不仅在游戏和娱乐领域有广泛应用，而且在医疗、教育、设计等领域也发挥了重要作用。随着Web技术的不断发展，VR技术和AR技术也开始在Web前端中得到广泛应用。

### VR技术

VR技术是一种通过计算机生成的虚拟环境来模拟人类感官体验的技术。通过VR设备（如头戴式显示器、手柄等），用户可以在虚拟环境中体验到身临其境的感觉。在Web前端中，VR技术主要应用于WebVR和WebXR两个标准中。

### AR技术

AR技术是一种通过计算机生成的虚拟物体来与现实世界进行融合的技术。通过AR设备（如手机、平板电脑等），用户可以在现实场景中看到虚拟物体，与之进行交互。在Web前端中，AR技术主要应用于WebAR中。

AR种类 1 平面识别类（ex.王者荣耀模型） 基础性功能 大部分都可实现 2 模型识别类（ex. 保时捷换色） 3 图片识别类 （ex.卡牌 书籍）大部分都可实现 4 AR导航 使用AR云 云锚点 储存当前场景的特点（ex. Cyberverse） 5 人脸检测技术 大部分都有 6 Web AR 7 实时光照与反射 8 Apple 另有动作捕捉等高端技术

手机端开发 ARKit（苹果） ARCore（安卓） 实现：官方SDK（资料少；需要熟悉资料；可以调用UI组件）; Unity（比较推荐，入门门槛低）

SenseAR（商汤科技） 优点：免费；有云锚点；有手指动作捕捉；人脸识别支持机型比ARCore丰富；满足硬件即可 缺点：仅支持安卓系统；识别方面不太成熟

华为AR 华为手机不支持ARCore（原因：与谷歌竞争） 优点：免费 缺点：资料少；开发不方便

Vuforia 优点：可以实现识别类以及虚拟点击等功能；图片识别和模型识别非常强大；稳定 缺点：贵（去水印需要20-30万）；平面检测能力很弱

EasyAR（视+） 优点：云锚点非常好；多见于大屏AR，小程序AR和导航 缺点：去水印收费

Void AR（太虚AR） 优点：SLAM效果全国领先，可与Vuforia抗衡 缺点：去水印收费

Web/小程序端 开发 （可分享） AR.js 是坑: 功能弱；兼容性差；需要很多技术：javascript+Web3D等

KIVICUBE（成都 弥知科技） 优点：免费；开发简单；小程序领先 缺点：改域名

眼镜端 微软：MRTK 联想，晨星，Rokid,凉风台等国内AR头显：官方SDK

ARFoundation: 不是底层技术，是一个开发工具集，是封装。它将ARKit ARCore SenseAR集合，只用使用一个API 优点：上手快 缺点：比官方发布要晚一点

### 关于WebGL

WebGL（Web 图形库）是一个 JavaScript API，可在任何兼容的 Web 浏览器中渲染高性能的交互式 3D 和 2D 图形，而无需使用插件

### WevXR

WebXR 是一组支持将渲染 3D 场景用来呈现虚拟世界（虚拟现实，也称作 VR）或将图形图像添加到现实世界（增强现实，也称作 AR）的标准。WebXR 设备 API 实现了 WebXR 功能集的核心，管理输出设备的选择，以适当的帧速率将 3D 场景呈现给所选设备，并管理使用输入控制器创建的运动矢量。

WebXR-兼容性设备包括沉浸式 3D 运动和定位跟踪耳机，通过框架覆盖在真实世界场景之上的眼镜，以及手持移动电话，它们通过用摄像机捕捉世界来增强现实，并通过计算机生成的图像增强场景。

WebXR 设备 API 提供了以下关键功能：

1. 查找兼容的 VR 或 AR 输出设备
2. 以适当的帧率将 3D 场景渲染到设备
3. （可选）将输出镜像到 2D 显示器
4. 创建代表输入控件运动的向量
在最基本的层面上，通过计算应用于场景的透视图，以从每个用户的视角呈现场景，从而在 3D 中呈现场景，考虑到眼睛之间的常规距离，然后渲染场景两次，每只眼睛一次。然后将生成的图像 (场景在一个帧上呈现两次，每只眼睛一半) 显示给用户。

由于 WebGL 用于将 3D 世界渲染到 WebXR 会话中，因此首先应该熟悉 WebGL 的一般用法以及 3D 图形的基本知识。若不会直接使用 WebGL API，可利用在 WebGL 之上构建的框架或库之一来使其使用更加方便。其中最流行的是three.js。

使用库而不是直接使用 WebGL API 的一个特殊好处是，库取向于实现虚拟相机函数性的接口。OpenGL（WebGL 的扩展）不直接提供照相机视图，使用库模拟一个的话可以使工作变得非常非常容易，特别是在构建允许在虚拟世界中自由移动的代码时。

###### WebXR 应用程序生命周期

使用 WebXR 的大多数应用程序将遵循类似的总体设计模式：

检查用户的设备和浏览器是否都能够呈现 XR 体验。确保 WebXR API 可用；如果 navigator.xr 未定义，则可以判断用户的浏览器/设备不支持 WebXR。调用 navigator.xr.isSessionSupported(), 指定要提供的 WebXR 体验模式：inline, immersive-vr, 或 immersive-ar。

当用户通过上述的界面开启了 WebXR 功能后，通过调用 navigator.xr.requestSession()，也是指定使用的模式为以下三种之一：inline, immersive-vr, 或 immersive-ar后，可以将一个 XRSession 设定在期望的模式下。

当 requestSession() 返回的 promise 被 resolve 后，使用新的 XRSession 在整个 WebXR 体验期间运行帧循环。调用 XRSession 的 requestAnimationFrame() 方法，以调度 XR 设备的首帧渲染。

每一个 requestAnimationFrame() 的回调都需要使用 WebGL 渲染已提供信息的 3D 世界中的物体。持续在回调中调用 requestAnimationFrame() 保证每一帧都成功地按顺序渲染。

当需要结束 XR 会话的时候；或者用户主动退出 XR 模式。通过调用 XRSession.end()可手动结束 XR 会话。无论通过何种方式（开发者、用户或者浏览器）终止会话，XRSession 的 end 事件都会接收到通知。

### three.js基本用法

* 创建一个场景(scene)
为了真正能够让一个3d的场景借助three.js来进行显示，我们需要以下几个对象：场景、相机和渲染器，这样我们就能透过摄像机渲染出场景。

```
const scene = new THREE.Scene();const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );const renderer = new THREE.WebGLRenderer();renderer.setSize( window.innerWidth, window.innerHeight );document.body.appendChild( renderer.domElement );
```
three.js里有几种不同的相机，在这里，我们使用的是PerspectiveCamera（透视摄像机）。

第一个参数是视野角度（FOV）。视野角度就是无论在什么时候，你所能在显示器上看到的场景的范围，它的单位是角度(与弧度区分开)。

第二个参数是长宽比（aspect ratio）。也就是你用一个物体的宽除以它的高的值。

接下来的两个参数是近截面（near）和远截面（far）。当物体某些部分比摄像机的远截面远或者比近截面近的时候，该这些部分将不会被渲染到场景中。

接下来是渲染器。这里是施展魔法的地方。要创建一个立方体，我们需要一个BoxGeometry（立方体）对象. 这个对象包含了一个立方体中所有的顶点（vertices）和面（faces）。

```
const geometry = new THREE.BoxGeometry( 1, 1, 1 );const material = new THREE.MeshBasicMaterial(  );const cube = new THREE.Mesh( geometry, material );scene.add( cube );camera.position.z = 5;
```
接下来，对于这个立方体，我们需要给它一个材质，来让它有颜色。第三步，我们需要一个Mesh（网格）。网格包含一个几何体以及作用在此几何体上的材质，我们可以直接将网格对象放入到我们的场景中，并让它在场景中自由移动。默认情况下，当我们调用scene.add()的时候，物体将会被添加到(0,0,0)坐标。但将使得摄像机和立方体彼此在一起。为了防止这种情况的发生，我们只需要将摄像机稍微向外移动一些即可。

* 渲染场景 “渲染循环”(render loop)或者“动画循环”(animate loop)
```
function animate() animate();
```
在这里我们创建了一个使渲染器能够在每次屏幕刷新时对场景进行绘制的循环（在大多数屏幕上，刷新率一般是60次/秒）。requestAnimationFrame有很多的优点。最重要的一点或许就是当用户切换到其它的标签页时，它会暂停，因此不会浪费用户宝贵的处理器资源，也不会损耗电池的使用寿命。

* .add()方法 在threejs中你创建了一个表示物体的虚拟对象Mesh，需要通过.add()方法，把网格模型mesh添加到三维场景scene中。
scene.add(mesh);

### Web 全景

全景图的实现原理：通过创造一个球体/正方体，并在其表面贴上专门的背景图，然后将相机放在球体/正方体的中心，监听手指拖动/陀螺仪移动来改变相机的面向，从而实现全景图使用建立正方体的方法，需要六个面，而摄像头被放在正方体的中央，六面衔接的需要很好，所以对图片素材的要求较高，或者自己拍摄下来进行裁剪也可以,或去一些专门的有vr全景的网站爬https://vr.quanjing.com/

1. 初始化THREE的场景
```
 const scene = new THREE.Scene();    camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 10000);    render = new THREE.WebGLRenderer();    render.setPixelRatio(window.devicePixelRatio);//设置像素比率    render.setSize(window.innerWidth, window.innerHeight)    const app = document.getElementById("app");    app.appendChild(render.domElement);    camera.position.set(200, 0, 0)
```
1. 创建场景贴图
这里就是把之前的图片覆盖到场景之上，让场景的background成为CubeTextureLoader立方体

```
scene.background = new THREE.CubeTextureLoader()    .setPath( '6img/' )      .load( [ 'r.png', 'l.png', 'u.png', 'd.png', 'f.png', 'b.png' ] );
```
CubeTextureLoader：加载CubeTexture的一个类。内部使用ImageLoader来加载文件 3. 因为THREE是在canvas画布上面的，所以要设定时时更新 更新方法：

```
function animate() 
```
1. 全景效果，鼠标应当可以拖动视角
需要THREE.OrbitControls的支持

> Orbit controls allow the camera to orbit around a target. To use this, as with all files in the /examples directory, you will have to include the file seperately in your HTML. 使用方式：
> 
> ```
"js/OrbitControls.js" type="text/javascript" charset="utf-8"></script>...// 鼠标控件  设置在init方法中const controls = new THREE.OrbitControls(camera, render.domElement);...function animate() 
```
### 使用 WebXR Device API 构建增强现实 (AR) 应用

* 准备工作：
1. 使用 Web Server for Chrome 将本地存储的webXRcode文件夹部署到localhost的8889端口，
2. 借助ngrok进行内网穿透，获取公网链接使得别的设备可以访问（也可在google浏览器开发工作站上，通过数据线使得手机等AR设备可以访问localhost:8889
3. 第一次运行 AR 应用时，会有相机权限提示以及通过google商店下载AR-Core
> 重要提示：出于安全考虑，WebXR Device API 只能在安全的 (HTTPS) 环境中运行，但 localhost 开发除外。
> 
> * 配置 WebXR
1. HTML 页面带有 CSS 样式和 JavaScript，用于启用基本的 AR 功能。这样可以加快设置过程的速度,使用现有的网络技术将 AR 体验融入传统网页中。由于可以使用全屏渲染画布，因此 HTML 文件无需过于复杂。AR 功能需要用户通过手势启动，因此一些 Material Design 组件可显示 Start AR（启动 AR）按钮和“Unsupported browser”（浏览器不受支持）消息。需先创立基础的index.html包含必要的dom元素
* 检查 WebXR 和 AR 支持 在用户使用 AR 之前，请先检查是否存在 navigator.xr 和必要的 XR 功能。navigator.xr 对象是 WebXR Device API 的入口点，因此，如果设备兼容，则该对象应该存在。此外，请检查 "immersive-ar" 现场录像模式是否受支持。
1. 如果一切正常，则点击 Enter augmented reality（进入增强现实）按钮会尝试创建 XR 现场录像。否则，系统将调用（shared/utils.js 中的）onNoXRDevice()，用于显示一条指示缺乏 AR 支持的消息。
```
(async function()  else })();
```
> 即使浏览器支持 WebXR Device API，请求的现场录像模式可能也不受支持。例如，桌面浏览器可能会实现该 API，但没有任何已连接的 VR 或 AR 硬件来支持体验。如需详细了解设备枚举，请参阅 WebXR Device API 规范。
> 
> * 请求 XRSession 当点击 Enter augmented Reality（进入增强现实）时，代码会调用 activateXR()。这会启动 AR 体验。
1. 在 app.js 中查找 activateXR() 函数。某些代码已省略：
```
activateXR = async () => 
```
WebXR 的入口点是通过 XRSystem.requestSession() 实现的。使用 immersive-ar 模式(沉浸式体验)允许在现实世界环境中查看渲染的内容。2. 使用 "immersive-ar" 模式初始化 this.xrSession：

```
activateXR = async () => 
```
1. 初始化 XRReferenceSpaceXRReferenceSpace 描述了用于虚拟世界中对象的坐标系。'local' 模式很适合 AR 体验，其参考空间的原点靠近观看器，并且可以进行稳定的跟踪。使用以下代码在 onSessionStarted() 中初始化 this.localReferenceSpace：
```
this.localReferenceSpace = await this.xrSession.requestReferenceSpace("local");
```
* 定义动画循环
1. 使用 XRSession 的 requestAnimationFrame 启动渲染循环，类似于 window.requestAnimationFrame。
> 通过 XRSession 的 requestAnimationFrame，可以接入原生 XR 设备的刷新率。标准网页的渲染循环是针对 60 FPS 设计的，而外部 VR 显示器可能是以 120 FPS 来渲染的。非专有 AR 现场录像仍然只能以 60 FPS 运行，但设备的姿势和视图信息只能在现场录像的 requestAnimationFrame 内访问。
> 
> 在每一帧上，系统使用时间戳和XRFrame 调用 onXRFrame。2. 完成 onXRFrame 的实现。绘制帧时，添加以下代码，将下一个请求加入队列：

```
// Queue up the next draw request.this.xrSession.requestAnimationFrame(this.onXRFrame);
```
3.添加代码以设置图形环境。添加到 onXRFrame 的底部：

```
// Bind the graphics framebuffer to the baseLayer's framebuffer.const framebuffer = this.xrSession.renderState.baseLayer.framebuffer;this.gl.bindFramebuffer(this.gl.FRAMEBUFFER, framebuffer);this.renderer.setFramebuffer(framebuffer);
```
1. 如需确定观看器姿势，请使用 XRFrame.getViewerPose()。此 XRViewerPose 描述设备在空间中的位置和方向。它还包含 XRView 数组，用于描述应渲染场景的每个视角，以便在当前设备上正确显示。立体 VR 有两个视图（每只眼睛各对应一个视图），而 AR 设备只有一个视图。pose.views 中的信息常用于配置虚拟摄像头的视图矩阵和投影矩阵。这会影响场景在 3D 中的布局。配置摄像头后，就可以渲染场景。
```
const pose = frame.getViewerPose(this.localReferenceSpace);if (pose) 
```
#### 添加定位十字线

设置基本的 AR 场景后，就可以使用点击测试开始与现实世界互动了。以下将编写一个点击测试程序，并使用该程序查找现实世界中的表面。点击测试是一种方法，通常用于从空间中的某个点向某个方向投射一条直线并确定该直线是否与任何相关物体相交。在此示例中，将设备对准现实世界中的某个位置。设想有一条光线从您设备的摄像头射出并直接进入前方的物理世界。

WebXR Device API 可让你知道该光线是否与现实世界中的任何物体相交，具体取决于底层 AR 功能以及对世界的理解。

* 使用额外的功能请求 XRSession 为了执行点击测试，请求 XRSession 时需要额外的功能。在 app.js 中，找到 navigator.xr.requestSession。将 "hit-test" 和 "dom-overlay" 功能添加为 requiredFeature，如下所示：
```
this.xrSession = await navigator.xr.requestSession("immersive-ar", );
```
配置 DOM 叠加层。如下所示，在 AR 摄像头视图上叠加 document.body 元素：

```
this.xrSession = await navigator.xr.requestSession("immersive-ar", });
```
添加运动提示 充分理解环境后，ARCore 便可发挥出色的作用。这是通过一个称为同时定位和映射 (SLAM) 的过程实现的，其中视觉上不同的特征点用于计算位置和环境特征的变化。

使用上一步中的 "dom-overlay" 在摄像头信息流顶部显示运动提示。

将 （ID 为 stabilization）添加到 index.html。此 会向用户显示一部表示稳定状态的动画，并提示他们拿着设备四处移动以增强 SLAM 过程。此动画会在用户进入 AR 模式时显示，在十字线发现表面后隐藏起来，具体由 类控制。

```
  <div id="stabilization">div>body>html>
```
* 添加十字线 使用十字线来指示设备的视图所指向的位置。
1. 在 app.js 中，将 setupThreeJs() 中的 DemoUtils.createCubeScene() 调用替换为空的 Three.Scene()。
```
setupThreeJs() 
```
1. 使用表示冲突点的对象填充新场景。提供的 Reticle 类负责在 shared/utils.js 中加载十字线模型。
2. 在 setupThreeJs() 中，将 Reticle 添加到场景：
```
setupThreeJs() 
```
如需执行点击测试，请使用新的 XRReferenceSpace。此参考空间从观看器的视角指示一个新的坐标系，以创建一条与观看方向对齐的光线。XRSession.requestHitTestSource()（可以计算点击测试）中会使用此坐标系。4.  app.js 中的 onSessionStarted()：

```
async onSessionStarted() );  // ...}
```
1. 使用此 hitTestSource，每帧执行一次点击测试：
* 如果点击测试没有结果，则说明 ARCore 没有足够的时间来理解环境。在这种情况下，请使用 stabilization提示用户移动设备。

* 如果有结果，请将十字线移到该位置。
1. 修改 onXRFrame 以移动十字线：
```
onXRFrame = (time, frame) =>   if (hitTestResults.length > 0)   // More code omitted.}
```
* 点按屏幕时添加行为 XRSession 可以通过 select 事件（表示主要操作）根据用户互动发出事件。在移动设备上的 WebXR 中，主要操作是点按屏幕。在 onSessionStarted 底部添加 select 事件监听器：
```
this.xrSession.addEventListener("select", this.onSelect);
```
在此示例中，点按屏幕后，系统会在十字线处放置一朵向日葵。在 App 类中为 onSelect 创建一个实现：

```
onSelect = () => }
```
* 测试应用 创建了一条十字线，可以使用设备通过点击测试对准该十字线。点按屏幕时，应该能够将向日葵放在十字线指定的位置。运行应用时，应该能够看到一条十字线在跟踪地面。如果没有看到，请尝试使用手机缓慢地环顾四周。看到十字线后，请点按它。它上面应该会放置一朵向日葵。可能必须稍微移动一下，以便底层 AR 平台能够更好地检测现实世界中的表面。较差的光线和没有特征的表面会导致场景理解质量降低，找不到点击的可能性也会增加。
* 添加阴影 创建逼真的场景涉及数字对象上适当的照明和阴影等元素，这些元素可以增加场景的真实感和沉浸感。
照明和阴影是由 three.js 处理的。可以指定哪些光线应投射阴影、哪些材料应接收和渲染这些阴影、哪些网格可以投射阴影。此应用的场景包含投射阴影的光线，以及仅用于渲染阴影的平坦表面。

1. 在 three.js WebGLRenderer 上启用阴影。创建渲染程序后，在其 shadowMap 上设置以下值：
```
setupThreeJs() 
```
> 此示例使用阴影的硬编码光线位置。将来，WebXR 的 AR 功能可能会包含光线估计和其他值，以使 3D 模型的阴影也与现实世界同步。如需了解详情，请参阅“功能：WebXR AR 照明估计”。
> 
> 在 DemoUtils.createLitScene() 中创建的示例场景包含一个名为 shadowMesh 的对象，这是一个仅渲染阴影的水平表面。此表面最初的 Y 位置为 10,000 个单位。放置向日葵后，将 shadowMesh 移动到与真实表面相同的高度，从而将花朵的阴影渲染在真实地面上。

1. 在 onSelect 中，向场景添加 clone 后，添加代码以调整阴影平面的位置：
```
onSelect = () => }
```

