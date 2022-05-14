# 内容

## 基础

### 形状

#### 绘制一个点

顶点着色器（`15-22`行）

```glsl
void main() {
  //给内置变量gl_PointSize赋值像素大小
  gl_PointSize=20.0;
  //顶点位置，位于坐标原点
  gl_Position =vec4(0.0,0.0,0.0,1.0);
}
```

片元着色器（`26-32`行）

```glsl
void main() {
  gl_FragColor = vec4(1.0,0.0,0.0,1.0);
}
```

所用知识点（内置变量）

- `gl_PointSize`：当WebGL执行绘制函数`gl.drawArrays()`绘制模式是点模式`gl.POINTS`的时候，顶点着色器语言`main`函数中才会用到内置变量`gl_PointSize`，使用内置变量`gl_PointSize`主要是用来设置顶点渲染出来的方形点像素大小。

- `gl_Position`：主要和顶点相关，出现的位置是顶点着色器语言的`main`函数中。`gl_Position`内置变量表示最终传入片元着色器片元化要使用的顶点位置坐标。

  如果只有一个顶点，直接在给顶点着色器中设置内置变量`gl_Position`赋值就可以，内置变量`gl_Position`的值是四维向量`vec4(x,y,z,1.0)`,前三个参数表示顶点的xyz坐标值，第四个参数是浮点数`1.0`。

- `gl_FragColor`：主要用来设置片元像素的颜色，出现的位置是片元着色器语言的`main`函数中。

  内置变量`gl_Position`的值是四维向量`vec4(r,g,b,a)`,前三个参数表示片元像素颜色值RGB，第四个参数是片元像素透明度A，`1.0`表示不透明,`0.0`表示完全透明。



#### 绘制一个矩形

顶点着色器（`14-21`行）

```glsl
//attribute声明vec4类型变量apos
attribute vec4 apos;
void main() {
  //顶点坐标apos赋值给内置变量gl_Position
  //逐顶点处理数据
  gl_Position = apos;
}
```

片元着色器（`25-30`行）

```glsl
void main() {
  // 逐片元处理数据，所有片元(像素)设置为红色
  gl_FragColor = vec4(1.0,0.0,0.0,1.0);
}
```

顶点数组（`52-60`行）

```js
//类型数组构造函数Float32Array创建顶点数组
var data=new Float32Array([
  //右上
  0.5,0.5,
  //左上
  -0.5,0.5,
  //左下
  -0.5,-0.5,
  //右下
  0.5,-0.5]);
```

所用知识点

- `attribute`关键字：通常用来声明与顶点数据相关的变量，比如顶点位置坐标数据、顶点颜色数据、顶点法向量数据...

  顶点着色器中通过`attribute`关键字声明的顶点变量，javascript代码可以通过相关的WebGL API把顶点的数据传递给着色器中相应的顶点变量。

  因为javascript没必要给片元着色器传递顶点数据，所以规定`attribute`关键字只能在顶点着色器中声明变量使用。只要注意`attribute`关键字声明顶点变量代码位于主函数`main`之外就可以。



#### 绘制一个立方体

顶点着色器（`10-36`行）

```glsl
//attribute声明vec4类型变量apos
attribute vec4 apos;
void main() {
  //设置几何体轴旋转角度为30度，并把角度值转化为弧度值
  float radian = radians(30.0);
  //求解旋转角度余弦值
  float cos = cos(radian);
  //求解旋转角度正弦值
  float sin = sin(radian);
  //引用上面的计算数据，创建绕x轴旋转矩阵
  // 1      0       0    0
  // 0   cosα   sinα   0
  // 0  -sinα   cosα   0
  // 0      0        0   1
  mat4 mx = mat4(1,0,0,0,  0,cos,-sin,0,  0,sin,cos,0,  0,0,0,1);
  //引用上面的计算数据，创建绕y轴旋转矩阵
  // cosβ   0   sinβ    0
  //     0   1   0        0
  //-sinβ   0   cosβ    0
  //     0   0   0        1
  mat4 my = mat4(cos,0,-sin,0,  0,1,0,0,  sin,0,cos,0,  0,0,0,1);
  //两个旋转矩阵、顶点齐次坐标连乘
  gl_Position = mx*my*apos;
}
```

片元着色器（`38-45`行）

```glsl
void main() {
  // 逐片元处理数据，所有片元(像素)设置为红色
  gl_FragColor = vec4(1.0,0.0,0.0,1.0);
}
```

所用知识

- 内置函数：

  - `radians()`函数：角度值转化为弧度制，参数是浮点数`float`
  - `cos()`是余弦函数，参数要求是弧度值且是浮点数`float`
  - `sin()`是正弦函数，参数要求是弧度值且是浮点数`float`

- 平移矩阵

  ```
  | 1  0  0  Tx |   | x |   | x+Tx |
  | 0  1  0  Ty | x | y | = | y+Ty |
  | 0  0  1  Tz |   | z |   | z+Tz |
  | 0  0  0  1  |   | 1 |   |  1   |
  ```

- 缩放矩阵

  ```
  | Sx 0  0  0 |   | x |   | x*Sx |
  | 0  Sy 0  0 | x | y | = | y*Sy |
  | 0  0  Sz 0 |   | z |   | z*Sz |
  | 0  0  0  1 |   | 1 |   |  1   |
  ```

- 旋转矩阵

  - 绕`x`轴旋转`α`度对应的旋转矩阵

  ```
  | 1  0     0     0 |   | x |   |       x       |
  | 0  cosα  -sinα 0 | x | y | = | cosα*y-sinα*z |
  | 0  sinα  cosα  0 |   | z |   | sinα*y+cosα*z |
  | 0  0     0     1 |   | 1 |   |        1      |
  ```

  - 绕`y`轴旋转`α`度对应的旋转矩阵

  ```
  | cosα  0  -sinα 0 |   | x |   |  cosα*x+sinα*z |
  | 0     1  0     0 | x | y | = |        y       |
  | sinα  0  cosα  0 |   | z |   | -sinα*x+cosα*z |
  | 0     0  0     1 |   | 1 |   |        1       |
  ```

  - 绕`z`轴旋转`α`度对应的旋转矩阵Rz

  ```
  | cosα  -sinα 0  0 |   | x |   |  cosα*x-sinα*y |
  | sinα  cosα  0  0 | x | y |   |  sinα*x+cosα*y |
  | 0     0     1  0 |   | z |   |        z       |
  | 0     0     0  1 |   | 1 |   |        1       |
  ```



### 颜色

#### 设置渐变色

顶点着色器（`15-24`行）

```glsl
void main() {
  //给内置变量gl_PointSize赋值像素大小
  gl_PointSize=500.0;
  //顶点位置，位于坐标原点
  gl_Position =vec4(0.0,0.0,0.0,1.0);
}
```

片段着色器（`26-32`行）

```glsl
void main() {
  gl_FragColor = vec4(gl_FragCoord.x/500.0*1.0,gl_FragCoord.x/500.0*1.0,0.0,1.0);
}
```

所用知识点

- `gl_FragCoord`：表示WebGL在canvas画布上渲染的所有片元或者说像素的坐标，坐标原点是canvas画布的左上角，x轴水平向右，y竖直向下，`gl_FragCoord`坐标的单位是像素，`gl_FragCoord`的值是`vec2(x,y)`,通过`gl_FragCoord.x`、`gl_FragCoord.y`方式可以分别访问片元坐标的纵横坐标。

  

#### 颜色插值

顶点着色器（`10-24`行）

```glsl
//attribute声明vec4类型变量apos
attribute vec4 apos;
// attribute声明顶点颜色变量
attribute vec4 a_color;
//varying声明顶点颜色插值后变量
varying vec4 v_color;
void main() {
  // 顶点坐标apos赋值给内置变量gl_Position
  gl_Position = apos;
  //顶点颜色插值计算
  v_color = a_color;
}
```

片段着色器（`26-36`行）

```glsl
// 所有float类型数据的精度是lowp
precision lowp float;
// 接收顶点着色器中v_color数据
varying vec4 v_color;
void main() {
  // 插值后颜色数据赋值给对应的片元
  gl_FragColor = v_color;
}
```

顶点和颜色数据（`56-60`行）

```js
/**
     创建顶点位置数据数组data，存储两个顶点(-0.5,0.5)、(0.5,0.5)
     创建顶点颜色数组colorData，存储两个顶点对应RGB颜色值(0,0,1)、(1,0,0)
     **/
var data=new Float32Array([-0.5,0.5,0.5,0.5]);
var colorData = new Float32Array([0,0,1,1,0,0]);
```

所用知识点

- `varying`关键字：如果想在片元着色器中获得顶点颜色插值计算以后的数据，需要同时在顶点着色器和片元着色器中执行`varying vec4 v_color;`，也就是在顶点、片元两个着色器代码中都需要通过关键字`varying`声明一个新变量`v_color`,最后再顶点着色器中执行`v_color = a_color;`即可。

- 着色器运算精度设置：除了片元着色器浮点数的精度意外，WebGL系统其它的数值精通全部设置了默认精度，所以如果片元着色器中使用了浮点数类型的变量，比如声明精度，否则会报错。

  片元着色器中设置浮点数变量精度，可以单独设置一个变量，也可以使用关键字`precision`批量设置。

  - `lowp`：低精度
  - `mediump`：中精度
  - `highp`：高精度

  

#### 插值三角形

顶点着色器（`10-24`行）

```glsl
//attribute声明vec4类型变量apos
attribute vec4 apos;
// attribute声明顶点颜色变量
attribute vec4 a_color;
//varying声明顶点颜色插值后变量
varying vec4 v_color;
void main() {
  // 顶点坐标apos赋值给内置变量gl_Position
  gl_Position = apos;
  //顶点颜色插值计算
  v_color = a_color;
}
```

片段着色器（`26-37`行）

```glsl
// 所有float类型数据的精度是lowp
// 只有float类型数据需要写着一句声明
precision lowp float;
// 接收顶点着色器中v_color数据
varying vec4 v_color;
void main() {
  // 插值后颜色数据赋值给对应的片元
  gl_FragColor = v_color;
}
```

顶点和颜色数据（`55-60`行）

```js
/**
     创建顶点位置数据数组data，存储两个顶点(-0.5,0.5)、(0.5,0.5)
     创建顶点颜色数组colorData，存储两个顶点对应RGB颜色值(0,0,1)、(1,0,0)
     **/
var data=new Float32Array([-0.5,0.5,0.5,0.5]);
var colorData = new Float32Array([0,0,1,1,0,0]);
```



#### 设置两个纯色三角形

顶点着色器（`10-24`行）

```glsl
//attribute声明vec4类型变量apos
attribute vec4 apos;
// attribute声明顶点颜色变量
attribute vec4 a_color;
//varying声明顶点颜色插值后变量
varying vec4 v_color;
void main() {
  // 顶点坐标apos赋值给内置变量gl_Position
  gl_Position = apos;
  //顶点颜色插值计算
  v_color = a_color;
}
```

片段着色器（`26-36`行）

```glsl
// 所有float类型数据的精度是lowp
precision lowp float;
// 接收顶点着色器中v_color数据
varying vec4 v_color;
void main() {
  // 插值后颜色数据赋值给对应的片元
  gl_FragColor = v_color;
}
```

所用顶点和颜色数据

```js
/**
     创建顶点位置数据数组data，存储6个顶点
     创建顶点颜色数组colorData，存储6个顶点对应RGB颜色值
     **/
var data=new Float32Array([
  -0.5,0.5,0.5,0.5,0.5,-0.5,//第一个三角形的三个点
  -0.5,0.5,0.5,-0.5,-0.5,-0.5//第二个三角形的三个点
]);
var colorData = new Float32Array([
  1,0,0,1,0,0,1,0,0,//三个红色点
  0,0,1,0,0,1,0,0,1//三个蓝色点
]);
```



### 光照

#### 平行光源

顶点着色器（`10-576行）

```glsl
//attribute声明vec4类型变量apos
attribute vec4 apos;
// attribute声明顶点颜色变量
attribute vec4 a_color;
//顶点法向量变量
attribute vec4 a_normal;
// uniform声明平行光颜色变量
uniform vec3 u_lightColor;
//平行光方向
uniform vec3 u_lightDirection;
//varying声明顶点颜色插值后变量
varying vec4 v_color;
void main() {
  //设置几何体轴旋转角度为30度，并把角度值转化为弧度值
  float radian = radians(30.0);
  //求解旋转角度余弦值
  float cos = cos(radian);
  //求解旋转角度正弦值
  float sin = sin(radian);
  //引用上面的计算数据，创建绕x轴旋转矩阵
  // 1      0       0    0
  // 0   cosα   sinα   0
  // 0  -sinα   cosα   0
  // 0      0        0   1
  mat4 mx = mat4(1,0,0,0,  0,cos,-sin,0,  0,sin,cos,0,  0,0,0,1);
  //引用上面的计算数据，创建绕y轴旋转矩阵
  // cosβ   0   sinβ    0
  //     0   1   0        0
  //-sinβ   0   cosβ    0
  //     0   0   0        1
  mat4 my = mat4(cos,0,-sin,0,  0,1,0,0,  sin,0,cos,0,  0,0,0,1);
  //两个旋转矩阵、顶点齐次坐标连乘
  gl_Position = mx*my*apos;

  // 顶点法向量进行矩阵变换，然后归一化
  vec3 normal = normalize((mx*my*a_normal).xyz);
  // 计算平行光方向向量和顶点法向量的点积
  // 颜色不存在负值，余弦值为负的物理意义就是光线无法照到的地方
  float dot = max(dot(u_lightDirection, normal), 0.0);
  // 计算反射后的颜色
  vec3 reflectedLight = u_lightColor * a_color.rgb * dot;
  //颜色插值计算
  v_color = vec4(reflectedLight, a_color.a);
}
```

片段着色器（`59-69`行）

```glsl
// 所有float类型数据的精度是lowp
precision lowp float;
// 接收顶点着色器中v_color数据
varying vec4 v_color;
void main() {
  // 插值后颜色数据赋值给对应的片元
  gl_FragColor = v_color;
}
```

光源数据（`94-100`行）

```js
/**
     * 给平行光传入颜色和方向数据，RGB(1,1,1),单位向量(x,y,z)
     **/
gl.uniform3f(u_lightColor, 1.0, 1.0, 1.0);
//保证向量(x,y,z)的长度为1，即单位向量
var x = 1/Math.sqrt(15), y = 2/Math.sqrt(15), z = 3/Math.sqrt(15);
gl.uniform3f(u_lightDirection,x,y,-z);

```

所用知识点

- `uniform`关键字：为了javascript可以通过相关的WebGL API给着色器变量传递数据，比如传递一个光源的位置数据、一个光源的方向数据、一个光源的颜色数据、一个用于顶点变换的模型矩阵、一个用于顶点变换的视图矩阵。

  javascript可以给顶点着色器的变量传递数据，也可以给片元着色器的变量传递数据，也就是说`uniform`关键字既可以在顶点着色器中使用，也可以在片元着色器中使用。只要注意`uniform`关键字声明变量需要在主函数`main`之前声明。

- 顶点法向量：为了表示物体表面各个位置的法线方向，可以给几何体的每个顶点定义一个方向向量。

  定义顶点法向量数据，这时候除了环境光以外，点光源也会参与光照计算，三角形整个表面比较明亮，同时两个三角形表面法线不同，即使光线方向相同，明暗自然不同，在分界位置有棱角感。

- 视线：javascript可以给顶点着色器的变量传递数据，也可以给片元着色器的变量传递数据，也就是说`uniform`关键字既可以在顶点着色器中使用，也可以在片元着色器中使用。只要注意`uniform`关键字声明变量需要在主函数`main`之前声明。



#### 点光源

顶点着色器（`10-57`行）

```glsl
// precision highp float;
//attribute声明vec4类型变量apos
attribute vec4 apos;
// attribute声明顶点颜色变量
attribute vec4 a_color;
//顶点法向量变量
attribute vec4 a_normal;
// uniform声明平行光颜色变量
uniform vec3 u_lightColor;
// uniform声明平行光颜色变量
uniform vec3 u_lightPosition;
//varying声明顶点颜色插值后变量
varying vec4 v_color;
void main() {
  //设置几何体轴旋转角度为30度，并把角度值转化为弧度值
  float radian = radians(30.0);
  //求解旋转角度余弦值
  float cos = cos(radian);
  //求解旋转角度正弦值
  float sin = sin(radian);
  //引用上面的计算数据，创建绕x轴旋转矩阵
  // 1      0       0    0
  // 0   cosα   sinα   0
  // 0  -sinα   cosα   0
  // 0      0        0   1
  mat4 mx = mat4(1,0,0,0,  0,cos,-sin,0,  0,sin,cos,0,  0,0,0,1);
  //引用上面的计算数据，创建绕y轴旋转矩阵
  // cosβ   0   sinβ    0
  //     0   1   0        0
  //-sinβ   0   cosβ    0
  //     0   0   0        1
  mat4 my = mat4(cos,0,-sin,0,  0,1,0,0,  sin,0,cos,0,  0,0,0,1);
  //两个旋转矩阵、顶点齐次坐标连乘
  gl_Position = mx*my*apos;
  // 顶点法向量进行矩阵变换，然后归一化
  vec3 normal = normalize((mx*my*a_normal).xyz);
  // 计算点光源照射顶点的方向并归一化
  vec3 lightDirection = normalize(vec3(gl_Position) - u_lightPosition);
  // 计算平行光方向向量和顶点法向量的点积
  float dot = max(dot(lightDirection, normal), 0.0);
  // 计算反射后的颜色
  vec3 reflectedLight = u_lightColor * a_color.rgb * dot;
  //颜色插值计算
  v_color = vec4(reflectedLight, a_color.a);
}
```

片段着色器（`59-69`行）

```glsl
// 所有float类型数据的精度是lowp
precision lowp float;
// 接收顶点着色器中v_color数据
varying vec4 v_color;
void main() {
  // 插值后颜色数据赋值给对应的片元
  gl_FragColor = v_color;
}
```

点光源数据（`94-98`行）

```js
/**
     * 传入点光源颜色数据、位置数据
     **/
gl.uniform3f(u_lightColor, 1.0, 1.0, 1.0);
gl.uniform3f(u_lightPosition, 2.0, 3.0, 4.0);
```

所用知识点

- 点光源与顶点方向的计算：点光源的光线是发散的，点光源与每一顶点连线的方向都需要单独计算。

  `vec3(gl_Position) - u_lightPosition`用来计算光线方向，然后利用内置函数 `normalize()`归一化向量数据，`vec3(gl_Position`)和`gl_Position.xyz`的写法是等效的，都是为了提取`vec4`类型顶点数据前三个分量，返回的数据类型是`vec3`，比如`vec2(vec4)`就是提取`vec4`的前两个分量。



### 纹理

#### 纹理映射

顶点着色器（`13-24`行）

```glsl
attribute vec4 a_Position;//顶点位置坐标
attribute vec2 a_TexCoord;//纹理坐标
varying vec2 v_TexCoord;//插值后纹理坐标
void main() {
  //顶点坐标apos赋值给内置变量gl_Position
  gl_Position = a_Position;
  //纹理坐标插值计算
  v_TexCoord = a_TexCoord;
}
```

片段着色器（`26-38`行）

```glsl
//所有float类型数据的精度是highp
precision highp float;
// 接收插值后的纹理坐标
varying vec2 v_TexCoord;
// 纹理图片像素数据
uniform sampler2D u_Sampler;
void main() {
  // 采集纹素，逐片元赋值像素值
  gl_FragColor = texture2D(u_Sampler,v_TexCoord);
}
```

`UV`纹理坐标数据（`61-79`行）

```js
/**
     * 四个顶点坐标数据data，z轴为零
     * 定义纹理贴图在WebGL坐标系中位置
     **/
var data=new Float32Array([
  -0.5, 0.5,//左上角——v0
  -0.5,-0.5,//左下角——v1
  0.5, 0.5,//右上角——v2
  0.5, -0.5 //右下角——v3
]);
/**
     * 创建UV纹理坐标数据textureData
     **/
var textureData = new Float32Array([
  0,1,//左上角——uv0
  0,0,//左下角——uv1
  1,1,//右上角——uv2
  1,0 //右下角——uv3
]);
```

所用知识点

- 取样器`sampler2D`：在处理图片纹理编写片元着色器的时候，会通过`sampler2D`关键字声明一个纹理相关的变量，`sampler2D`和`vec2`、`float`表示一种数据类型，只不过`sampler2D`关键字声明的变量表示一种取样器类型变量，简单点说就是该变量对应纹理图片的像素数据。
- 内置函数`texture2D`：可以从纹理图像提取像素值，赋值给内置变量`gl_FragColor`。
- `UV`坐标映射：纹理坐标系统记做`UV`坐标，是一个二维坐标系，而WebGL坐标系统为三维坐标系。图片称为纹理图像，图片上的一个像素称为纹素，一个纹素就是一个RGB或RGBA值。 把整个图片看成一个平面区域，用一个二维UV坐标系可以描述每一个纹素的位置。



#### 彩色图转灰度图

顶点着色器（`13-24`行）

```glsl
attribute vec4 a_Position;//顶点位置坐标
attribute vec2 a_TexCoord;//纹理坐标
varying vec2 v_TexCoord;//插值后纹理坐标
void main() {
  //顶点坐标apos赋值给内置变量gl_Position
  gl_Position = a_Position;
  //纹理坐标插值计算
  v_TexCoord = a_TexCoord;
}
```

片段着色器（`26-42`行）

```glsl
//所有float类型数据的精度是highp
precision highp float;
// 接收插值后的纹理坐标
varying vec2 v_TexCoord;
// 纹理图片像素数据
uniform sampler2D u_Sampler;
void main() {
  //采集纹素
  vec4 texture = texture2D(u_Sampler,v_TexCoord);
  //计算RGB三个分量光能量之和，也就是亮度
  float luminance = 0.299*texture.r+0.587*texture.g+0.114*texture.b;
  //逐片元赋值，RGB相同均为亮度值，用黑白两色表达图片的明暗变化
  gl_FragColor = vec4(luminance,luminance,luminance,1);
}
```

`UV`纹理坐标数据（`66-83`行）

```js
/**
     * 四个顶点坐标数据data，z轴为零
     * 定义纹理贴图在WebGL坐标系中位置
     **/
var data=new Float32Array([
  -0.5, 0.5,//左上角——v0
  -0.5,-0.5,//左下角——v1
  0.5,  0.5,//右上角——v2
  0.5, -0.5 //右下角——v3
]);
/**
     * 创建UV纹理坐标数据textureData
     **/
var textureData = new Float32Array([
  0,1,//左上角——uv0
  0,0,//左下角——uv1
  1,1,//右上角——uv2
  1,0 //右下角——uv3
]);
```

所用知识点

- 亮度：`亮度 = 0.299 x R + 0.587 x G + 0.114 x B`



## 进阶

### 造型

#### `smoothstep`函数

顶点着色器

```glsl
// 使用ShaderMaterial类，顶点位置变量position无需声明，顶点着色器可以直接调用
// attribute vec3 position;
void main(){
  // 逐顶点处理：顶点位置数据赋值给内置变量gl_Position
  gl_Position = vec4( position, 1.0 );
}
```

片段着色器

```glsl
precision mediump float;
//画布尺寸（宽、高）
uniform vec2 u_resolution;

//用smoothstep计算插值绘制曲线
float plot(vec2 st, float pct){
  return  smoothstep( pct-0.02, pct, st.y) -
    smoothstep( pct, pct+0.02, st.y);
}

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution;

  float y = smoothstep(0.2,0.5,st.x) - smoothstep(0.5,0.8,st.x);

  vec3 color = vec3(y);
	
  //绘制
  float pct = plot(st,y);
  color = (1.0-pct)*color+pct*vec3(0.0,1.0,0.0);

  gl_FragColor = vec4(color,1.0);
}
```

所用知识点

- `smoothstep()`：当给定一个范围的上下限和一个数值，这个函数会在已有的范围内给出插值。前两个参数规定转换的开始和结束点，第三个是给出一个值用来插值。

#### `step`函数

片段着色器

```glsl
uniform vec2 u_resolution;
uniform float u_time;

float plot(vec2 st, float pct){
  return  smoothstep( pct-0.02, pct, st.y) -
          smoothstep( pct, pct+0.02, st.y);
}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution;

    //除非值超过 0.5，否则 Step 将返回 0.0，在这种情况下，它将返回 1.0
    float y = step(0.5,st.x);

    vec3 color = vec3(y);

    float pct = plot(st,y);
    color = (1.0-pct)*color+pct*vec3(0.0,1.0,0.0);

    gl_FragColor = vec4(color,1.0);
}
```

所用知识点

- `step`函数：`step`插值函数需要输入两个参数。第一个是极限或阈值，第二个是我们想要检测或通过的值。对任何小于阈值的值，返回 `0.0`，大于阈值，则返回 `1.0`。

#### 空间距离

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

void main(){
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  st.x *= u_resolution.x/u_resolution.y;
  vec3 color = vec3(0.0);
  float d = 0.0;

  // 将空间重新映射为 -1 为 1
  st = st *2.-1.;

  // 制作距离场
  d = length( abs(st)-.3 );
  // d = length( min(abs(st)-.3,0.) );
  // d = length( max(abs(st)-.3,0.) );

  // 可视化距离场
  gl_FragColor = vec4(vec3(fract(d*10.0)),1.0);

  // 用距离场绘图
  // gl_FragColor = vec4(vec3( step(.3,d) ),1.0);
  // gl_FragColor = vec4(vec3( step(.3,d) * step(d,.4)),1.0);
  // gl_FragColor = vec4(vec3( smoothstep(.3,.4,d)* smoothstep(.6,.5,d)) ,1.0);
}
```

所用知识点

- 距离场：假设圆形是一个圆锥的投影，则不断从中间任何部位截圆锥，就会得到或大或小的圆纹面。假设站在圆锥的顶端，那么这些渐变就像是圆锥色等高线图。这种通过“空间距离”来解释图形的技巧，称为“距离场”。



#### 极坐标

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

void main(){
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  vec3 color = vec3(0.0);

  vec2 pos = vec2(0.5)-st;

  float r = length(pos)*2.0;
  float a = atan(pos.y,pos.x);

  float f = cos(a*3.);
  // f = abs(cos(a*3.));
  // f = abs(cos(a*2.5))*.5+.3;
  // f = abs(cos(a*12.)*sin(a*3.))*.8+.1;
  // f = smoothstep(-.5,1., cos(a*10.))*0.2+0.5;

  color = vec3( 1.-smoothstep(f,f+0.02,r) );

  gl_FragColor = vec4(color, 1.0);
}
```

所用知识点

- 坐标系转换：

  极坐标是以半径和角度定义的。

  将笛卡尔坐标系中的`x`、`y`通过以下方程映射为极坐标

  ```glsl
  vec2 pos = vec2(0.5)-st;
  float r = length(pos)*2.0;
  float a = atan(pos.y,pos.x);
  ```

  

#### 平移

片段着色器

```glsl
uniform vec2 u_resolution;
uniform float u_time;

float box(in vec2 _st, in vec2 _size){
  _size = vec2(0.5) - _size*0.5;
  vec2 uv = smoothstep(_size,
                       _size+vec2(0.001),
                       _st);
  uv *= smoothstep(_size,
                   _size+vec2(0.001),
                   vec2(1.0)-_st);
  return uv.x*uv.y;
}

float cross(in vec2 _st, float _size){
  return  box(_st, vec2(_size,_size/4.)) +
    box(_st, vec2(_size/4.,_size));
}

void main(){
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  vec3 color = vec3(0.0);

  // 为了移动十字架，我们移动空间
  vec2 translate = vec2(cos(u_time),sin(u_time));
  st += translate*0.35;

  // 在背景上显示空间的坐标
  color = vec3(st.x,st.y,0.0);

  // 在背景上添加形状
  color += vec3(cross(st,0.25));

  gl_FragColor = vec4(color,1.0);
}
```

所用知识点

- `u_time`：获取时间。

- 平移：这里的平移是通过移动整个坐标系。

  

#### 缩放

片段着色器

```glsl
#define PI 3.14159265359
uniform vec2 u_resolution;
uniform float u_time;

mat2 scale(vec2 _scale){
  return mat2(_scale.x,0.0,
              0.0,_scale.y);
}

float box(in vec2 _st, in vec2 _size){
  _size = vec2(0.5) - _size*0.5;
  vec2 uv = smoothstep(_size,
                       _size+vec2(0.001),
                       _st);
  uv *= smoothstep(_size,
                   _size+vec2(0.001),
                   vec2(1.0)-_st);
  return uv.x*uv.y;
}

float cross(in vec2 _st, float _size){
  return  box(_st, vec2(_size,_size/4.)) +
    box(_st, vec2(_size/4.,_size));
}

void main(){
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  vec3 color = vec3(0.0);

  st -= vec2(0.5);
  st = scale( vec2(sin(u_time)+1.0) ) * st;
  st += vec2(0.5);

  // 在背景上显示空间的坐标
  color = vec3(st.x,st.y,0.0);

  // 在上面添加形状
  color += vec3(cross(st,0.2));

  gl_FragColor = vec4(color,1.0);
}
```

所用知识点

- 缩放：通过改变空间坐标进行缩放

- `scale`：

  ```glsl
  mat2 scale(vec2 _scale){
      return mat2(_scale.x,0.0,
                  0.0,_scale.y);
  }
  ```

  

#### 旋转

片段着色器

```glsl
#define PI 3.14159265359
uniform vec2 u_resolution;
uniform float u_time;

mat2 rotate2d(float _angle){
  return mat2(cos(_angle),-sin(_angle),
              sin(_angle),cos(_angle));
}

float box(in vec2 _st, in vec2 _size){
  _size = vec2(0.5) - _size*0.5;
  vec2 uv = smoothstep(_size,
                       _size+vec2(0.001),
                       _st);
  uv *= smoothstep(_size,
                   _size+vec2(0.001),
                   vec2(1.0)-_st);
  return uv.x*uv.y;
}

float cross(in vec2 _st, float _size){
  return  box(_st, vec2(_size,_size/4.)) +
    box(_st, vec2(_size/4.,_size));
}

void main(){
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  vec3 color = vec3(0.0);

  // 将空间从中心移到 vec2(0.0)
  st -= vec2(0.5);
  // 旋转空间
  st = rotate2d( sin(u_time)*PI ) * st;
  // 把它移回原来的地方
  st += vec2(0.5);

  // 在背景上显示空间的坐标
  color = vec3(st.x,st.y,0.0);

  // 在上面添加形状
  color += vec3(cross(st,0.4));

  gl_FragColor = vec4(color,1.0);
}
```

所用知识点

- 旋转：通过旋转坐标空间来达到旋转的目的

- `rotate2d`：

  ```glsl
  mat2 rotate2d(float _angle){
      return mat2(cos(_angle),-sin(_angle),
                  sin(_angle),cos(_angle));
  }
  ```



#### 小结

片段着色器

```glsl
#define PI 3.14159265359
#define TWO_PI 6.28318530718

uniform vec2 u_resolution;
uniform float u_time;

float shape(vec2 st, float N){
    st = st*2.-1.;
    float a = atan(st.x,st.y)+PI;
    float r = TWO_PI/N;
    return abs(cos(floor(.5+a/r)*r-a)*length(st));
}

float box(vec2 st, vec2 size){
    return shape(st*size,4.);
}

float rect(vec2 _st, vec2 _size){
    _size = vec2(0.5)-_size*0.5;
    vec2 uv = smoothstep(_size,_size+vec2(1e-4),_st);
    uv *= smoothstep(_size,_size+vec2(1e-4),vec2(1.0)-_st);
    return uv.x*uv.y;
}

float hex(vec2 st, float a, float b, float c, float d, float e, float f){
    st = st*vec2(2.,6.);

    vec2 fpos = fract(st);
    vec2 ipos = floor(st);

    if (ipos.x == 1.0) fpos.x = 1.-fpos.x;
    if (ipos.y < 1.0){
        return mix(box(fpos, vec2(0.84,1.)),box(fpos-vec2(0.03,0.),vec2(1.)),a);
    } else if (ipos.y < 2.0){
        return mix(box(fpos, vec2(0.84,1.)),box(fpos-vec2(0.03,0.),vec2(1.)),b);
    } else if (ipos.y < 3.0){
        return mix(box(fpos, vec2(0.84,1.)),box(fpos-vec2(0.03,0.),vec2(1.)),c);
    } else if (ipos.y < 4.0){
        return mix(box(fpos, vec2(0.84,1.)),box(fpos-vec2(0.03,0.),vec2(1.)),d);
    } else if (ipos.y < 5.0){
        return mix(box(fpos, vec2(0.84,1.)),box(fpos-vec2(0.03,0.),vec2(1.)),e);
    } else if (ipos.y < 6.0){
        return mix(box(fpos, vec2(0.84,1.)),box(fpos-vec2(0.03,0.),vec2(1.)),f);
    }
    return 0.0;
}

float hex(vec2 st, float N){
    float b[6];
    float remain = floor(mod(N,64.));
    for(int i = 0; i < 6; i++){
        b[i] = 0.0;
        b[i] = step(1.0,mod(remain,2.));
        remain = ceil(remain/2.);
    }
    return hex(st,b[0],b[1],b[2],b[3],b[4],b[5]);
}

void main(){
    vec2 st = gl_FragCoord.xy/u_resolution.xy;
    st.y *= u_resolution.y/u_resolution.x;

    st *= 10.0;
    vec2 fpos = fract(st);
    vec2 ipos = floor(st);

    float t = u_time*5.0;
    float df = 1.0;
    df = hex(fpos,ipos.x+ipos.y+t)+(1.0-rect(fpos,vec2(0.7)));

    gl_FragColor = vec4(mix(vec3(0.),vec3(1.),step(0.7,df)),1.0);
}
```



### 随机

#### 2D随机

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float random (vec2 st) {
  return fract(sin(dot(st.xy,
                       vec2(12.9898,78.233)))*
               u_time);
}

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution.xy;

  float rnd = random( st );

  gl_FragColor = vec4(vec3(rnd),1.0);
}
```

所用知识点

- 伪随机混沌的生成：利用`fract`提取了`sin`函数其波形的分数部分。可以用这种效果通过把正弦函数打散成小片段来得到一些伪随机数。

  这里将它应用到二维的时候，需要将一个二维向量转化为一维浮点数，这里就会用到点乘方法。



#### 使用混沌

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float random (vec2 st) {
  return fract(sin(dot(st.xy,
                       vec2(12.9898,78.233)))*
               43758.5453123);
}

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution.xy;

  st *= 10.0; // 将坐标系缩放 10
  vec2 ipos = floor(st);  // 获取整数坐标
  vec2 fpos = fract(st);  // 获取分数坐标

  // 根据整数坐标分配一个随机值
  vec3 color = vec3(random( ipos ));

  // 取消注释以查看细分网格
  // color = vec3(fpos,0.0);

  gl_FragColor = vec4(color,1.0);
}
```

所用知识点

- `floor()`函数：找到小于或等于参数最接近的整数。

  这里使用这个函数产生一个单元整数阵列，将坐标系统的整数和小数部分分离，从而有更加明显的分割效果。



#### 迷宫特效

片段着色器

```glsl
#define PI 3.14159265358979323846
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float random (in vec2 _st) {
  return fract(sin(dot(_st.xy,
                       vec2(12.9898,78.233)))*
               43758.5453123);
}

// 通过该随机值随便画出某一方向的对角线
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

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  st *= 10.0;
  st = (st-vec2(5.0))*(abs(sin(u_time*0.2))*5.);
  st.x += u_time*3.0;

  vec2 ipos = floor(st);  // 整数
  vec2 fpos = fract(st);  // 分数

  vec2 tile = truchetPattern(fpos, random( ipos ));

  float color = 0.0;

  // 迷宫
  color = smoothstep(tile.x-0.3,tile.x,tile.y)-
    smoothstep(tile.x,tile.x+0.3,tile.y);

  // 圆形迷宫
  color = (step(length(tile),0.6) -
           step(length(tile),0.4) ) +
    (step(length(tile-vec2(1.)),0.6) -
     step(length(tile-vec2(1.)),0.4) );

  gl_FragColor = vec4(vec3(color),1.0);
}
```



#### 小结

片段着色器

```glsl
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



### 噪声

#### 2D噪声

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

// 二维随机
float random (in vec2 st) {
  return fract(sin(dot(st.xy,
                       vec2(12.9898,78.233)))
               * 43758.5453123);
}

float noise (in vec2 st) {
  vec2 i = floor(st);
  vec2 f = fract(st);

  // 瓷砖二维中的四个角
  float a = random(i);
  float b = random(i + vec2(1.0, 0.0));
  float c = random(i + vec2(0.0, 1.0));
  float d = random(i + vec2(1.0, 1.0));

  // 平滑插值
  // 三次 Hermine 曲线。 与 SmoothStep() 相同
  vec2 u = f*f*(3.0-2.0*f);
  // u = smoothstep(0.,1.,f);

  // 混合4个角落百分比
  return mix(a, b, u.x) +
    (c - a)* u.y * (1.0 - u.x) +
    (d - b) * u.x * u.y;
}

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution.xy;

  // 缩放坐标系以在行动中看到一些噪音
  vec2 pos = vec2(st*5.0);

  // 使用噪音功能
  float n = noise(pos);

  gl_FragColor = vec4(vec3(n), 1.0);
}
```

所用知识点

- 噪声与随机的区别：之前的随机实际上是一种“伪随机”，而真实世界的不可预测性往往比“伪随机”更加丰富而复杂，噪声就是为了表达这种更加复杂的随机。
- 二维噪声的实现：先通过随机得到一个平面上方形四个角的数据，然后在四个角上做插值。



#### 单纯形噪声

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

vec2 skew (vec2 st) {
  vec2 r = vec2(0.0);
  r.x = 1.1547*st.x;
  r.y = st.y+0.5*r.x;
  return r;
}

vec3 simplexGrid (vec2 st) {
  vec3 xyz = vec3(0.0);

  vec2 p = fract(skew(st));
  if (p.x > p.y) {
    xyz.xy = 1.0-vec2(p.x,p.y-p.x);
    xyz.z = p.y;
  } else {
    xyz.yz = 1.0-vec2(p.x-p.y,p.y);
    xyz.x = p.x;
  }

  return fract(xyz);
}

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  vec3 color = vec3(0.0);

  // 缩放空间以查看网格
  st *= 10.;

  // 显示二维网格
  color.rg = fract(st);

  // 倾斜二维网格
  color.rg = fract(skew(st));

  // 将网格细分为等边三角形
  color = simplexGrid(st);

  gl_FragColor = vec4(color,1.0);
}
```

所用知识点

- 单纯形：将正方形网格替换成单纯形等边三角形网格。

  从原先的`N`维需要插入`2^N`个点变成了`N`维形状就只需要`N+1`个点。

- 单纯形网格的制作：先把常规的四角网格分成两个等腰三角形，再把三角形歪斜成等边三角形。

- 与之前的噪声的优化：

  - 有更低的计算复杂度和更少的乘法计算
  - 可以用更少的计算量达到更高的维度
  - 制造出`noise`没有明显的人工痕迹
  - 有着定义德很精巧的连续的梯度，可以大大降低计算成本
  - 特别易于硬件实现

#### 流体特效

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

vec3 mod289(vec3 x) { return x - floor(x * (1.0 / 289.000)) * 289.0; }
vec2 mod289(vec2 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }
vec3 permute(vec3 x) { return mod289(((x*34.0)+1.0)*x); }

float snoise(vec2 v) {
  const vec4 C = vec4(0.211,  // (3.0-sqrt(3.0))/6.0
                      0.366025403784439,  // 0.5*(sqrt(3.0)-1.0)
                      -0.577350269189626,  // -1.0 + 2.0 * C.x
                      0.024390243902439); // 1.0 / 41.0
  vec2 i  = floor(v + dot(v, C.yy) );
  vec2 x0 = v -   i + dot(i, C.xx);
  vec2 i1;
  i1 = (x0.x > x0.y) ? vec2(1.0, 0.0) : vec2(0.0, 1.0);
  vec4 x12 = x0.xyxy + C.xxzz;
  x12.xy -= i1;
  i = mod289(i); // 避免排列中的截断效应
  vec3 p = permute( permute( i.y + vec3(0.0, i1.y, 1.0 ))
                   + i.x + vec3(0.0, i1.x, 1.0 ));

  vec3 m = max(0.5 - vec3(dot(x0,x0), dot(x12.xy,x12.xy), dot(x12.zw,x12.zw)), 0.0);
  m = m*m ;
  m = m*m ;
  vec3 x = 2.0 * fract(p * C.www) - 1.0;
  vec3 h = abs(x) - 0.5;
  vec3 ox = floor(x + 0.5);
  vec3 a0 = x - ox;
  m *= 1.79284291400159 - 0.85373472095314 * ( a0*a0 + h*h );
  vec3 g;
  g.x  = a0.x  * x0.x  + h.x  * x0.y;
  g.yz = a0.yz * x12.xz + h.yz * x12.yw;
  return 130.0 * dot(m, g);
}

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  st.x *= u_resolution.x/u_resolution.y;
  vec3 color = vec3(0.0);
  vec2 pos = vec2(st*3.);

  float DF = 0.0;

  // 添加随机位置
  float a = 0.0;
  vec2 vel = vec2(u_time*.1);
  DF += snoise(pos+vel)*.25+.25;

  // 添加随机位置
  a = snoise(pos*vec2(cos(u_time*0.15),sin(u_time*0.1))*0.1)*3.1415;
  vel = vec2(cos(a),sin(a));
  DF += snoise(pos+vel)*.25+.25;

  color = vec3( smoothstep(.7,.75,fract(DF)) );

  gl_FragColor = vec4(1.0-color,1.0);
}
```



#### 网格噪声

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  st.x *= u_resolution.x/u_resolution.y;

  vec3 color = vec3(.0);

  // 单元格位置
  vec2 point[5];
  point[0] = vec2(0.83,0.75);
  point[1] = vec2(0.60,0.07);
  point[2] = vec2(0.28,0.64);
  point[3] =  vec2(0.31,0.26);
  point[4] = u_mouse/u_resolution;

  float m_dist = 1.;  // 最小距离

  // 遍历点位置
  for (int i = 0; i < 5; i++) {
    float dist = distance(st, point[i]);

    // 保持更近的距离
    m_dist = min(m_dist, dist);
  }

  // 绘制最小距离（距离场）
  color += m_dist;

  gl_FragColor = vec4(color,1.0);
}
```

所用知识点

- 网格噪声：网格噪声基于距离场，这里的距离是指到一个特征点集最近的点的距离。

  比如要写4个特征点的距离场，对每一个像素，计算它到最近的特征点的距离。也就是说需要遍历所有4个特征点，计算他们到当前像素点的距离，并把最近的那个距离存下来。



#### 平铺和迭代

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

vec2 random2( vec2 p ) {
  return fract(sin(vec2(dot(p,vec2(127.1,311.7)),dot(p,vec2(269.5,183.3))))*43758.5453);
}

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  st.x *= u_resolution.x/u_resolution.y;
  vec3 color = vec3(.0);

  // 缩放
  st *= 3.;

  // 平铺空间
  vec2 i_st = floor(st);
  vec2 f_st = fract(st);

  float m_dist = 1.;  // 最小距离

  for (int y= -1; y <= 1; y++) {
    for (int x= -1; x <= 1; x++) {
      // 网格中的邻居位置
      vec2 neighbor = vec2(float(x),float(y));

      // 网格中当前+相邻位置的随机位置
      vec2 point = random2(i_st + neighbor);

      // 动画点
      point = 0.5 + 0.5*sin(u_time + 6.2831*point);

      // 像素和点之间的向量
      vec2 diff = neighbor + point - f_st;

      // 到点的距离
      float dist = length(diff);

      // 保持更近的距离
      m_dist = min(m_dist, dist);
    }
  }

  // 绘制最小距离（距离场）
  color += m_dist;

  // 绘制单元格中心
  color += 1.-step(.02, m_dist);

  // 绘制网格
  color.r += step(.98, f_st.x) + step(.98, f_st.y);

  gl_FragColor = vec4(color,1.0);
}
```

所用知识点

- 从循环转向GPU：已经知道每个像素点是在自己的线程中运行，我们可以把空间分割成网格，每个网格对应一个特征点。

  

### 分形布朗运动

#### 2D分形布朗运动

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float random (in vec2 st) {
  return fract(sin(dot(st.xy,
                       vec2(12.9898,78.233)))*
               43758.5453123);
}

float noise (in vec2 st) {
  vec2 i = floor(st);
  vec2 f = fract(st);

  // 瓷砖二维中的四个角
  float a = random(i);
  float b = random(i + vec2(1.0, 0.0));
  float c = random(i + vec2(0.0, 1.0));
  float d = random(i + vec2(1.0, 1.0));

  vec2 u = f * f * (3.0 - 2.0 * f);

  return mix(a, b, u.x) +
    (c - a)* u.y * (1.0 - u.x) +
    (d - b) * u.x * u.y;
}

#define OCTAVES 6
  float fbm (in vec2 st) {
    // 初始值
    float value = 0.0;
    float amplitude = .5;
    float frequency = 0.;
    //
    // 八度循环
    for (int i = 0; i < OCTAVES; i++) {
      value += amplitude * noise(st);
      st *= 2.;
      amplitude *= .5;
    }
    return value;
  }

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution.xy;
  st.x *= u_resolution.x/u_resolution.y;

  vec3 color = vec3(0.0);
  color += fbm(st*3.0);

  gl_FragColor = vec4(color,1.0);
}
```

所用知识点

- 分形布朗运动：噪声可以看做是声波，波都有频率（`frequency`）和振幅（`amplitude`），除此之外，波的另一个特性是叠加性，通过在循环（octaves）中叠加噪声，并以一定的倍数（lacunarity）连续升高频率，同时以一定的比例降低噪声的振幅，最终的结果会有更好的细节。这种技术叫做分形布朗运动。



#### 域失真（翘曲）

片段着色器

```glsl
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float random (in vec2 _st) {
  return fract(sin(dot(_st.xy,
                       vec2(12.9898,78.233)))*
               43758.5453123);
}

float noise (in vec2 _st) {
  vec2 i = floor(_st);
  vec2 f = fract(_st);

  // 瓷砖二维中的四个角
  float a = random(i);
  float b = random(i + vec2(1.0, 0.0));
  float c = random(i + vec2(0.0, 1.0));
  float d = random(i + vec2(1.0, 1.0));

  vec2 u = f * f * (3.0 - 2.0 * f);

  return mix(a, b, u.x) +
    (c - a)* u.y * (1.0 - u.x) +
    (d - b) * u.x * u.y;
}

#define NUM_OCTAVES 5

  float fbm ( in vec2 _st) {
    float v = 0.0;
    float a = 0.5;
    vec2 shift = vec2(100.0);
    // 旋转以减少轴向偏差
    mat2 rot = mat2(cos(0.5), sin(0.5),
                    -sin(0.5), cos(0.50));
    for (int i = 0; i < NUM_OCTAVES; ++i) {
      v += a * noise(_st);
      _st = rot * _st * 2.0 + shift;
      a *= 0.5;
    }
    return v;
  }

void main() {
  vec2 st = gl_FragCoord.xy/u_resolution.xy*3.;
  st += st * abs(sin(u_time*0.1)*3.0);
  vec3 color = vec3(0.0);

  vec2 q = vec2(0.);
  q.x = fbm( st + 0.00*u_time);
  q.y = fbm( st + vec2(1.0));

  vec2 r = vec2(0.);
  r.x = fbm( st + 1.0*q + vec2(1.7,9.2)+ 0.15*u_time );
  r.y = fbm( st + 1.0*q + vec2(8.3,2.8)+ 0.126*u_time);

  float f = fbm(st+r);

  color = mix(vec3(0.101961,0.619608,0.666667),
              vec3(0.666667,0.666667,0.498039),
              clamp((f*f)*4.0,0.0,1.0));

  color = mix(color,
              vec3(0,0,0.164706),
              clamp(length(q),0.0,1.0));

  color = mix(color,
              vec3(0.666667,1,1),
              clamp(length(r.x),0.0,1.0));

  gl_FragColor = vec4((f*f*f+.6*f*f+.5*f)*color,1.);
}
```



## 综合

### 例1

顶点着色器

```glsl
uniform vec2 uvScale;
varying vec2 vUv;

void main()
{

  vUv = uvScale * uv;
  vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
  gl_Position = projectionMatrix * mvPosition;

}
```

片段着色器

```glsl
uniform float time;

uniform float fogDensity;
uniform vec3 fogColor;

uniform sampler2D texture1;
uniform sampler2D texture2;

varying vec2 vUv;

void main( void ) {

  vec2 position = - 1.0 + 2.0 * vUv;

  vec4 noise = texture2D( texture1, vUv );
  vec2 T1 = vUv + vec2( 1.5, - 1.5 ) * time * 0.02;
  vec2 T2 = vUv + vec2( - 0.5, 2.0 ) * time * 0.01;

  T1.x += noise.x * 2.0;
  T1.y += noise.y * 2.0;
  T2.x -= noise.y * 0.2;
  T2.y += noise.z * 0.2;

  float p = texture2D( texture1, T1 * 2.0 ).a;

  vec4 color = texture2D( texture2, T2 * 2.0 );
  vec4 temp = color * ( vec4( p, p, p, p ) * 2.0 ) + ( color * color - 0.1 );

  if( temp.r > 1.0 ) { temp.bg += clamp( temp.r - 2.0, 0.0, 100.0 ); }
  if( temp.g > 1.0 ) { temp.rb += temp.g - 1.0; }
  if( temp.b > 1.0 ) { temp.rg += temp.b - 1.0; }

  gl_FragColor = temp;

  float depth = gl_FragCoord.z / gl_FragCoord.w;
  const float LOG2 = 1.442695;
  float fogFactor = exp2( - fogDensity * fogDensity * depth * depth * LOG2 );
  fogFactor = 1.0 - clamp( fogFactor, 0.0, 1.0 );

  gl_FragColor = mix( gl_FragColor, vec4( fogColor, gl_FragColor.w ), fogFactor );

}
```



### 例2

顶点着色器

```glsl
varying vec2 vUv;

void main()	{

  vUv = uv;

  gl_Position = vec4( position, 1.0 );

}
```

片段着色器

```glsl
varying vec2 vUv;

uniform float time;

void main()	{

  vec2 p = - 1.0 + 2.0 * vUv;
  float a = time * 40.0;
  float d, e, f, g = 1.0 / 40.0 ,h ,i ,r ,q;

  e = 400.0 * ( p.x * 0.5 + 0.5 );
  f = 400.0 * ( p.y * 0.5 + 0.5 );
  i = 200.0 + sin( e * g + a / 150.0 ) * 20.0;
  d = 200.0 + cos( f * g / 2.0 ) * 18.0 + cos( e * g ) * 7.0;
  r = sqrt( pow( abs( i - e ), 2.0 ) + pow( abs( d - f ), 2.0 ) );
  q = f / r;
  e = ( r * cos( q ) ) - a / 2.0;
  f = ( r * sin( q ) ) - a / 2.0;
  d = sin( e * g ) * 176.0 + sin( e * g ) * 164.0 + r;
  h = ( ( f + d ) + a / 2.0 ) * g;
  i = cos( h + r * p.x / 1.3 ) * ( e + e + a ) + cos( q * g * 6.0 ) * ( r + h / 3.0 );
  h = sin( f * g ) * 144.0 - sin( e * g ) * 212.0 * p.x;
  h = ( h + ( f - e ) * q + sin( r - ( a + h ) / 7.0 ) * 10.0 + i / 4.0 ) * g;
  i += cos( h * 2.3 * sin( a / 350.0 - q ) ) * 184.0 * sin( q - ( r * 4.3 + a / 12.0 ) * g ) + tan( r * g + h ) * 184.0 * cos( r * g + h );
  i = mod( i / 5.6, 256.0 ) / 64.0;
  if ( i < 0.0 ) i += 4.0;
  if ( i >= 2.0 ) i = 4.0 - i;
  d = r / 350.0;
  d += sin( d * d * 8.0 ) * 0.52;
  f = ( sin( a * g ) + 1.0 ) / 2.0;
  gl_FragColor = vec4( vec3( f * i / 1.6, i / 2.0 + d / 13.0, i ) * d * p.x + vec3( i / 1.3 + d / 8.0, i / 2.0 + d / 18.0, i ) * d * ( 1.0 - p.x ), 1.0 );

}
```



### 例3

顶点着色器

```glsl
varying vec2 vUv;

void main()
{
  vUv = uv;
  vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
  gl_Position = projectionMatrix * mvPosition;
}
```

片段着色器

```glsl

uniform float time;

varying vec2 vUv;

void main(void) {

  vec2 p = - 1.0 + 2.0 * vUv;
  float a = time * 40.0;
  float d, e, f, g = 1.0 / 40.0 ,h ,i ,r ,q;

  e = 400.0 * ( p.x * 0.5 + 0.5 );
  f = 400.0 * ( p.y * 0.5 + 0.5 );
  i = 200.0 + sin( e * g + a / 150.0 ) * 20.0;
  d = 200.0 + cos( f * g / 2.0 ) * 18.0 + cos( e * g ) * 7.0;
  r = sqrt( pow( abs( i - e ), 2.0 ) + pow( abs( d - f ), 2.0 ) );
  q = f / r;
  e = ( r * cos( q ) ) - a / 2.0;
  f = ( r * sin( q ) ) - a / 2.0;
  d = sin( e * g ) * 176.0 + sin( e * g ) * 164.0 + r;
  h = ( ( f + d ) + a / 2.0 ) * g;
  i = cos( h + r * p.x / 1.3 ) * ( e + e + a ) + cos( q * g * 6.0 ) * ( r + h / 3.0 );
  h = sin( f * g ) * 144.0 - sin( e * g ) * 212.0 * p.x;
  h = ( h + ( f - e ) * q + sin( r - ( a + h ) / 7.0 ) * 10.0 + i / 4.0 ) * g;
  i += cos( h * 2.3 * sin( a / 350.0 - q ) ) * 184.0 * sin( q - ( r * 4.3 + a / 12.0 ) * g ) + tan( r * g + h ) * 184.0 * cos( r * g + h );
  i = mod( i / 5.6, 256.0 ) / 64.0;
  if ( i < 0.0 ) i += 4.0;
  if ( i >= 2.0 ) i = 4.0 - i;
  d = r / 350.0;
  d += sin( d * d * 8.0 ) * 0.52;
  f = ( sin( a * g ) + 1.0 ) / 2.0;
  gl_FragColor = vec4( vec3( f * i / 1.6, i / 2.0 + d / 13.0, i ) * d * p.x + vec3( i / 1.3 + d / 8.0, i / 2.0 + d / 18.0, i ) * d * ( 1.0 - p.x ), 1.0 );

}
```

```glsl
uniform float time;

uniform sampler2D colorTexture;

varying vec2 vUv;

void main( void ) {

  vec2 position = - 1.0 + 2.0 * vUv;

  float a = atan( position.y, position.x );
  float r = sqrt( dot( position, position ) );

  vec2 uv;
  uv.x = cos( a ) / r;
  uv.y = sin( a ) / r;
  uv /= 10.0;
  uv += time * 0.05;

  vec3 color = texture2D( colorTexture, uv ).rgb;

  gl_FragColor = vec4( color * r * 1.5, 1.0 );

}
```

```glsl
uniform float time;

varying vec2 vUv;

void main( void ) {

  vec2 position = vUv;

  float color = 0.0;
  color += sin( position.x * cos( time / 15.0 ) * 80.0 ) + cos( position.y * cos( time / 15.0 ) * 10.0 );
  color += sin( position.y * sin( time / 10.0 ) * 40.0 ) + cos( position.x * sin( time / 25.0 ) * 40.0 );
  color += sin( position.x * sin( time / 5.0 ) * 10.0 ) + sin( position.y * sin( time / 35.0 ) * 80.0 );
  color *= sin( time / 10.0 ) * 0.5;

  gl_FragColor = vec4( vec3( color, color * 0.5, sin( color + time / 3.0 ) * 0.75 ), 1.0 );

}
```

```glsl
uniform float time;

varying vec2 vUv;

void main( void ) {

  vec2 position = - 1.0 + 2.0 * vUv;

  float red = abs( sin( position.x * position.y + time / 5.0 ) );
  float green = abs( sin( position.x * position.y + time / 4.0 ) );
  float blue = abs( sin( position.x * position.y + time / 3.0 ) );
  gl_FragColor = vec4( red, green, blue, 1.0 );

}
```

