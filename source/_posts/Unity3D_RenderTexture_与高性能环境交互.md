---
title: 'Unity3D: RenderTexture 与高性能环境交互'
cover_image: /img/cover.png
abbrlink: 3c87549d
date: 2024-06-03 08:04:18
---

![](https://mmbiz.qpic.cn/mmbiz_jpg/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDxXXHPts996ux9ricKyzv4icQmu69WXglWj5Ry2C6pH5B73eAYdCic1z4Cg/640?wx_fmt=jpeg&from=appmsg)# 环境交互？

请想象你正在扮演游戏里的一位英雄，踏上冒险的旅途——

你跨越草原，野草与鲜花伴随你的脚步左右摇，你横穿沙漠，漫天黄沙伴你的身形漫天飞舞。

野草，鲜花，黄沙，他们都随着游戏主角的到来与离去而产生扰动，为游戏增添了沉浸感与生命力。此为环境交互。

# 最简单的尝试：触发器

碰撞体触发器是最容易被想到的实现方式。它允许开发者在游戏角色进入或退出某个区域时收到消息，进而触发控制代码。

让我们创建一个随玩家的移动而摇摆的2D花朵为例。在玩家进入和退出时，都将会刷新一个定时器，花朵会在定时器有效时持续摆动，并在定时器结束后停止摆动，以表现玩家的移动对其产生影响。

```
[RequireComponent(typeof(Collider2D))]public class Flower : MonoBehaviour    private void Update()        }    private void OnTriggerEnter2D(Collider2D other)        private void OnTriggerExit2D(Collider2D other)    }
```
我们确实完美地实现了功能，这很好——但是还不够。我们现在拥有了一朵可以摆动的花，但为了产生良好的视觉效果，我们可能需要更多的花，聚集在一起——或许需要一片花海？

在这种情况下，我们需要做出一些取舍：

* 将单个花朵堆叠，以达成大量花朵的效果。
* 优点：每一朵花都可以独立摆动，效果更加真实优雅
* 缺点：大量的触发器占用较多的性能，产生浪费
* 将一大丛花朵视为整体，检测碰撞
* 优点：性能消耗相对较低
* 缺点：对玩家位置的敏感程度降低，且花朵需要额外逻辑控制方能独立摆动
然而，上述两种方法都无法逃脱物理模拟，即占用一部分CPU资源进行处理。在规模相对较大的交互中，物理模拟无遗是低效的。

# 改变思路：让GPU干活

环境交互本质上是环境布置的延伸，是在画面表现上做出的优化，不应当将用于游戏具体逻辑的宝贵CPU资源出让。因此，借助于GPU的并行运算能力，是一种更有效率的解决方案。我们应当重新审视遇到的问题。

## 随“风”摇摆，“风”是什么？

主角奔跑着越过花丛，花丛随风摇摆。这所谓的“风”的本质是一个影响范围，其影响最强的地方总是位于运动中的主角四周，并随着时间逐渐减弱。在这一影响范围内，环境相应地表现出受到影响的样子。

这一影响范围在GPU中的表达，可以是一张利用颜色来描述影响范围的贴图，再利用着色器，通过对这一张贴图采样来确定某个位置受到多大的影响，最终实现依据影响采样。

那么显然，这一张贴图并不能渲染上主角或环境物体，而是应当由一套专用于描述影响范围的物体进行渲染。基于这一特性，我们自然想到可以使用摄像机的层级遮罩来实现只渲染指定层级的物体。

## 先把功能搭起来！

让我们创建新层级 Trail ，并用一张简单的圆形贴图作为主角的影响范围，将其放置于主角的子物体，并调整层级：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDxsIXbnZDLnhiaicde00WZ84MIfp99E3r9772wC6BVx08bO5UDO3FicpRxA/640?wx_fmt=png&from=appmsg)接下来，创建一个专门用于渲染这些范围的摄像机，将其层级调整为 Trail。与此同时，将主摄像机的 Trail 层级取消勾选：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDx2asSLNkxUvhJVTz2m4mKuDVSpQV0WZia74gJPYYBR20wtLic4sKu2OrA/640?wx_fmt=png&from=appmsg)为了防止 Trail 摄像机渲染天空盒造成干扰，需要对 Environment 项的 Background 进行调整，改为 Alpha 为 0 的纯黑色：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDxWKFWZbAoBU31nvdicZU1Ke69yh8uvrdsicHweU6kLrf9kM7etZBFHiaVQ/640?wx_fmt=png&from=appmsg)现在，我们已经将摄像机的结构建立完毕了。让我们继续实现功能！

我们并不希望 Trail 摄像机将输出直接渲染到屏幕上，而是交给着色器进行调用。因此，需要将一张 RenderTexture  作为摄像机的渲染目标，并使用 Shader.SetGlobalTexture(string, Texture) 将其送给着色器进行处理。由此，编写如下脚本并挂载到 Trail 摄像机上：

```
[RequireComponent(typeof(Camera))]public class TrailCamera : MonoBehaviour    private void OnDestroy()    }
```
让我们来验证一下这一功能是否在正常工作！编写一个 ShaderGraph，直接输出该 RenderTexture 的内容。

> 如果你在使用 URP 2D 管线，则可创建一个 Sprite Lit/Unlit Graph
> 
> ![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDxAY0sCuWbibThQ0f7Orq2vVtIRLnOiaYhibqRIhpEExojzia6Sfp1pRt0EQ/640?wx_fmt=png&from=appmsg)请注意：输入的贴图

* Reference 项需要与上方代码中 SetGlobalTexture 给定的字符串一致
* Exposed 项需要取消勾选。
* 由于 Trail 摄像机将会与主摄像机保持一致的位置和缩放，所以应当使用屏幕坐标对 TrailTex 贴图进行采样
在场景内任意创建一个矩形贴图，将 ShaderGraph 自动产生的材质拖拽给它：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDxcgJ6UoHMcCJ8j3eGFRic3T9YQVuxxr18rLXxBtf2OVMn9sJ6ibysqbPw/640?wx_fmt=png&from=appmsg)进行测试，发现这张贴图完美地渲染出了 Trail 层级的物体：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDxHMkthRCyuYLxlVw60Ampu0XEI4AhIRckNxPakiblEwEOJsXTUMkmdAw/640?wx_fmt=png&from=appmsg)表明我们已经成功地获取到了 TrailTex 贴图传递的信息。接下来，只需要利用对 TrailTex 的采样，通过着色器实现不同强度的贴图摆动，即可实现功能！

## 优化那个圆

在上述的例子中，我们使用了一个简单的圆形贴图来作为影响范围，但这并不是实际期望的结果。实际的功能需求应当是仅在主角移动的时候产生影响，且这个影响会随着时间消散。

有没有哪一种渲染器，可以仅在移动时渲染物体，并随时间逐渐消散呢？

TrailRenderer: 轨迹渲染器！将渲染圆形的 SpriteRenderer 替换为 TrailRenderer, 并适当调整参数，可以得到这样的效果：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDxeUoeOFVzibDN6ZACfDl8vIymhSIoJmJFXibkjAmibhDxUMZNs4s8oDdfg/640?wx_fmt=png&from=appmsg)这与我们希望的效果很接近了！红色浓重的地方环境交互更加剧烈，而减淡则表示环境交互逐渐消失，这符合预期中，主角走路带起的风越来越弱的结果。

然而，TrailRenderer 本身仍然存在一部分弊端。最为明显的是，存在透明度渐变时，由于 UV 拉扯，在转角处常见较为尖锐的转折线（在上图中亦有所体现），这会导致环境交互不平滑。

还有替代品吗？ParticleSystem!ParticleSystem 粒子系统并非专用于实现尾迹的，但借助其按照移动距离发射粒子的特性，我们可以实现更加平滑的“伪拖尾”。“伪拖尾”需要的核心配置如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDx107ERgPNKGyULJItwmsQXSj6qmPVsSeziaH1ZEicDEERRCFNBxaqRKZg/640?wx_fmt=png&from=appmsg)在适当调整颜色随时间的变化（主要调整 Alpha 值）后，我们可以达成这样的效果：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDxrjicraGuLiaxYnTQxhREGe3MhPwdibuBuclbKW8q76V5UOS4C9ic0qIh3g/640?wx_fmt=png&from=appmsg)在这张图中，可以看出拖尾变得更加平滑，但在尾部仍具有瑕疵。这是因为使用了 Unity 内置的粒子贴图，而该贴图具有少量的黑边导致的。若使用自行绘制的、仅具有白色且中心向四周渐变的贴图，则可以避免这种情况。

大功告成！

# 小结

在实现与画面有关的效果时，如果规模较大，则应当将其处理交由 GPU 进行。为了实现与 GPU 的交互，可以使用 RenderTexture 配合 Camera 来实现，并最终通过着色器来获取信息。

借助于上述方法以及拖尾渲染的实现，我们可以将游戏角色走路、奔跑时产生的影响范围转换为贴图，交由 GPU 借助着色器实现场景元素的交互，增加游戏的沉浸感与真实性。

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8QX2kbw3pM8uQSFvVUUEVDxBseWw2dG4VcrQYCSaLXGY8kw5icLXNIkJuZaEswEqvhXRgXQCGEibJibQ/640?wx_fmt=png&from=appmsg)

