---
title: "3.8 特效"
keywords: sample homepage
tags: [getting_started]
sidebar: rtr
layout: book
permalink: rtr_ch3_effects.html
summary: GPU
---

### 3.8 特效
目前为止，我们一直关注的是流水线的各个不同的可编程阶段。我们使用顶点、几何以及像素着色器程序来控制这些阶段，他们并不是凭空产生的。首先，一个独立隔绝的着色程序不会很有用：顶点着色器会把他的结果传递给像素着色器。任何一个工作，都需要这两个着色器程序加载后来完成。程序员需要保证顶点着色器的输出和像素着色器的输入相匹配。一个渲染特效可以任意数量的着色器参与，通过执行一些`pass`来完成。对于这些着色器本身，有些时候需要设定一些特定的渲染状态配置才能让他们正常工作。比方说，渲染状态包括Z缓冲和模板缓冲是否使用、如何使用，以及片元与当前颜色值如何作用（替换、相加或混合）。

因为这些原因，多个组织发展多个不同的特效语言，比如<font color="limegreen">HLSL FX, CgFX 和 COLLADA FX</font>。一个特效文件就是，执行指定渲染算法所需要的所有相关的信息的集合。它通常会定义一些全局变量以便应用程序来访问。比方说，一个特效文件可能会定义可信的塑料材质所需要的顶点着色器和像素着色器需要的数据。它可能会开放塑料颜色和光泽度等参数，这样不同的模型可以设置不同的参数，然后使用同一个特效文件来渲染。

为了感受下特效文件，我们会演示一个精简的例子，它是从NVIDIA的 FX Composer 2特效系统中摘取出来的。这个DX9的特效文件实现了一个非常简单的`古氏着色`（Gooch shading)。古氏着色的一部分是使用表面法线与光源位置比较。如果法线指向光源，就使用暖色调，否则就使用冷色调。该算法会在两种给定的颜色中间根据角度来插值。这是一种非真实感的渲染技术，将会在第11章讨论。图3.8展示了这个效果。

![图](/images/RTR3.03.08.png)

特效文件的开头定义了特效的一些变量。最开始的是跟相机相关的一些`untweakalbes`（不可变的）变量,他们会自动关联到特效上：

``` c
	float4x4 WorldXf    : World;
	float4x4 WorldITXf  : WorldInverseTranspose;
	float4x4 WvpXf      : WorldViewProjection;
```

这个语法是 `type id : sematic`（类型 名称 ：语义）。float4x4用来标识矩阵，id名称则是用户自定义的，语义是内置的名称。正如语义名称所表达的，`WorldXf`是模型到世界空间的转换矩阵，而`WorldITXf`是它的妮转置矩阵，`WvpXf`则是从模型空间到相机裁剪空间的转换矩阵。这些被语义标识的值需要由应用程序提供，并且用户没有接口去修改。

接下来是用户自定义的变量：

``` c
	float3 Lamp0Pos : Position <
		string Object = "PointLight0";
		string UIName = "Lamp 0 Position";
		string Space  = "World";
	> = {-0.5f, 2.0f, 1.25f};

	float3 WarmColor <
		string UIName   = "Gooch Warm Tone";
		string UIWidget = "Color";
	> = {1.3f, 0.9f, 0.15f};

	float3 CoolColor <
		string UIName   = "Gooch Cool Tone";
		string UIWidget = "Color";
	> = {0.05f, 0.05f, 0.6f};
```

<>中附加了一些注释，然后给定了默认值。这些注释是应用预定义的，但对于着色编译器来说没有任何意义。应用程序可以访问这些注释，就前面这个例子而言，这些注释描述了应用程序应该如何展示这些变量。

``` c
	struct appdata {
		float3 Position : POSITION;
		float3 Normal   : NORMAL;
	};

	struct vertexOutput {
		float4 HPosition     : POSITION;
		float3 LightVec      : TEXCOORD1;
		float3 WorldNormal   : TEXCOORD2;
	};
```

`appdata`定义了模型中顶点的数据类型，也就是顶点着色器的输入数据。`vertexOutput`则是顶点着色器产生的输出数据，也是像素着色器的输入数据。TEXCOORD*的使用，是流水线发展的结果。最开始，因为表面可以关联多个纹理，这些数据字段用来存储不同的纹理坐标(`texture coordinates`)。而在实践当中，这些数据字段可以存储从顶点着色器传递给像素着色器的任意数据。

接下来是定义着色器程序代码定义部分，这里我们只定义一个顶点着色器程序：

``` c
	vertexOutput std_VS(appdata IN) {
		vertextOutput OUT;
		float4 No = float4(IN.Normal, 0);
		OUT.WorldNormal = mul(No, WorldITXf).xyz;
		float4 Po = float4(IN.Position, 1);
		float4 Pw = mul(Po, WorldXf);
		OUT.LightVec = (Lamp0Pos - Pw.xyz);
		OUT.HPosition = mul(Po, WvpXf);
		return OUT;
	}
```

这段代码首先使用矩阵乘法计算世界空间的表面法线。变换是下一章的主题，我们就不详细解释这里为什么使用逆转置矩阵了。世界空间的坐标的计算也使用了离屏变换。表面到光源的向量，使用光源的位置减去这个前面获得的这个世界空间位置来获得。最后，对象的位置变换到裁剪空间下，以便栅格器来使用。这是顶点着色器必须要完成的部分。

给定世界空间下光照方向和表面法线，像素着色器程序就能计算表面的颜色：

``` c
	float4 gooch_PS(vertexOutput IN) : COLOR
	{
		float3 Ln = normalize(IN.LightVec);
		float3 Nn = normalize(IN.WorldNormal);
		float ldn = dot(Ln, Nn);
		float mixer = 0.5 * (ldn + 1.0);
		float4 result = lerp(CoolColor, WarmColor, mixer);
		return result;
	}
```

`Ln`向量是光线方向的规范化，`Nn`是表面法线的规范化。经过规范化，这两者的点积的结果就是这两个向量夹角的`cosine`值。我们希望使用这个值，在冷色和暖色之间线性的插值。函数`lerp()`需要一个在[0,1]之间的混合值，0表示返回`CoolColor`，1则是返回`WarmColor`，中间的值则是混合这两者。因为`cosine`的取值范围是[-1,1]，`mixer`的取值范围则被转换到[0,1]。然后这个值用来混合色调得到片元的正确颜色。这些着色器就是函数化的。一个特效文件可以定义多个着色器函数，也可以使用其他特效文件定义的着色器函数。

一个`pass`会定义一个顶点和一个像素着色器（以及可能的几何着色器），以及`pass`需要的状态设置。一个`technique`是使用一个或多个`pass`的组合，来达到目标效果需求。我们这个例子中只使用单个`pass`:

``` c
	technique Gooch < string Script = "Pass=p0;"; > {
		pass p0 < string Script = "Draw=geometry;"; > {
			VertexShader = compile vs_2_0 std_VS();
			PixelShader = compile ps_2_a gooch_PS();
			ZEnable = true;
			ZWriteEnable = true;
			ZFunc = LessEqual;
			AlphaBlendEnable = false;
		}
	}
```

这些设置项设置了强制Z缓冲可读写，片元的深度小于等于已存储的Z深度则被继续使用。透明混合关闭，使用这个`technique`的对象被假定是不透明的。这些规则表明，如果当前片元的z深度与当前该点存储的深度相当，或离相机更近，则当前片元的颜色将替换原来的像素颜色（就是帧缓冲中该点的颜色）。这也就是Z缓冲的默认用法。

一个特效文件中，可以存储多个`technique`。这些`technique`一般都是实现同一种效果，不过针对的是不同的着色模型（例如SM2.0 SM3.0之类的）。可以实现非常多的效果。图3.9稍稍展示了一下现代可编程着色管线的威力。一个特效通常包含了相关的`techniques`。很多方法被开发出来，来管理着色器集合。

![图](/images/RTR3.03.09.png)

我们以及走到GPU旅程的终点。还有许多东西是GPU可以做的，各种功能的组合使用方式也用非常多。本书的中心主题也正是要讨论相关的理论和算法的协调使用。有了这些基础部分，我们的重心将转移到如何深入理解变换、视觉表现以及流水线中的关键元素。


### 深入阅读的资源
David Blythe关于DX10的论文，以及它引用的文献，是现代GPU管线以及它背后的基本原理的一个很好的概览。

可编程顶点着色器和像素着色器的相关问题很容易就写满一本书。我们得建议是：访问ATI和NVIDIA的开发网站来获取最新的开发技巧。免费的FX Composer 2和RenderMonkey都是可交互的着色器设计工具，他们很合适用来实验着色器，以及观察他们是如何运作的。Sander提供了一个在着色模型2.0下，使用HLSL来实现固定功能流水线的实现。

要学习正式的着色器编程就需要花一番功夫了。《OpenGL Shading Language》讲解了《红宝书》的遗漏，它使用GLSL来描述实现，就是OpenGL的可编程着色语言。要学习HLSL的话，DX的API一直在发展中，可以上[本书的网站](http://www.realtimerendering.com)来看相关的一些链接。O'Rorker的文章提供了一种易读的特效介绍，以及有效的管理着色器方法。Cg语言提供一个抽象层，可以导出到主流的各种API和平台，也提供了主流模型动画包的插件。Sh 元编程语言更加的抽象，基本上是作为一个C++库来工作，映射图形代码给GPU。

更高级的着色器技巧，请看《GPU Gems》及《ShaderX》系列书籍。《游戏编程精粹》系列书籍也包含一些相关内容。DX SDK中包含了很多重要的着色器和算法实例。


