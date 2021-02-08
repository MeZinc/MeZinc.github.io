[TOC]



# ShaderLab Syntax

## 未解决的问题

gamma,sRGB

[PerRendererData]

[GLSL to HLSL](https://docs.microsoft.com/en-us/previous-versions/windows/apps/dn166865(v=win.10)?redirectedfrom=MSDN)

[Platform-specific rendering differences](https://docs.unity3d.com/Manual/SL-PlatformDifferences.html)

光栅化理解

multiple render target (MRT) **rendering**

Pass tags 使用

延迟渲染下的stencil test

## 基本格式

```glsl
Shader "name"
{
    Properties
    {
    }
    SubShader
    {
    }
    SubShader
    {
    }
    [Fallback]
    [CustomEditor]
}
```

## Properties

```glsl
Properties
    {
        name("display name", Range(min,max)) = number
        name ("display name", Float) = number
		name ("display name", Int) = number
        
        name ("display name", Color) = (number,number,number,number)
		name ("display name", Vector) = (number,number,number,number)
        
        name ("display name", 2D) = "defaulttexture" {}
		name ("display name", Cube) = "defaulttexture" {}
		name ("display name", 3D) = "defaulttexture" {}
        //2D贴图,默认值要么是空字符串，要么是内建贴图的一个: “white” (RGBA: 1,1,1,1), “black” (RGBA: 0,0,0,0), “gray” (RGBA: 0.5,0.5,0.5,0.5), “bump” (RGBA: 0.5,0.5,1,0.5) or “red” (RGBA: 1,0,0,0).
        //非2D贴图，默认值是一个空字符串。如果这项没有赋值，shader会使用灰色作为默认色
        //property values can be accessed using property name in square brackets: [name].例如： 透明混合 Blend [_SrcBlend] [_DstBlend]
        
        //可以通过前缀控制参数的UI表现方式。以下是Unity自带的attribute。
        [HideInInspector]//面板里隐藏的属性
        [NoScaleOffset]//平移拉伸属性不显示
        [Normal]//要求使用法线贴图
        [HDR]//HDR贴图
        [Gamma]// indicates that a float/vector property is specified as sRGB value in the UI(just like colors are), and possibly needs conversion according to color space used. See Properties in Shader Programs.
        [PerRendererData]//贴图属性来自于 MaterialPropertyBlock,这个贴图位置的UI会不同
        [MainTexture]//变量名为_MainTex效果相同，代码可以直接控制。
        [MainColor]//_Color,同上
        //还可以通过 MaterialPropertyDrawer 来自己写attribute
        
    }//shader的变量如果不在Properties区域里，则只能通过代码进行设置,例如:material.SetFloat等等
```

## SubShader

```glsl
//基本格式
Subshader
{
    [Tags]
    [CommonState]
    [PassDef...]
}
```

每个pass渲染一次（或多次）,也可以根据Tags的设定来选择特定情况下渲染，例如“ForwardAdd“，仅仅在Forward渲染方式下渲染，并且根据光照数量计算多次。

### Pass

```glsl
Pass{
	[Name and Tags]
	[RenderSetup]
}

//Render state set-up
//cull
Cull Back | Front | Off
//ZTest
ZTest (Less | Greater | LEqual | GEqual | NotEqual |Always)
//ZWrite
ZWrite On | Off
//Offset
Offset OffsetFactor, OffsetUnits
//Blend
Blend sourceBlendMode destBlendMode
Blend sourceBlendMode destBlendMode, alphaSourceBlendMode alphaDestBlendMode
BlendOp colorOp
BlendOp colorOp, alphaOp
AlphaToMask On | Off
//Color Mask
ColorMask RGB | A | 0 | any combination of R, G, B, A
```



#### Legacy fixed-function Shader commands

A number of commands are used for writing legacy “fixed-function style” **Shaders**
. This is considered deprecated functionality, as writing [Surface Shaders](https://docs.unity3d.com/Manual/SL-SurfaceShaders.html)
 or [Shader programs](https://docs.unity3d.com/Manual/SL-ShaderPrograms.html) allows much more flexibility. However, for very simple Shaders, writing them in fixed-function style can sometimes be easier, so the commands are provided here. Note that all of the following commands are are ignored if you are not using fixed-function Shaders.

##### Fixed-function Lighting and Material

```
Lighting On | Off
Material { Material Block }
SeparateSpecular On | Off
Color Color-value
ColorMaterial AmbientAndDiffuse | Emission
```

All of these control fixed-function per-vertex Lighting: they turn it on, set up Material colors, turn on specular highlights, provide default color (if vertex Lighting is off), and controls how the **mesh**
 vertex colors affect Lighting. See documentation on [Materials](https://docs.unity3d.com/Manual/SL-Material.html)
 for more details.

##### Fixed-function Fog

```
Fog { Fog Block }
```

Set fixed-function Fog parameters. See documentation on[Fogging](https://docs.unity3d.com/Manual/SL-Fog.html) for more details.

##### Fixed-function AlphaTest

```
AlphaTest (Less | Greater | LEqual | GEqual | Equal | NotEqual | Always) CutoffValue
```

Turns on fixed-function alpha testing. See documentation on [alpha testing](https://docs.unity3d.com/Manual/SL-AlphaTest.html) for more details.

##### Fixed-function Texture combiners

After the render state setup, use [SetTexture](https://docs.unity3d.com/Manual/SL-SetTexture.html) commands to specify a number of Textures and their combining modes:

```
SetTexture textureProperty { combine options }
```





### 剔除和深度测试

#### Cull

```glsl
Cull back | Front | Off
```



- **Back** 不渲染里面的面
- **Front** 不渲染外面的面
- **Off** 关闭剔除

#### ZWrite

```glsl
ZWrite On | Off
```

控制物体的像素是否写入深度数据。默认开启，透明物体需要关掉。

#### ZTest

```glsl
ZTest Less | Greater | LEqual | GEqual | Equal | NotEqual | Always
```

深度测试如何进行： 默认LEqual（小于等于）

#### Offset

```glsl
Offset Factor, Units
```

 *Factor* scales the maximum Z slope, with respect to X or Y of the polygon, and *units* scale the minimum resolvable depth buffer value. This allows you to force one polygon to be drawn on top of another although they are actually in the same position.For example `Offset 0, -1` pulls the polygon closer to the **camera**
 ignoring the polygon’s slope, whereas `Offset -1, -1` will pull the polygon even closer when looking at a grazing angle.

处理z-fighting,处于同一个z平面的渲染顺序每一个Fragment的深度值都会增加如下所示的偏移量：$offset = (m * factor) + (r * units)$

m是多边形的深度的斜率（在光栅化阶段计算得出）中的最大值。这句话难以理解，你只需知道，一个多边形越是与近裁剪面（near clipping plan）平行，m就越接近0。

r是能产生在窗口坐标系的深度值中可分辨的差异的最小值，r是由具体实现OpenGL的平台指定的一个常量。

一个大于0的offset 会把模型推到离你（摄像机）更远一点的位置，相应地，一个小于0的offset 会把模型拉近。

#### 深度写入的透明shader

半透明着色器通常不写入深度数据。这样写入第一个pass写入深度但是不写入颜色，第二个pass正常渲染（关闭zwrite），此时深度信息已经正确。缺点是消耗性能。

```glsl
Shader "Transparent/Diffuse ZWrite" {
Properties {
    _Color ("Main Color", Color) = (1,1,1,1)
    _MainTex ("Base (RGB) Trans (A)", 2D) = "white" {}
}
SubShader {
    Tags {"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"}
    LOD 200

    // extra pass that renders to depth buffer only
    Pass {
        ZWrite On
        ColorMask 0
    }

    // paste in forward rendering passes from Transparent/Diffuse
    UsePass "Transparent/Diffuse/FORWARD"
}
Fallback "Transparent/VertexLit"
}
```

#### Debugging Normals

背面变粉色（调试用）

```glsl
Shader "Reveal Backfaces" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" { }
    }
    SubShader {
        // Render the front-facing parts of the object.
        // We use a simple white material, and apply the main texture.
        Pass {
            Material {
                Diffuse (1,1,1,1)
            }
            Lighting On
            SetTexture [_MainTex] {
                Combine Primary * Texture
            }
        }

        // Now we render the back-facing triangles in the most
        // irritating color in the world: BRIGHT PINK!
        Pass {
            Color (1,0,1,1)
            Cull Front
        }
    }
}
```

#### Glass Culling

```glsl
Shader "Simple Glass" {
    Properties {
        _Color ("Main Color", Color) = (1,1,1,0)
        _SpecColor ("Spec Color", Color) = (1,1,1,1)
        _Emission ("Emmisive Color", Color) = (0,0,0,0)
        _Shininess ("Shininess", Range (0.01, 1)) = 0.7
        _MainTex ("Base (RGB)", 2D) = "white" { }
    }

    SubShader {
        // We use the material in many passes by defining them in the subshader.
        // Anything defined here becomes default values for all contained passes.
        Material {
            Diffuse [_Color]
            Ambient [_Color]
            Shininess [_Shininess]
            Specular [_SpecColor]
            Emission [_Emission]
        }
        Lighting On
        SeparateSpecular On

        // Set up alpha blending
        Blend SrcAlpha OneMinusSrcAlpha

        // Render the back facing parts of the object.
        // If the object is convex, these will always be further away
        // than the front-faces.
        Pass {
            Cull Front
            SetTexture [_MainTex] {
                Combine Primary * Texture
            }
        }
        // Render the parts of the object facing us.
        // If the object is convex, these will be closer than the
        // back-faces.
        Pass {
            Cull Back
            SetTexture [_MainTex] {
                Combine Primary * Texture
            }
        }
    }
}
```



### Blending

![img](assets\004\PipelineBlend.png)

#### syntax

`Blend off`:默认，关闭blend

`Blend SrcFactor DstFactor`:生成的color乘以Src再加上 当前屏幕颜色乘以Dst.

`Blend SrcFactor DstFactor,SrcFactorA DstFactorA`:alpha通道使用后面两个参数混合。

`BlendOp Op`:Op 即 operation指令

`BlendOp OpColor, OpAlpha`: color (RGB) 和alpha (A)通道使用不同指令 .

multiple render target (MRT) **rendering**,N为目标index（0~7）：

`Blend N SrcFactor DstFactor`

`Blend N SrcFactor DstFactor, SrcFactorA DstFactorA`

`BlendOp N Op`

`BlendOp N OpColor,OpAlpha`

`AlphaToMask On`: Turns on alpha-to-coverage. When MSAA is used, alpha-to-coverage modifies multisample coverage mask proportionally to the pixel Shader result alpha value. This is typically used for less aliased outlines than regular alpha test; useful for vegetation and other alpha-tested Shaders.

**参数**（factor）和**指令**(op)下方详细列出：

#### Blend factors

下列属性可以用于**Blend** command中both SrcFactor & DstFactor. **Source** 计算得到的颜色, **Destination**:屏幕上已经有的颜色. The blend factors are ignored if **BlendOp** is using logical operations.

|                      |                                                              |
| :------------------- | :----------------------------------------------------------- |
| **One**              | The value of one - use this to let either the source or the destination color come through fully. |
| **Zero**             | The value zero - use this to remove either the source or the destination values. |
| **SrcColor**         | The value of this stage is multiplied by the source color value. |
| **SrcAlpha**         | The value of this stage is multiplied by the source alpha value. |
| **DstColor**         | The value of this stage is multiplied by frame buffer source color value. |
| **DstAlpha**         | The value of this stage is multiplied by frame buffer source alpha value. |
| **OneMinusSrcColor** | The value of this stage is multiplied by (1 - source color). |
| **OneMinusSrcAlpha** | The value of this stage is multiplied by (1 - source alpha). |
| **OneMinusDstColor** | The value of this stage is multiplied by (1 - destination color). |
| **OneMinusDstAlpha** | The value of this stage is multiplied by (1 - destination alpha). |

常用混合方式：

```glsl
Blend SrcAlpha OneMinusSrcAlpha //传统透明度计算
Blend One OneMinusSrcAlpha // Premultiplied transparency计算式先给Src颜色乘以Alpha
```

#### Blend operations

The following blend operations can be used:

|                         |                                                           |
| :---------------------- | :-------------------------------------------------------- |
| **Add**                 | Add source and destination together.                      |
| **Sub**                 | Subtract destination from source.                         |
| **RevSub**              | Subtract source from destination.                         |
| **Min**                 | Use the smaller of source and destination.                |
| **Max**                 | Use the larger of source and destination.                 |
| **LogicalClear**        | Logical operation: Clear (0) **DX11.1 only**.             |
| **LogicalSet**          | Logical operation: Set (1) **DX11.1 only**.               |
| **LogicalCopy**         | Logical operation: Copy (s) **DX11.1 only**.              |
| **LogicalCopyInverted** | Logical operation: Copy inverted (!s) **DX11.1 only**.    |
| **LogicalNoop**         | Logical operation: Noop (d) **DX11.1 only**.              |
| **LogicalInvert**       | Logical operation: Invert (!d) **DX11.1 only**.           |
| **LogicalAnd**          | Logical operation: And (s & d) **DX11.1 only**.           |
| **LogicalNand**         | Logical operation: Nand !(s & d) **DX11.1 only**.         |
| **LogicalOr**           | Logical operation: Or (s \| d) **DX11.1 only**.           |
| **LogicalNor**          | Logical operation: Nor !(s \| d) **DX11.1 only**.         |
| **LogicalXor**          | Logical operation: Xor (s ^ d) **DX11.1 only**.           |
| **LogicalEquiv**        | Logical operation: Equivalence !(s ^ d) **DX11.1 only**.  |
| **LogicalAndReverse**   | Logical operation: Reverse And (s & !d) **DX11.1 only**.  |
| **LogicalAndInverted**  | Logical operation: Inverted And (!s & d) **DX11.1 only**. |
| **LogicalOrReverse**    | Logical operation: Reverse Or (s \| !d) **DX11.1 only**.  |
| **LogicalOrInverted**   | Logical operation: Inverted Or (!s \| d) **DX11.1 only**. |

#### Alpha blending, alpha testing, alpha-to-coverage

##### Alpha blending

半透明渲染普遍使用的混合方式，因为被认定为半透明物体，不能使用Deferred shading，不能接收阴影。凹陷物体或者自相交物体的绘制顺序往往有问题（一般render queue为transparent，关闭深度写入）。

##### Alpha testing/Cutout

使用`clip()`函数丢弃部分像素。由于视为不透明物体，没有渲染顺序问题，但是像素要么完全存在要么完全丢弃，会导致锯齿严重。（render queue 为TransparentCutout）

##### Alpha-to-coverage

使用MSAA时，可以使用这项功能，平滑边缘（效果跟MSAA级别设置相关）。对于没有（或极少）半透明区域的贴图最有效果。（render queue 为TransparentCutout）`AlphaToMask On`

#### Pass Tags

 `Tags { "TagName1" = "Value1" "TagName2" = "Value2" }`

位置在Pass内，而不是subshader的Tag。

##### LightMode Tag

定义Pass在光照管线中的职责。和光照相关的shader通常写成surface shader.

`"LightMode" = "value"`

- **Always**: 总是渲染; no lighting is applied.
- **ForwardBase**: Used in [Forward rendering](https://docs.unity3d.com/Manual/RenderTech-ForwardRendering.html)
  , ambient, main directional light, vertex/SH lights and **lightmaps**
   are applied.
- **ForwardAdd**: Used in [Forward rendering](https://docs.unity3d.com/Manual/RenderTech-ForwardRendering.html); additive per-pixel lights are applied, one pass per light.
- **Deferred**: Used in [Deferred Shading](https://docs.unity3d.com/Manual/RenderTech-DeferredShading.html)
  ; renders g-buffer.
- **ShadowCaster**: Renders object depth into the shadowmap or a depth texture.
- **MotionVectors**: Used to calculate per-object motion vectors.
- **PrepassBase**: Used in [legacy Deferred Lighting](https://docs.unity3d.com/Manual/RenderTech-DeferredLighting.html), renders normals and specular exponent.
- **PrepassFinal**: Used in [legacy Deferred Lighting](https://docs.unity3d.com/Manual/RenderTech-DeferredLighting.html), renders final color by combining textures, lighting and emission.
- **Vertex**: Used in [legacy Vertex Lit rendering](https://docs.unity3d.com/Manual/RenderTech-VertexLit.html) when object is not lightmapped; all vertex lights are applied.
- **VertexLMRGBM**: Used in [legacy Vertex Lit rendering](https://docs.unity3d.com/Manual/RenderTech-VertexLit.html) when object is lightmapped; on platforms where lightmap is RGBM encoded (PC & console).
- **VertexLM**: Used in [legacy Vertex Lit rendering](https://docs.unity3d.com/Manual/RenderTech-VertexLit.html) when object is lightmapped; on platforms where lightmap is double-LDR encoded (mobile platforms).

##### PassFlags tag

A pass can indicate flags that change how [rendering pipeline](https://docs.unity3d.com/Manual/SL-RenderPipeline.html) passes data to it. This is done by using **PassFlags** tag, with a value that is space-separated flag names. Currently the flags supported are:

- **OnlyDirectional**: When used in ForwardBase pass type, this flag makes it so that only the main directional light and ambient/lightprobe data is passed into the shader. This means that data of non-important lights is *not* passed into vertex-light or spherical harmonics shader variables. See [Forward rendering](https://docs.unity3d.com/Manual/RenderTech-ForwardRendering.html) for details.

##### RequireOptions tag

A pass can indicate that it should only be rendered when some external conditions are met. This is done by using **RequireOptions** tag, whose value is a string of space separated options. Currently the options supported by Unity are:

- **SoftVegetation**: Render this pass only if Soft Vegetation is on in the [Quality](https://docs.unity3d.com/Manual/class-QualitySettings.html) window.



#### Stencil（模板测试）

如果需要模板测试，在Pass开头写Stencil结构体，如果每个Pass都要用，则可以提到外面。

例子：

```glsl
Stencil{
    Ref 2
    Comp equal
    Pass keep
    Fail decrWrap
    ZFail keep
}
```

**Ref**:`Ref refernceValue`refernceValue取值0-255的整数，基准值。

**ReadMask**:`ReadMask readMask`读取遮罩，这个值将会和**Ref**值和**StencilBuffer**值进行按位与(**&**)操作，默认255（二进制11111111），即：读取原始值。

**WriteMask**:`WriteMask writeMask`写入模板缓冲时的遮罩，同上按位与（**&**）操作，默认也是255，写入原始值

**Comp**:比较操作符，默认：always。

```glsl
//Properties
_StencilComp("Stencil Comparison", Float) = 8

//Pass
Comp [_StencilComp]
```

 数值对应表

- 0 - Disabled
- 1 - Never
- 2 - Less
- 3 - Equal
- 4 - LEqual
- 5 - Greater
- 6 - NotEqual
- 7 - GEqual
- 8 - Always

|              |			                                                   |
| ------------ | ------------------------------------------------------------ |
| **Greater**  | Only render pixels whose reference value is greater than the value in the buffer. |
| **GEqual**   | Only render pixels whose reference value is greater than or equal to the value in the buffer. |
| **Less**     | Only render pixels whose reference value is less than the value in the buffer. |
| **LEqual**   | Only render pixels whose reference value is less than or equal to the value in the buffer. |
| **Equal**    | Only render pixels whose reference value equals the value in the buffer. |
| **NotEqual** | Only render pixels whose reference value differs from the value in the buffer. |
| **Always**   | Make the stencil test always pass.                           |
| **Never**    | Make the stencil test always fail.                           |

**Pass**:`Pass stencilOperation`模板测试(和深度测试)通过时的模板缓冲值操作。默认keep.

**Fail**:`Fail stencilOperation`模板测试(和深度测试)失败时的操作，默认keep.

**ZFail**:`ZFail stencilOperation`通过模板测试但没通过深度测试时的操作，默认keep.

- 0 - Keep
- 1 - Zero
- 2 - Replace
- 3 - IncrSat
- 4 - DecrSat
- 5 - Invert
- 6 - IncrWrap
- 7 - DecrWrap

|              |                                                              |
| :----------- | :----------------------------------------------------------- |
| **Keep**     | Keep the current contents of the buffer.                     |
| **Zero**     | Write 0 into the buffer.                                     |
| **Replace**  | Write the reference value into the buffer.                   |
| **IncrSat**  | Increment the current value in the buffer. If the value is 255 already, it stays at 255. |
| **DecrSat**  | Decrement the current value in the buffer. If the value is 0 already, it stays at 0. |
| **Invert**   | Negate all the bits.                                         |
| **IncrWrap** | Increment the current value in the buffer. If the value is 255 already, it becomes 0. |
| **DecrWrap** | Decrement the current value in the buffer. If the value is 0 already, it becomes 255. |

**Deferred rendering Path**下objects的stencil使用：

Stencil functionality for objects rendered in the [deferred rendering path](https://docs.unity3d.com/Manual/RenderTech-DeferredShading.html) is somewhat limited, as during the G-buffer pass and lighting pass, Unity uses the stencil buffer for other purposes. During those two stages, the stencil state defined in the shader will be ignored. Because of that, you cannot mask out these objects based on a stencil test, but they can still modify the buffer contents, to be used by objects rendered later in the frame. Objects rendered in the forward **rendering**
 path following the deferred path (e.g. transparent objects or objects without a surface shader) will set their stencil state normally again.

These bits are used for the stencil buffer in the deferred **rendering path**
:

- Bit #7 (value=128) indicates any non-background object.
- Bit #6 (value=64) indicates non-lightmapped objects.
- Bit #5 (value=32) is not used by Unity.
- Bit #4 (value=16) is used for light shape culling during the lighting pass, so that the lighting shader is only executed on pixels that the light touches, and not on pixels where the surface geometry is actually behind the light volume.
- Lowest four bits (values 1,2,4,8) are used for light layer **culling masks**.

It is possible to operate within the range of the unused bits using the stencil read and write masks, or you can force the **camera** to clean the stencil buffer after the lighting pass using [Camera.clearStencilAfterLightingPass](https://docs.unity3d.com/ScriptReference/Camera-clearStencilAfterLightingPass.html).

原地址：https://docs.unity3d.com/Manual/SL-Stencil.html

#### Name

`Name "PassName"`:为当前pass命名，会被转换为大写，使这个Pass能被UsePass调用。例：通过

`UsePass "VertexLit/SHADOWCASTER"`来使用VertexLit的SHADOWCASTER这个pass来渲染阴影。

#### Legacy部分暂不加入

### UsePass

上部分有

### GrabPass

抓取物体被绘制的范围中的屏幕图像绘制成贴图，随后可以被之后的pass使用。

- `Grab Pass {}`,之后可以使用`_GrabTexture`使用抓取到的贴图。Note: this form of grab pass will do the time-consuming screen grabbing operation for each object that uses it.
- `Grab Pass {"TextureName"}`

