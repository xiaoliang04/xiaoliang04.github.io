---
layout: mypost
title: 第一个shader helloworld
categories: [xiaoliang]
---

记录一下第一个shader

------

# unity shader 初体验：简单的shader实例

## 简介

跟着b站ub主庄懂，了解了unity shader的一些基础渲染知识。这是我学习unity shader编程时第一个尝试，通过这个例子，我希望能够掌握Unity shader的基本结构和工作原理。虽然功能非常基础，但它为我后续的shader开发奠定了基础/

再这篇博客中，我将分享这个简单的shader，并逐步解释它的每一部分，希望能够帮助那些刚开始接触unity shader的朋友们。

## 内容

### shader概览

简单解释一下shader的作用，shader（着色器）是计算机图形学中的重要组成部分，它辅助控制物体的渲染外观，决定物体的颜色、光照效果以及材质反应等等。在游戏开发中，shader常被用于优化视觉效果，并让物体在不同的光照和场景下表现得更真实。

我编写的这个shader目标非常明确：让物体显示出固定的颜色，尽管它并没有复杂的光照处理，但是通过它可以了解shader的核心结构。

## 基础渲染过程图

![再](https://liangx.work/assets/渲染管线.png)

------

向量含义

![再](https://liangx.work/assets/向量.png)

------

### 完整代码示例

以下是我根据教程编写的第一个unity shader，命名为“shader/helloworld"。这个shader使用unity的内置函数，来处理顶点位置和法线，并最终为物体着色。

```
Shader "shader/helloworld" {
    Properties {
    }
    SubShader {
        Tags {
            "RenderType"="Opaque"
        }
        Pass {
            Name "FORWARD"
            Tags {
                "LightMode"="ForwardBase"
            }
            
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            #include "UnityPBSLighting.cginc"
            #include "UnityStandardBRDF.cginc"
            #pragma multi_compile_fwdbase_fullshadows
            #pragma target 3.0
            // 输入结构
            struct VertexInput {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };
            // 输出结构
            struct VertexOutput {
                float4 pos : SV_POSITION;     //由模型顶点信息换算而来的顶点屏幕位置
                float3 normalDir : TEXCOORD;  //由模型法线信息换算而来的世界空间法线信息
            };
             // 输入结构>>>顶点Shader>>>输出结构
            VertexOutput vert (VertexInput v) {
                VertexOutput o = (VertexOutput)0;                               // 新建一个输出结构
                o.normalDir = UnityObjectToWorldNormal(v.normal);               // 变换法线信息 并将其塞给输出结构  
                o.pos = UnityObjectToClipPos( v.vertex );                       // 变换顶点信息 并将其塞给输出结构
                return o;                                                       // 变换法线信息 并将其塞给输出结构
            }
             // 输出结构>>>像素
            float4 frag(VertexOutput i) : COLOR {
                return float4(0.5,0.5,1,1.0);                                 //输出最终颜色   
            }
            ENDCG
        }
    }
    FallBack "Diffuse"
}

```

------

### 代码详解

**1.Properties**

```
Properties {
}

```

在这个unity shader中，`properties`用于定义Shader参数，比如颜色、贴图等。在这个shader中，我并没有定义任何参数，因为这是一个最基础的shader，所以它是空的。

**2.SubShader和Tags**

```
SubShader {
    Tags {
        "RenderType"="Opaque"
    }

```

`Subshader`是Shader的核心部分，它决定了如何绘制物体。`Tags`中的`RenderType=Opaque`表示这个物体是不透明的。这是最常见的渲染类型。

**3.Pass和LightMode**

```
Pass {
    Name "FORWARD"
    Tags {
        "LightMode"="ForwardBase"
    }

```

`pass`指定了渲染的阶段，一个shader可以有多个pass来完成不同的效果。这里的`LightMode`设置为`Forwardbase`，意味着使用前向渲染模式。这是unity中的一种常用的光照计算模式。

**4.CGPROGRAM部分**

```
CGPROGRAM
    #pragma vertex vert
    #pragma fragment frag
    #include "UnityCG.cginc"
    #include "UnityPBSLighting.cginc"
    #include "UnityStandardBRDF.cginc"
    #pragma multi_compile_fwdbase_fullshadows
    #pragma target 3.0

```

`CGPROGRAM`块是我们编写实际着色器代码的地方。这里指定了两个主要函数：`vert`是顶点着色器，`frag`是像素着色器。

**·**`#pragma vertex vert`:指明哪个函数处理顶点数据。

·`#pragma fragment farg`：指明哪个函数处理像素数据。

·`#include`:导入Unity的标准库函数，这些库函数包含了光照、矩阵变换等功能。

·`#pragma target 3.0`:指定了shader的目标平台，这里使用的是shader model 3.0。

**5.顶点着色器 vert**

```
VertexOutput vert (VertexInput v) {
    VertexOutput o = (VertexOutput)0;
    o.normalDir = UnityObjectToWorldNormal(v.normal);  
    o.pos = UnityObjectToClipPos( v.vertex );
    return o;
}

```

顶点着色器`vert`接收输入结构`vertexInput`，包含顶点位置和法线。主要有两个步骤：

·将法线从模型空间转换到世界空间，存储在`o.normalDir`中，`o.normalDir`存储的是每个顶点的世界空间法线。

·使用unity的`unityobjecttoclipPos`函数，将顶点位置转换到裁剪信息，并将结果存储在`o.pos`中。

**6.像素着色器 farg**

```
float4 frag(VertexOutput i) : COLOR {
    return float4(1.0, 0.5, 0.5, 1.0);
}

```

片段着色器`frag`接收顶点着色器的输出作为输入。这里返回了一个固定的颜色值——浅红色（`float4（1.0，0.5，0.5，1.0）`），该颜色会应用到物体上。

![shader](https://liangx.work/assets/helloworld.png)

### 总结

通过这个简单的shader，我学习并掌握了unity shader 的基础结构，包括如何处理顶点和片段，以及基本的光照效果和颜色输出。在实际应用中，shader只会更复杂，包括更多的动态效果、光照处理以及贴图处理。

尽管这是一个非常基础的shader，但是通过它我认识到了shader的大致结构，每个模块的意义。

------

此次博客是作者第一次写，在chatGPT的帮助下成功完成书写，希望我自己以后会有更多的技术分享博客。
