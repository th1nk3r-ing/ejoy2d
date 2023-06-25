# <font color=#0099ff> **ejoy2d 引擎学习** </font>

> `@think3r` 2023-06-25 17:06:28
>
> 1. <git@github.com:th1nk3r-ing/ejoy2d.git>
> 2. <https://blog.codingnow.com/cgi-bin/mt/mt-search.cgi?IncludeBlogs=1&search=ejoy2d>
>    - [Ejoy2D 开源](https://blog.codingnow.com/2013/12/ejoy2d.html)
>    - [ejoy2d shader 模块改进计划](https://blog.codingnow.com/2014/09/ejoy2d_shader.html)
> 3. [ejoy2d源码阅读笔记1](https://www.cnblogs.com/chaoswong/p/4039509.html)

## <font color=#009A000> 0x00 框架 </font>

- 它只封装了 opengl 的可编程管线（但其实扩展到 DirectX10/11 也很简单），没有做那些华而不实的抽象层。在 RendererBase 之上，又根据 3d engine 的需要设计了 Renderer 一层。
- 在 RendererBase 的层次上，主要是封装显卡驱动级的各种渲染状态切换，创建 shader ，把 attribute layout 抽象出来，和 vertex buffer 一起加以管理。uniform 的管理可以裸露出来，到上一层 Renderer 再说。texture 和 render target 也是在这一层次做封装的。horde3d 很好的把这些东西用自己的 id 管理起来，屏蔽了 opengl 层的 id 。
- 对于 render target ，在移动设备上其实暂时还不需要对 MRT (multiple render target) 做太多支持。因为短期内还看不到在移动设备上使用 Deferred Shading 的实用性。

---

## <font color=#009A000> AI 体育相关 </font>

### <font color=#FF4500> 优化 </font>

1. 而 2d engine 和 3d engine 的区别通常在于 2d engine 的顶点变换很简单。不需要用 projection matrix 和 view matrix 做变换。**2d engine 中的对象多半是四边形，数量很多，常见的优化手法是将大量的四边型合并到同一个渲染批次中；** 所以 world matrix （以平移变换为主）在 CPU 中和顶点计算再提交更常见一些。
2. 2d engine 从应用上说，就是在处理一张张图片。所以对图片（四边型）的处理的变化要多一些。这使得 fs 要多变一点，需要引擎提供一定的可定制性。但很少去处理 3d engine 常见的光照投影这些东西。**更多的是为了优化贴图用量等目的而技巧性的去使用一些图片。**
3. 突出 2d engine 的专门面对的业务的特性，而简化 GPU 提供的模型，用简短的代码构建 engine 框架，是 ejoy2d 设计的初衷。而且我也相信，简单可以带来更好的性能。所以一开始设计 ejoy2d 的时候，shader 模块的很多东西都被写死了，**以最简单的方式达到目的。仅暴露了很少的外部接口，再在这些有限的接口上设计数据结构，做性能优化。**


### <font color=#FF4500> 数据结构抽象 </font>

```c
struct {
    uint32_t id;        // 全局, 唯一标识
    char  pName[32];    // 名字
    uint32_t target;
    int32_t textureId;
    int32_t channels;
    // --------------------- 下方后续支持 ?
    int32_t cnt;  // ????
    float rect[cnt][4]; // ?????
    char * pData; // ?
} Textures;
```
