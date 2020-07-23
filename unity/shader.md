# Shader

#### 渲染管线
- **Shader按管线分类**一般分为固定渲染管线与可编程渲染管线
　　(1)**固定渲染管线** ——这是标准的几何&光照(Transforming&Lighting)管线，功能是固定的，它控制着世界、视、投影变换及固定光照控制和纹理混合。T&L管线可以被渲染状态控制，矩阵，光照和采制参数。功能比较有限。基本所有的显卡都能正常运行。
　　(2)**可编程渲染管线**——对渲染管线中的顶点运算和像素运算分别进行编程处理，而无须象固定渲染管线那样套用一些固定函数，取代设置参数来控制管线。

- **unity3d的三种Shader**
　　(1)**Fixed function shader** 属于固定渲染管线 Shader, 基本用于高级Shader在老显卡无法显示时的Fallback(之后有详细介绍)。使用的是ShaderLab语言，语法与微软的FX files 或者NVIDIA的 CgFX类似。
　　(2)**Vertex and Fragment Shader** 最强大的Shader类型，属于可编程渲染管线. 使用的是CG/HLSL语法。
　　(3)**Surface Shader** Unity3d推崇的Shader类型，使用Unity预制的光照模型来进行光照运算。使用的也是CG/HLSL语法。

