# The Book of Shaders

中文翻译版真的不靠谱，还是看原文同时查找相关资料理解比较准确和直接


## Shaping functions

### 基础

```glsl
y = mod(x,0.5); // 返回 x 对 0.5 取模的值
y = fract(x); // 仅仅返回数的小数部分
y = ceil(x);  // 向正无穷取整
y = floor(x); // 向负无穷取整
y = sign(x);  // 提取 x 的正负号
y = abs(x);   // 返回 x 的绝对值
y = clamp(x,0.0,1.0); // 把 x 的值限制在 0.0 到 1.0
y = min(0.0,x);   // 返回 x 和 0.0 中的较小值
y = max(0.0,x);   // 返回 x 和 0.0 中的较大值  
y = step(0.5,x);// Step will return 0.0 unless the value is over 0.5,in that case it will return 1.0
y = smoothstep(0.1,0.9,x);//Smooth interpolation between 0.1 and 0.9
```

![](assets/kynd.png)

[shadershop](http://tobyschachman.com/Shadershop/)函数生成网页工具

[useful little functions](https://www.iquilezles.org/www/articles/functions/functions.htm) by [Iñigo Quiles](http://www.iquilezles.org/)

### Complex shaping functions

[Golan Levin](http://www.flong.com/) as great documentation of more complex shaping functions that are extraordinarily helpful.

+ Polynomial Shaping Functions: [www.flong.com/texts/code/shapers_poly](http://www.flong.com/texts/code/shapers_poly/)
+ Exponential Shaping Functions: [www.flong.com/texts/code/shapers_exp](http://www.flong.com/texts/code/shapers_exp/)
+ Circular & Elliptical Shaping Functions: [www.flong.com/texts/code/shapers_circ](http://www.flong.com/texts/code/shapers_circ/)
+ Bezier and Other Parametric Shaping Functions: [www.flong.com/texts/code/shapers_bez](http://www.flong.com/texts/code/shapers_bez/)



- [ ] **以上内容请实现到自己的库中**

## HSB to RGB shader code

[代码来源](https://www.shadertoy.com/view/MsS3Wc "shadertoy")

```glsl
hue = pow(hue, 1.5);//RGB=>RYB色轮（缩小蓝紫范围，扩大黄绿范围）
vec3 rgb = clamp(abs(mod(hue * 6.0 + vec3(0.0,4.0,2.0), 6.0) - 3.0) - 1.0, 0.0, 1.0 );
rgb = rgb * rgb * (3.0 - 2.0 * rgb);//smooth
return brightness * mix (vec3(1.0), rgb, saturation);
```

<center>
    <img src="assets/001/rgb.png" width="200"/> 
    <img src="assets/001/rgb.png" width="200"/>
</center>
## <u>*Interaction of Color*</u> 的例子

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float rect(in vec2 st, in vec2 size){
	size = 0.25-size*0.25;
    vec2 uv = smoothstep(size,size+size*vec2(0.002),st*(1.0-st));
	return uv.x*uv.y;
}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy;

    vec3 influenced_color = vec3(0.745,0.678,0.539);
    
    vec3 influencing_color_A = vec3(0.653,0.918,0.985); 
    vec3 influencing_color_B = vec3(0.980,0.576,0.113);
    
    vec3 color = mix(influencing_color_A,
                     influencing_color_B,
                     step(.5,st.x));
    
    color = mix(color,
               influenced_color,
               rect(abs((st-vec2(.5,.0))*vec2(2.,1.)),vec2(.05,.125)));

    gl_FragColor = vec4(color,1.0);
}
```

rect函数理解：step 函数作为 分支（if） 功能使用， smoothstep类似，过渡是平滑的。

x(1-x)构造了一个二次函数图像用来确定x轴两个边界（0边界外和1边界内）

y轴同理，再用边界信息相乘来获得 逻辑运算 “与”（同时满足两个条件）的效果。

这么做size和center都不太直观。但是构造起来很简洁。

### 语法：in out inout

```c++
int newFunction(in vec4 aVec4,      // read-only
                out vec3 aVec3,     // write-only
                inout int aInt);    // read-write
```



## Shapes 形状

画一个方形的例子

```glsl
vec2 bl = step(vec2(0.1),st);//用step函数进行 bottom left 边界判断（0.1处）
vec2 tr = step(vec2(0.1),1 - st);//top right
vec3 color = vec3(bl.x * bl.y * tr.x * tr.y);//4个函数同时为1时即为方形界内
```

用floor()来实现

```glsl
vec2 bl = floor(st + vec2(0.9)) ;
vec2 tr = 1 - floor(st + vec2(0.1));
vec3 color = vec3(bl.x * bl.y * tr.x * tr.y);
```

变成函数

```glsl
float rect_stepVersion(vec2 size, vec2 center, in vec2 st)
{
	vec2 blp = center - size/2.0;
	vec2 trp = vec2(1.0) - center - size/2.0;
	vec2 bl = step(vec2(blp.x,blp.y),st);
	vec2 tr = step(vec2(trp.x,trp.y),1 - st);
	return bl.x * bl.y * tr.x * tr.y;
}

float rect_floorVersion(vec2 size, vec2 center, in vec2 st)
{
	vec2 blp = center - size/2.0;
	vec2 trp = center + size/2.0;
	vec2 bl = floor(st + vec2(1-blp.x,1-blp.y)) ;
	vec2 tr = 1 - floor(st + vec2(1-trp.x,1-trp.y));
	return bl.x * bl.y * tr.x * tr.y;
}//还有问题
```

