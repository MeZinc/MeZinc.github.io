# The Book of Shaders

中文翻译版有的不太靠谱，用作参考的同时还是看原文同时查找相关资料理解比较准确和直接


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
y = step(0.5,x);//  x<0.5 return 0.0 ,x>=0.5 return 1.0
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
    <img src="assets/001/RGB.png" width="200"/>
    <img src="assets/001/RYB.png" width="200"/>
</center>
## <u>*Interaction of Color*</u> 的例子

**颜色混合函数**`mix(color1,color2,pct)`

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

### 方形

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
}
//原文中提供的画方形函数
float box(vec2 _st, vec2 _size, float _smoothEdges){
    _size = vec2(0.5)-_size*0.5;
    vec2 aa = vec2(_smoothEdges*0.5);
    vec2 uv = smoothstep(_size,_size+aa,_st);
    uv *= smoothstep(_size,_size+aa,vec2(1.0)-_st);
    return uv.x*uv.y;
}
```

### 圆形

#### 基础

```glsl
distance(vec2,vec2);//两点
length(vec2);//向量长度

//点乘（dot）获得距离场
vec2 dist = st - vec2(0.5);
dot(dist,dist);//获得的是距离的平方，节省开方操作
```

#### 圆角方形

```glsl
st = st * 2. - 1. ;//映射到(-1, 1）
d = length(max(abs(st)-.3),0.);//圆角方形距离场，之后再用step或者smoothstep获得目标图形
```

#### 更多用途

```glsl
//d 为 获得的距离场
d = step(.3, d);
d = step(.3, d)* step(d, .4);//获取距离0.3到0.4的部分（环）
d = smoothstep(.3, .4, d) * smoothstep(.6, .5, d);//平滑的环。
```

![smoothstepReverse](assets/001/smoothstepReverse.png)



### 极坐标 Polar shapes



#### 基本

```glsl
vec2 pos = vec2(0.5) - st;
float r = length(pos) * 2.0;//半径 * 2（之前的色环为了把(0,0.5)扩展到(0,1)来放所有颜色
float a = atan(pos.y, pos.x);//角度(-PI,PI)
```

#### 应用

```glsl
float f = cos(a*3.);
// f = abs(cos(a*3.));
// f = abs(cos(a*2.5))*.5+.3;//花
// f = abs(cos(a*12.)*sin(a*3.))*.8+.1;//雪花
// f = smoothstep(-.5,1., cos(a*10.))*0.2+0.5;//齿轮

color = vec3( 1.-smoothstep(f,f+0.02,r) );
```

多边形

```glsl
vec3 color = vec3(0.0);
float d = 0.0;

// Remap the space to -1. to 1.
st = st *2.-1.;

// Number of sides of your shape
int N = 3;

// Angle and radius from the current pixel
float a = atan(st.x,st.y)+PI;//0 -> PI
float r = TWO_PI/float(N);

// Shaping function that modulate the distance
d = cos(floor(.5+a/r)*r-a)*length(st);//根据极坐标角度划分为N个区域，每个区域值从cos(π/n)到cos(π/2)再到cos(π/n)。这个值乘以极坐标的r,即可得到对应多边形的大小（顶点到边的长度，注意st映射到了(-1,1)）
//d = cos((.5-fract(a/r+.5))*r)*length(st);//同理

color = vec3(1.0-smoothstep(.4,.41,d));//由于精确度有限，若使用step,会出现线不够平滑的情况
// color = vec3(d);

gl_FragColor = vec4(color,1.0);
```

![使用step的效果](assets/001/usingstep.png)

![实现](assets/003/pixelspiritsdeck.png)

## 二位矩阵 2D Matrices

书中之前的例子里都在归一化坐标后使用了`st.y *= u_resolution.y/u_resolution.x;`，这样可以使绘制的图形比例不随分辨率比例变化而变化。

**注意**

glsl的矩阵$M$对向量$v$的乘法,向量$v$被当做列向量，例如`mat2 * vec2`,写做$Mv$,实际计算的是$M^Tv$,
$$
M^Tv =
\left[\matrix{
m_{1,1} & m_{1,2} \\
m_{2,1} & m_{2,2}
}\right]^T
\left[\matrix{v_1	\\v_2}\right]
$$
因此需要将推导得到的矩阵转置后再在代码中使用，例如：推导出平移矩阵$M=\left[\matrix{m_{1,1} & m_{1,2} \\m_{2,1} & m_{2,2}}\right]$,若直接使用则会出错，代码里必须使用$M^T$，假设得到的是$M'$。因此公式里写做$M'^T$，（这样公式是正确的），代码里直接使用$M'$，这种方式直观的出现了代码里要用的矩阵，同时保证公式正确（就是在写文档时要进行转置操作）。

### 平移 Translate

$$
\begin{bmatrix}
1 	& 0 	& 0 \\
0 	& 1 	& 0 \\
t_x & t_y 	& 1
\end{bmatrix}^T
\cdot
\left[\matrix{ x \\ y \\ 1}\right]
=
\left[\matrix{ x + t_x \\ y + t_y \\ 1}\right]
\tag 1
$$

```glsl
mat3 translate(vec2 _t)
{
	return mat3(
		1	,0		,0	,
		0	,1		,0	,
		_t.x,_t.y	,1
	);
}
```



### 旋转 Rotations

$$
\begin{bmatrix}
\cos\theta & \sin\theta & 0 \\
-\sin\theta & \cos\theta & 0 \\
0 & 0 & 1
\end{bmatrix}^T
\cdot
\left[\matrix{ x \\ y \\ 1}\right]
=
\left[\matrix{
x \cdot \cos\theta - y \cdot \sin\theta \\
x \cdot \sin\theta + y \cdot \cos \theta\\
1
}\right]
\tag 2
$$

```glsl
mat3 rotate2d(float _angle)
{
	return mat3(
		cos(_angle)	,sin(_angle)	,0,
		-sin(_angle)	,cos(_angle)	,0,
		0			,0				,1
	);
}
```

注意这里的$\theta$，根据矩阵的推导，是逆时针的旋转角度。

### 缩放 Scale

$$
\begin{bmatrix}
S_x & 0 & 0 \\
0 & S_y & 0 \\
0 & 0 & 1
\end{bmatrix}^T
\cdot
\left[\matrix{ x \\ y \\ 1}\right]
=
\left[\matrix{
S_x \cdot x \\
S_y \cdot y\\
1
}\right]
\tag 2
$$

```glsl
mat3 scale(vec2 _t)
{
	return mat3(
		_t.x,0		,0	,
		0	,_t.y	,0	,
		0	,0		,1
	);
}
```

原文中对st应用变换矩阵，相当于操作坐标系而不改变画布，如平移`vec2(0.5)`,坐标系向右上平移0.5个单位，书中给出的图形（cross）就会更加接近原点，放大坐标系，图形就会相对缩小，顺时针旋转坐标系，图形相应逆时针旋转。

[fake UI or HUD (heads up display)](https://www.pinterest.com/patriciogonzv/huds/)

[例子](https://www.shadertoy.com/view/4s2SRt)

**YUV**和**RGB**色彩空间通过矩阵相互转换。

[https://en.wikipedia.org/wiki/YUV](https://en.wikipedia.org/wiki/YUV)



[一个参考例子](https://thebookofshaders.com/edit.php?log=160909065147)



## Patterns

### Tile

平铺

```glsl
vec2 tile(vec2 _st, float _zoom)
{
    _st *= _zoom;
    return fract(_st);
}
```

 [Scottish Tartan Patterns](https://www.google.com/search?q=scottish+patterns+fabric&tbm=isch&tbo=u&source=univ&sa=X&ei=Y1aFVfmfD9P-yQTLuYCIDA&ved=0CB4QsAQ&biw=1399&bih=799#tbm=isch&q=Scottish+Tartans+Patterns)



练习：

2.   

![img](https://thebookofshaders.com/09/diamondtiles-long.png)

```glsl
// Author MeZinc - 2020
// Title: Diamond Tiles
#ifdef GL_ES
precision mediump float;
#endif

#define PI 3.14159265359
#define SQRT2 1.41421

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float d1(in float angle,in float r)
{
	return r * cos(angle);
}

float d2(in float angle,in float r,in float stAn)
{
	return r* cos(stAn) / cos(PI/4.0 - stAn)* cos(angle);
}

float theta(in float x,in float top,in float r)//生成八边形
{
	float f2 =PI / 2. * abs(fract(x * 2. / PI + .5) - .5);
	f2 = clamp(f2,0.0, top);
	float pct = step(top ,f2);
	float f3 =  PI / 2. * abs((fract(x * 2. / PI)-0.5));
	return  pct * d2(f3,r,top) + (1. - pct) * d1(f2,r);
}

void main() {
	vec2 st = gl_FragCoord.xy / u_resolution;
    st.y *= u_resolution.y/u_resolution.x;
	
	st = fract(10. * st);
	
	vec2 pos = st - 0.5;
	
	float a = atan(pos.y , pos.x);
	float r = length(pos) * 2.;
	
	float set_angle = PI / 6.0;
	float sideLength = 1.;
	
	vec3 color = vec3(theta(a,set_angle,r));
	color  =1. - vec3(
        smoothstep(sideLength -0.051 , sideLength - 0.05,theta(a,set_angle,r)) *
        smoothstep(sideLength + 0.001, sideLength ,theta(a,set_angle,r))
    );
    
	gl_FragColor = vec4(color,1.0);
}
```
原文解：

```glsl
// Author @patriciogv ( patriciogonzalezvivo.com ) - 2015
// Title: Diamond Tiles

#ifdef GL_ES
precision mediump float;
#endif

#define PI 3.14159265358979323846

uniform vec2 u_resolution;
uniform float u_time;

vec2 rotate2D(vec2 _st, float _angle){
    _st -= 0.5;
    _st =  mat2(cos(_angle),-sin(_angle),
      sin(_angle),cos(_angle)) * _st;
    _st += 0.5;
    return _st;
}

vec2 tile(vec2 _st, float _zoom){
    _st *= _zoom;
    return fract(_st);
}

float box(vec2 _st, vec2 _size, float _smoothEdges){
    _size = vec2(0.5)-_size*0.5;
    vec2 aa = vec2(_smoothEdges*0.5);
    vec2 uv = smoothstep(_size,_size+aa,_st);
    uv *= smoothstep(_size,_size+aa,vec2(1.0)-_st);
    return uv.x*uv.y;
}

vec2 offset(vec2 _st, vec2 _offset){
    vec2 uv;

    if(_st.x>0.5){
        uv.x = _st.x - 0.5;
    } else {
        uv.x = _st.x + 0.5;
    }

    if(_st.y>0.5){
        uv.y = _st.y - 0.5;
    } else {
        uv.y = _st.y + 0.5;
    }

    return uv;
}

void main(void){
    vec2 st = gl_FragCoord.xy/u_resolution.xy;
    st.y *= u_resolution.y/u_resolution.x;

    st = tile(st,10.);

    vec2 offsetSt = offset(st,vec2(0.5));

    st = rotate2D(st,PI*0.25);

    vec3 color = vec3( box(offsetSt,vec2(0.95),0.01) - box(st,vec2(0.3),0.01) + 2.*box(st,vec2(0.2),0.01) );

    gl_FragColor = vec4(color,1.0);
}
```

3.   

   ![Vector Pattern Scottish Tartan By Kavalenkava](https://thebookofshaders.com/09/tartan.jpg)

```glsl
// Author:MeZinc
// Title:Scottish Tartan Patterns

#ifdef GL_ES
precision mediump float;
#endif

#define PI 3.14159265359

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

vec2 tile(vec2 st, float zoom){
    st *= zoom;
    return fract(st);
}

float strip_horizontal(in float sty, in float width, in float height,in float sp, in vec2 _st,in float offset)
{
	float total = width + sp;
	
	float fx = mod(_st.x - width * floor((_st.y - sty)/width + offset),total);
	fx = step(fx, sp) ;
    
	float fy = step(sty,_st.y) * step(_st.y,sty+height) ;
	return fx *fy;
}
float strip_vertical(in float sty, in float width, in float height,in float sp, in vec2 _st,in float offset)
{
	float total = width + sp;
	
	float fx = mod(_st.y - width * floor((_st.x - sty + offset)/width),total);
	fx =1. - step(fx, sp) ;
    
	float fy = step(sty,_st.x) * step(_st.x,sty+height) ;
	return fx *fy;
}
vec2 rotate2D(vec2 _st, float _angle){
    _st -= 0.5;
    _st =  mat2(cos(_angle),-sin(_angle),
                sin(_angle),cos(_angle)) * _st;
    _st += 0.5;
    return _st;
}
float basestyle(in vec2 _st)
{
    _st = tile(_st,30.);
    vec2 uv = smoothstep(1. / 3. - 0.02,1. / 3. - 0.01, _st) ;/
    vec2 uv2 =step(1. / 3. , _st) * smoothstep(0.99, 0.98, _st);
    vec2 uv3 = smoothstep(1. / 3.- 0.02, 1. / 3. - 0.01, _st) * smoothstep(1. / 3.- 0.02, 1. / 3. - 0.01, 1. - _st);
    
    float t1 = clamp(uv.x*uv.y - uv2.x*uv2.y -uv3.x*uv3.y,0.,1.);
    
    vec2 uv4 = step(1. / 3. - 0.02, 1. - _st);
    vec2 uv5 = smoothstep(1. / 3. - 0.01, 1. / 3., 1. - _st);
    vec2 uv6 = smoothstep(1. / 3., 1./3. - 0.01,_st);
    
    t1 = clamp(t1 + uv4.x * uv4.y - uv5.x * uv5.y - uv6.x * uv6.y, 0., 1.);
    vec2 uv7 = smoothstep(1. / 3., 1. / 3. - 0.01, _st);
    uv7 = 1. -step(1./3.,_st);
    vec2 uv8 = smoothstep(1. / 3. - 0.01, 1./ 3. - 0.02, _st);
    t1 = t1 + uv7.x * uv7.y - uv8.x * uv8.y;
    t1= clamp(t1,0.,1.);
    return 1. - t1;
}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy;
    st.x *= u_resolution.x/u_resolution.y;
    
    st -= vec2(0.370,-0.220);
    st = rotate2D(st,PI/6.);
    st = tile(st,1.736);

    vec3 color = vec3(0.5);
    
    color = basestyle(st)*vec3(0.654,0.676,0.705);
    
    color = mix(color, vec3(1.0,0.,0.), strip_horizontal(0., 0.04 / 8., 0.02, 0.12 / 8.,st, 0.));
    color = mix(color, vec3(0.0), strip_horizontal(0.4, 0.05 / 6., 0.05, 0.1 / 6.,st, 0.));
    color = mix(color, vec3(0.9), strip_horizontal(0.45, 0.05 / 6., 0.05, 0.1 / 6.,st, 0.));
    color = mix(color, vec3(0.0), strip_horizontal(0.5, 0.05 / 6., 0.05, 0.1 / 6.,st, 0.));
    color = mix(color, vec3(0.9), strip_horizontal(0.55, 0.05 / 6., 0.05, 0.1 / 6.,st, 0.));
    color = mix(color, vec3(0.0), strip_horizontal(0.6, 0.05 / 6., 0.05, 0.1 / 6.,st, 0.));
    
    color = mix(color, vec3(1.0,0.,0.), strip_vertical(0., 0.04 / 8., 0.046, 0.12 / 8.,st, 0.));
    color = mix(color, vec3(0.0), strip_vertical(0.4, 0.05 / 6., 0.05, 0.1 / 6.,st, 0.1 / 6.));
    color = mix(color, vec3(0.9), strip_vertical(0.45, 0.05 / 6., 0.05, 0.1 / 6.,st, 0.));
    color = mix(color, vec3(0.0), strip_vertical(0.5, 0.05 / 6., 0.05, 0.1 / 6.,st, 0.1 / 6.));
    color = mix(color, vec3(0.9), strip_vertical(0.55, 0.05 / 6., 0.05, 0.1 / 6.,st, 0.));
    color = mix(color, vec3(0.0), strip_vertical(0.6, 0.05 / 6., 0.05, 0.1 / 6.,st, 0.1 / 6.));
    gl_FragColor = vec4(color,1.0);
}
```

存在的问题：

+ 浮点数精度问题，背景纹理不清晰（应该是）
+ 条纹样式固定（主要懒得再改了）

尽管没有还原，本节知识的内容已经理解了，精度问题还要继续阅读看是否有解，或者直接用`gl_FragCoord`，而不进行归一化来进行计算。

### 偏移Offset

前一节练习代码中的 条纹（strip）函数 已经使用过了类似技巧了。

练习：Marching Dots

```glsl
// Author:MeZinc
// Title:marching dots

#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float func(float x)
{
	float f1 = step(1.,mod(x,2.));
	float f2 = floor(x / 2.);
	return (x - f2)*(1.-f1) +floor((x+1.)/2.)* f1;
}

vec2 balltile(vec2 _st, float zoom){
    float stepDis = 1.0 / zoom;
    _st *= zoom ;
    
    vec2 speed =2.0 * (step(1.0, mod(vec2(_st.y,_st.x), 2.))- 0.5);
    
    float f1 = step(1.,mod(u_time,2.));
	 float f2 = floor(u_time / 2.);
	 float y =(u_time - f2)*(1.-f1) +floor((u_time+1.)/2.)* f1;
    
    float f3 = mod( speed.x * func(u_time), 1.);
    
    float f4 = mod( speed.y * func(u_time + 1.), 1.);
    _st += vec2(f3,f4);
    
    return fract(_st);
}

float circle(vec2 _st, float _radius){
    vec2 pos = vec2(0.5)-_st;
    return 1. - smoothstep(1.0-_radius,1.0-_radius+_radius*0.2,1.-dot(pos,pos)*3.14);
}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy;
    st.x *= u_resolution.x/u_resolution.y;

    st = balltile(st,10.);
    vec3 color = vec3(st,0.0);
    
    color = circle(st, 0.25) * vec3(1.);
    //color = vec3(st,0.0);

    gl_FragColor = vec4(color,1.0);
}
```

原文使用了分支（if）

###  Truchet Tiles

Truchet是这个样式的提出提出者，详见[https://en.wikipedia.org/wiki/Truchet_tiles](https://en.wikipedia.org/wiki/Truchet_tiles)。[其它样例](https://www.pinterest.co.kr/zaueqh/truchet-tiling/)

### 更多样式

构造样式就是寻找最小可重复元素。阅读[decorative](https://archive.org/stream/traditionalmetho00chririch#page/130/mode/2up)会有更多发现。



# Generative designs

### 随机

从`y = fract(sin(x) * 1.0)`开始，系数到100000.0的过程。

![](assets/001/random.gif)  

伪随机，而且分布中间集中边缘分散。

```glsl
float random (in float x) { return fract(sin(x)*1e4); }
```



### 二维随机

使用了dot来使二维向量转为了一维的float值

```glsl
float random (vec2 st) 
{
    return fract(sin(dot(st.xy,
                         vec2(12.9898,78.233)))*
        43758.5453123);
}
```

### 应用

```glsl
st *= 10.0;
vec3 ipos = floor(st);
vec3 color = vec3(random(ipos));//10*10的随机序列;可以将这个序列分别应用到10*10的TruchePattern上，random返回值归化为4个值作为每个小块的旋转角度（0,90,180，270）
```

```glsl
//例如文中使用的用来Tile的函数对每个部分进行了旋转。（因为只有4个角度，没有用旋转矩阵，直接应用了对应角度的转换方法）
vec2 truchetPattern(in vec2 _st, in float _index){
    _index = fract(((_index-0.5)*2.0));
    if (_index > 0.75) {
        _st = vec2(1.0) - _st;
    } else if (_index > 0.5) {
        _st = vec2(1.0-_st.x,_st.y);
    } else if (_index > 0.25) {
        _st = 1.0-vec2(1.0-_st.x,_st.y);
    }
    return _st;
}
```

练习：

1. Ikeda Data Stream：这个练习中每行的随机分布控制没有做到，最后参考了原文代码（pattern函数中的`random(100.+p*.00001)`部分）；原文数据计算安排更合理，我写的有重复计算；原文对颜色rgb三个通道的坐标间有一个极小的offset。

```glsl
// Author:MeZinc
// Title:Ikeda Data Stream

#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float random (in float x) { return fract(sin(x) * 1e4); }
float random (in vec2 st) 
{
    return fract(sin(dot(st.xy, vec2(12.9898, 78.233))) * 43758.5453123);
}
float randomserie(in vec2 _st, vec2 freq, float t,in float pct)
{
    _st *= freq;
    vec2 p = floor( _st- vec2(t,0.));
	return step(pct,random(100.+p*.00001) + 0.5 * random(p.x));
}

float pattern(vec2 st, vec2 v, float t) {
    vec2 p = floor(st+v);
    return step(t, random(100.+p*.00001)+random(p.x)*0.5 );
}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy;
    st.x *= u_resolution.x/u_resolution.y;
    vec2 mousePositon = u_mouse / u_resolution;
    
    float col = 50.0;

    vec3 color = vec3(0.0);
	 vec2 freq = vec2(100.0,col);
    float streamingSpeed = 30.0 * (1. + random(floor( col * st.y )) );
    
    
	 float t = floor(u_time * streamingSpeed);
    
	 color =vec3(randomserie(st, freq, t,  mousePositon.x+0.5));
    
    //color = vec3(pattern(st*freq, vec2(t,0.), mousePositon.x+0.5));
    color *= step(0.2, fract(st.y * col));

    gl_FragColor = vec4(1. - color,1.0);
}

```

2. DeFrag ：打字出现，鼠标x与密度关联，y与出现速度关联，方向一行左一行右，运动速度也一致.效果未实现完。Margin、smooth和threshold部分之后还要再分析补充。

   ```glsl
   // Author:MeZinc
   // Title:Ikeda Data Stream
   
   #ifdef GL_ES
   precision mediump float;
   #endif
   
   uniform vec2 u_resolution;
   uniform vec2 u_mouse;
   uniform float u_time;
   
   float random (in float x) { return fract(sin(x) * 1e4); }
   float random (in vec2 st) 
   {
       return fract(sin(dot(st.xy, vec2(12.9898, 78.233))) * 43758.5453123);
   }
   
   void main() {
       vec2 st = gl_FragCoord.xy/u_resolution.xy;
       st.x *= u_resolution.x/u_resolution.y;
       vec2 mousePositon = u_mouse / u_resolution;
       
   	 vec2 grid = vec2(100.0,50.0);
       vec3 color = vec3(1.0);
       
       st *= grid;
       
       vec2 ipos = floor(st);
       vec2 fpos = fract(st);
       
       float speed = 2. * ( step(1.0 ,mod(ipos.y , 2.0)) - 0.5) * floor(random(ipos.y+1.0) * u_time *15.0) ;
       
       color = step(mousePositon.x,random(ipos + vec2(speed,0.0))  ) * vec3(1.0);
       
       float type = step(0.0,mod(u_time * grid.x * mousePositon.y * 10., grid.x * grid.y) - (ipos.x +(grid.y - ipos.y)*grid.x)) ;
       color *=type;
       
       //color = vec3(pattern(st*freq, vec2(t,0.), mousePositon.x+0.5));
       color *= step(0.2, fpos.y);
   
       gl_FragColor = vec4(1. - color,1.0);
   }
   ```

   

```glsl
// Author @patriciogv - 2015
// Title: DeFrag

#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float random (in float x) { return fract(sin(x)*1e4); }
float random (in vec2 _st) { return fract(sin(dot(_st.xy, vec2(12.9898,78.233)))* 43758.5453123);}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy;
    st.x *= u_resolution.x/u_resolution.y;

    // Grid
    vec2 grid = vec2(100.0,50.);
    st *= grid;

    vec2 ipos = floor(st);  // integer

    vec2 vel = floor(vec2(u_time*10.)); // time
    vel *= vec2(-1.,0.); // direction

    vel *= (step(1., mod(ipos.y,2.0))-0.5)*2.; // Oposite directions
    vel *= random(ipos.y); // random speed

    // 100%
    float totalCells = grid.x*grid.y;
    float t = mod(u_time*max(grid.x,grid.y)+floor(1.0+u_time*u_mouse.y),totalCells);
    vec2 head = vec2(mod(t,grid.x), floor(t/grid.x));

    vec2 offset = vec2(0.1,0.);

    vec3 color = vec3(1.0);
    color *= step(grid.y-head.y,ipos.y);                                // Y
    color += (1.0-step(head.x,ipos.x))*step(grid.y-head.y,ipos.y+1.);   // X
    color = clamp(color,vec3(0.),vec3(1.));

    // Assign a random value base on the integer coord
    color.r *= random(floor(st+vel+offset));
    color.g *= random(floor(st+vel));
    color.b *= random(floor(st+vel-offset));

    color = smoothstep(0.,.5+u_mouse.x/u_resolution.x*.5,color*color); // smooth
    color = step(0.5+u_mouse.x/u_resolution.x*0.5,color); // threshold

    //  Margin
    color *= step(.1,fract(st.x+vel.x))*step(.1,fract(st.y+vel.y));

    gl_FragColor = vec4(1.0-color,1.0);
}
```



## 噪声 Noise

*smooth randomness*

```glsl
float i = floor(x);  // integer
float f = fract(x);  // fraction

y = rand(i); //rand() is described in the previous chapter
y = mix(rand(i), rand(i + 1.0), f);
y = mix(rand(i), rand(i + 1.0), smoothstep(0.,1.,f));

float u = f * f * (3.0 - 2.0 * f ); // custom cubic curve
y = mix(rand(i), rand(i + 1.0), u); // using it in the interpolation
```



```glsl
float noise(float x)
{
    float i = floor(x);
    float u = f * f * (3.0 - 2.0 * f );
    return mix(rand(i), rand(i + 1.0), u);
}
```

二维噪声的插值（4个值）

```glsl
// By Morgan McGuire @morgan3d, http://graphicscodex.com
// https://www.shadertoy.com/view/4dS3Wd
float hash(float p) 
{ 
    p = fract(p * 0.011); 
    p *= p + 7.5; 
    p *= p + p; 
    return fract(p); 
}
float hash(vec2 p)
{
    vec3 p3 = fract(vec3(p.xyx) * 0.13); 
    p3 += dot(p3, p3.yzx + 3.333); 
 	return fract((p3.x + p3.y) * p3.z); 
}
float noise(vec2 x) {
    vec2 i = floor(x);
    vec2 f = fract(x);

	// Four corners in 2D of a tile
	float a = hash(i);
    float b = hash(i + vec2(1.0, 0.0));
    float c = hash(i + vec2(0.0, 1.0));
    float d = hash(i + vec2(1.0, 1.0));

    // Simple 2D lerp using smoothstep envelope between the values.
	// return vec3(mix(mix(a, b, smoothstep(0.0, 1.0, f.x)),
	//			mix(c, d, smoothstep(0.0, 1.0, f.x)),
	//			smoothstep(0.0, 1.0, f.y)));

	// Same code, with the clamps in smoothstep and common subexpressions
	// optimized away.
    // Cubic Hermine Curve.  Same as SmoothStep()
    vec2 u = f * f * (3.0 - 2.0 * f);
	return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
}
```

练习：[Mark Rothko](http://en.wikipedia.org/wiki/Mark_Rothko) painting：写了一个在rect边缘应用噪声的函数

```glsl
float noise_rect_stepVersion(vec2 size, vec2 center,in vec2 st,float noiseScale)
{
	vec2 blp = center - size/2.0;
	vec2 trp = vec2(1.0) - center - size/2.0;
	vec2 bl = step(noise(st*ns) * 0.01 + blp ,st );//0.01是边缘程度
	vec2 tr = step(noise(st*ns) * 0.01 + trp ,1 - st);
	return bl.x * bl.y * tr.x * tr.y;
}
```

或者直接在调用以前的rect函数，size根据st变化,如下

```glsl
float noiseSize = 100;//噪声粒度
float edgeLenth = 0.01;//噪声边缘大小
vec2 rectSize = 0.5;
vec2 rectCenter = 0.5;

vec3 color = vec3(1.0);

color = noise_rect_stepVersion(
    vec2(noise(st * noiseSize) * edgeLenth) + rectSize,
    rectCenter,
    st
) * color;
```

### 生成设计的噪声应用

之前的噪声都是使用的**值噪声**（**Value Noise**），即通过一维、二维插值得到的，有个非常明显的特点就是，方块感非常严重，如下 [value noise](https://www.shadertoy.com/view/lsf3WH) 

![Inigo Quilez - Value Noise](https://thebookofshaders.com/11/value-noise.png)

1985年[Ken Perlin](https://mrl.nyu.edu/~perlin/)开发出了**Gradient Noise**，使用了一个二维随机函数返回一个二维向量`vec2`生成的*gradiant*来取代固定的`float`值来进行插值。每个点上生成随机二维向量，这个向量和st值点乘即可得到随st变化而渐变的*gradiant*。 [gradient noise](https://www.shadertoy.com/view/XdXGW8)

![Inigo Quilez - Gradient Noise](https://thebookofshaders.com/11/gradient-noise.png)

```glsl
vec2 random2(vec2 st){
    st = vec2( dot(st,vec2(127.1,311.7)),
              dot(st,vec2(269.5,183.3)) );
    return -1.0 + 2.0*fract(sin(st)*43758.5453123);
}

// Gradient Noise by Inigo Quilez - iq/2013
// https://www.shadertoy.com/view/XdXGW8
float noise(vec2 st) {
    vec2 i = floor(st);
    vec2 f = fract(st);

    vec2 u = f*f*(3.0-2.0*f);

    return mix( mix( dot( random2(i + vec2(0.0,0.0) ), f - vec2(0.0,0.0) ),
                     dot( random2(i + vec2(1.0,0.0) ), f - vec2(1.0,0.0) ), u.x),
                mix( dot( random2(i + vec2(0.0,1.0) ), f - vec2(0.0,1.0) ),
                     dot( random2(i + vec2(1.0,1.0) ), f - vec2(1.0,1.0) ), u.x), u.y);
}

    color = vec3( noise(pos)*.5+.5 );
```

应用：

[improve noise](assets/001/paper445_improvenoise.pdf)插值曲线改进

[simplexnoise](assets/001/simplexnoise.pdf) Ian McEwan的论文，simplex noise算法再GLSL中的应用。

1. Wood texture：将噪声应用在画布旋转上，由于噪声图相邻位置平滑过渡，相邻位置的旋转角度也是连续的

![Wood texture](https://thebookofshaders.com/11/wood-long.png)

```glsl
pos = rotate2d( noise(pos) ) * pos; // rotate the space
pattern = lines(pos,.5); // draw lines
```

2. 距离场:直接将噪声当做距离场

```glsl
color += smoothstep(.15,.2,noise(st*10.)); // Black splatter
color -= smoothstep(.35,.4,noise(st*10.)); // Holes on splatter
```

3. 形状动画：平移噪声图，再将噪声按坐标应用，形成了连续的动画

```glsl
float noiseOnEdge =  sin(a * 50.) * noise(pos  + u_time * 0.2) ;
vec3 color = vec3(circle(d , 0.39 + noiseOnEdge) - circle(d , 0.4 + noiseOnEdge));
```

4. 位移动画： 平移噪声图，选取固定两点值作为位移向量，这样就能得到随机连续变化的位移



## Celluar Noise

[worley_Celluar_Texture_basic_function.pdf](assets/001/worley_Celluar_Texture_basic_function.pdf)

一定数量的特征点，对每个像素存下该像素到所有特征点的最短距离

```glsl
st*=10.0;
vec2 ipos = floor(st);
vec2 fpos = fract(st);
	
//point = random(ipos);//特征点
	 
float dist = 100.;
for (int y= -1; y <= 1; y++) 
{
    for (int x= -1; x <= 1; x++) 
    {
        // Neighbor place in the grid
        vec2 neighbor = vec2(float(x),float(y));
        vec2 point = ipos + neighbor;
		point = random2(point);
		point = 0.5 + 0.5 * sin(point*2*3.14 + u_time);
		vec2 diff = neighbor + point - fpos;
		dist = min(length(diff),dist);
		
		// if( dist < m_dist ) 
        // {
        // 		m_dist = dist;
        // 		m_point = point;
        // }
		//可以存储最近的point
    }
}

vec3 color = vec3(dist);
```

一个优化算法，仅仅对一个 2x2 的矩阵作遍历（而不是 3x3 的矩阵）。这显著地减少了工作量，但是会在网格边缘制造人工痕迹。

 [Inigo Quilez wrote an article on how to make precise Voronoi borders](http://www.iquilezles.org/www/articles/voronoilines/voronoilines.htm).2014 年，他写了一篇非常漂亮的文章，提出一种他称作为 [voro-noise](http://www.iquilezles.org/www/articles/voronoise/voronoise.htm) 的噪声。

## Fractal Brownian Motion 分形布朗

将一个噪声按一定数值**提升频率**（**lacunarity**），缩小**幅度**（**gain**）,连续叠加一定**次数**（**octaves**）的方法称为分形布朗**fBM**。下面是个简单实现的公式：
$$
fBM(x)=\sum_{i=0}^{octaves}A \cdot gain^{i} \cdot f(x*lancunarity^i)
$$
很有分形特点，细节增加，分为不同层次，都是相似的，如果无限叠加下去就会得到数学中真正的分形图像。

二维分形布朗：

```glsl

#define OCTAVES 6  

float fbm(in vec2 st)
{
    float amplitude = 0.5;
    float frequency = 1.0;

    float value = 0.0;

    float gain = 0.5;
    float lacunarity = 2.0;

    for(int i = 0; i < OCTAVES ; i++)
    {
        value += amplitude * noise(frequency * st);
        frequency *= lacunarity;
        amplitude *= gain;
    }
    return value;
}
```

这种噪声很适合生成山脉地形，因为现实中大范围的山脉受到气候侵蚀也有这种自相似性。 read [this great article by Inigo Quiles about advanced noise](http://www.iquilezles.org/www/articles/morenoise/morenoise.htm).

[ "Texturing and Modeling: a Procedural Approach" (3rd edition), by Kenton Musgrave](assets/001/Texturing and Modeling - A Procedural Approach.pdf)

> **Turbulence**. It's essentially an fBm, but constructed from the absolute value of a signed noise to create sharp valleys in the function.

湍流，这个液体流动感觉不像湍流那种。不过意思get了。

```glsl
for (int i = 0; i < OCTAVES; i++) {
    value += amplitude * abs(snoise(st));//simplex noise 文中使用的snoise来自下面链接
    st *= 2.;
    amplitude *= .5;
}
```

> **Ridge**, where the sharp valleys are turned upside down to create sharp ridges instead:

山脊

```glsl
    n = abs(n);     // create creases
    n = offset - n; // invert so creases are at top
    n = n * n;      // sharpen creases
```

[ashima/webgl-noise](https://github.com/ashima/webgl-noise/wiki)

之前每部分噪声相乘而非相加。前一个噪声来缩放后一个噪声（multifractals），参考资料[ "Texturing and Modeling: a Procedural Approach" (3rd edition), by Kenton Musgrave](assets/001/Texturing and Modeling - A Procedural Approach.pdf)

### Domain Warping 域翘曲

[Inigo Quiles wrote this other fascinating article](http://www.iquilezles.org/www/articles/warp/warp.htm) 使用fBM来扭曲fBM，Warping( pinch an object, strech it, twist it, bend it, make it thicker or apply [any deformation you want](https://www.iquilezles.org/www/articles/distfunctions/distfunctions.htm).)这翻译好高端。

> A useful tool for this is to displace the coordinates with the derivative (gradient) of the noise.

 [A famous article by Ken Perlin and Fabrice Neyret called "flow noise"](http://evasion.imag.fr/Publications/2001/PN01/)

> Some modern implementations of Perlin noise include a variant that computes both the function and its analytical gradient. If the "true" gradient is not available for a procedural function, you can always compute finite differences to approximate it, although this is less accurate and involves more work.

finite differences有限差分法？(这段还不懂，例如 额外计算analytical gradient是什么有什么用处，"true" gradient 指的什么compute finite differences如何计算，有待后续学习来补充)

[https://thebookofshaders.com/edit.php#12/2d-cnoise-2x2.frag](https://thebookofshaders.com/edit.php#12/2d-cnoise-2x2.frag)中也有返回f1,f2，参考文章在前面有

```glsl
// Cellular noise, returning F1 and F2 in a vec2.
// Speeded up by using 2x2 search window instead of 3x3,
// at the expense of some strong pattern artifacts.
// F2 is often wrong and has sharp discontinuities.
// If you need a smooth F2, use the slower 3x3 version.
// F1 is sometimes wrong, too, but OK for most purposes.
vec2 cellular2x2(vec2 P) {
	#define K 0.142857142857 // 1/7
	#define K2 0.0714285714285 // K/2
	#define jitter 0.8 // jitter 1.0 makes F1 wrong more often
	vec2 Pi = mod(floor(P), 289.0);
 	vec2 Pf = fract(P);
	vec4 Pfx = Pf.x + vec4(-0.5, -1.5, -0.5, -1.5);
	vec4 Pfy = Pf.y + vec4(-0.5, -0.5, -1.5, -1.5);
	vec4 p = permute(Pi.x + vec4(0.0, 1.0, 0.0, 1.0));
	p = permute(p + Pi.y + vec4(0.0, 0.0, 1.0, 1.0));
	vec4 ox = mod(p, 7.0)*K+K2;
	vec4 oy = mod(floor(p*K),7.0)*K+K2;
	vec4 dx = Pfx + jitter*ox;
	vec4 dy = Pfy + jitter*oy;
	vec4 d = dx * dx + dy * dy; // d11, d12, d21 and d22, squared
	// Sort out the two smallest distances
#if 0
	// Cheat and pick only F1
	d.xy = min(d.xy, d.zw);
	d.x = min(d.x, d.y);
	return d.xx; // F1 duplicated, F2 not computed
#else
	// Do it right and find both F1 and F2
	d.xy = (d.x < d.y) ? d.xy : d.yx; // Swap if smaller
	d.xz = (d.x < d.z) ? d.xz : d.zx;
	d.xw = (d.x < d.w) ? d.xw : d.wx;
	d.y = min(d.y, d.z);
	d.y = min(d.y, d.w);
	return sqrt(d.xy);
#endif
}
```



