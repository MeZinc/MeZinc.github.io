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

