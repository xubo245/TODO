# Paint的深入理解与使用
---
## 1.颜色
基本颜色->ColorFilter->Xfermode
### 1.1基本颜色
Canvas.drawColor/ARGB()-颜色参数  
Canvas.drawBitmap()-Bitmap参数  
Canvas 图形或者文字绘制 -paint 参数
  
**Paint**设置颜色有俩种方式：一种是直接用Paint.setColor/ARGB(),另一种是使用Shader来指定着色方案
#### 1.1.1 直接设置颜色
	Paint paint = new Paint();
	paint.setColor(Color.parseColor("#009966"))
	paint.setARGB(0,100,100,100)
#### 1.1.2 setShader(Shader shader)
shader又被称为着色器，是图形领域一个通用的概念，与直接设置颜色的区别是：着色器设置的是一个颜色方案，或者说是一套着色规则。当设置了shader之后，paint在绘制图形和文字时就不使用setColor/ARGB了。 
   
另外Shader这个类，我们通常不会直接使用，而是使用其子类BitmapShader,ComposeShader,LinearGradient,RadialGradient,SweepGradient
##### 1.1.2.1 LinearGradient 线性渐变  
设置俩个点和俩种颜色，以这俩个点为端点，渐变出的颜色用来绘制。

	Shader shader = new LinearGradient(0,0,100,100,Color.RED,Color.GREEN,Shader.TileMode.CLAMP);
	paint.setShader(shader)
	
需要注意的是 模式：CLAMP 会在端点之外 延续端点处颜色，MIRROR 镜像模式，REPEAT 重复模式

##### 1.1.2.2 RadicalGradient 辐射渐变
就是从中心向周围辐射渐变
  
与LinearGradient 类似的使用方法。。具体去查文档就可以了。同样也有三个模式  CLAMP ,REAPEAT ,MIRROR

##### 1.1.2.3 SweepGradient 扫描渐变
有点类似雷达扫描的那种效果

##### 1.1.2.4 BitmapShader
就是用Bitmap的像素来作为图形或文字的填充
例如：利用canvas.drawCircle() 和 setShader(bitmapShader),可以实现绘制圆形bitmap的效果  
**注意：默认的bitmap是画在左上角的！另外bitmap大小不会更改的**

##### 1.1.2.5 ComposeShader 混合着色器
就是把俩个着色器一起使用  
**注意：ComposeShader()需关闭硬件加速  **
这里注意一个参数 ProterDuff.Mode，是用来指定俩个图案共同绘制时的颜色策略。它是一个enum，**[颜色策略」的意思**，就是说把源图像绘制到目标图像处时应该怎样确定二者结合后的颜色，而对于 ComposeShader(shaderA, shaderB, mode) 这个具体的方法，就是指应该怎样把 shaderB 绘制在 shaderA 上来得到一个结合后的 Shader。  
  
具体来说，PorterDuff.Mode有17个，分为俩大类：  
1.Alpha合成  
2.混合  
具体效果直接看官方文档https://developer.android.com/reference/android/graphics/PorterDuff.Mode.html


### 1.2 setColorFilter(ColorFilter colorFilter)  
为绘制设置颜色过滤。颜色过滤的意思，就是为绘制的内容设置一个统一的过滤策略，然后 Canvas.drawXXX() 方法会对每个像素都进行过滤后再绘制出来。
  
ColorFilter 并不会直接被使用，而是会使用其子类：  
ColorMatrixColorFilter,LightingColorFilter,PorterDuffColorFilter
#### 1.2.1 LightingColorFilter  
光照颜色过滤器 ,完成色彩过滤和色彩增强的功能
  
`LightingColorFilter (int mul, int add)`  
构造方法中有俩个参数，一个是mul 用来和目标像素相乘，一个是add 用来和目标像素相加。  
**计算公式：**  
R' = R * mul.R / 0xff + add.R  
G' = G * mul.G / 0xff + add.G  
B' = B * mul.B / 0xff + add.B  


一个**保持原样**的基本LightingColorFilter,mul为0xffffff,add为0x000000,那么对于一个像素  
例如我想去除红色,可以将mul 改为0x00ffff(RGB R部分为0)，其计算过程：  
R' = R * 0x0 / 0xff + 0x0 = 0  // 红色被移除  
G' = G * 0xff / 0xff + 0x0 = G  
B' = B * 0xff / 0xff + 0x0 = B    

#### 1.2.2 PorterDuffColorFilter  
使用一个指定的颜色和一种指定的PorterDuff.Mode来与绘制对象进行合成。  
就是与ComposeShader相似，只是PorterDuffColorFilter 只能指定一种颜色作为源，而不能是一个Bitmap  
>src是所设置的color  
>dst是所绘制的图形  

- Mode.ADD(饱和度相加)，Mode.DARKEN（变暗），Mode.LIGHTEN（变亮），Mode.MULTIPLY（正片叠底），Mode.OVERLAY（叠加），Mode.SCREEN（滤色） 

例如可以利用  PorterDuff.Mode.SRC(Mode.SRC、Mode.SRC_IN、Mode.SRC_ATOP)来实现 v4包中的DrawableCompat.setTint()

#### 1.2.3 ColorMatrixColorFilter  
使用一个ColorMatrix来对颜色进行处理  
其内部是一个4*5的矩阵：  
[ a, b, c, d, e,  
  f, g, h, i, j,  
  k, l, m, n, o,  
  p, q, r, s, t ]  
转换公式是这样的：  
R’ = a*R + b*G + c*B + d*A + e;  
G’ = f*R + g*G + h*B + i*A + j;  
B’ = k*R + l*G + m*B + n*A + o;  
A’ = p*R + q*G + r*B + s*A + t;  
  
另外ColorMatrix有些自带的方法可以做简单的转换，例如setSaturation(float sat)来设置饱和度，当然也可以自己手动设置每一个原色来进行调整  
参考：https://github.com/chengdazhi/StyleImageView

### 1.3 setXfermode(Xfermode xfermode)  
处理的是颜色遇上view的问题，其实就是`transfer mode` ,用X 替代trans 是一种美国人喜欢的简写。。。。。  
Xfermode 严谨地讲， Xfermode 指的是你要绘制的内容和 Canvas 的目标位置的内容应该怎样结合计算出最终的颜色。但通俗地说，**其实就是要你以绘制的内容作为源图像，以 View 中已有的内容作为目标图像，选取一个 PorterDuff.Mode 作为绘制内容的颜色处理方案 ** 

PorterDuff.Mode 在Paint中一共有三处被用到，其工作原理都一样，只是用途不同：  ComposeShader,PorterDuffColorFilter,Xfermode. 用途分别是：混合俩个Shader，增加一个单色的ColorFilter,设置绘制内容和View中已有内容的混合计算方式。  

另外，创建Xfermode，也是在创建其子类： PorterDuffXfermode,AvoidXfermode（已废弃）,PixelXorXfermode（已废弃）

- PorterDuffXferMode 实例：  

		int layerID = canvas.saveLayer(0, 0, mWidth, mHeight, mPaint, Canvas.ALL_SAVE_FLAG);
    	canvas.translate(mWidth / 2, mHeight / 2);
    	mPaint.setColor(Color.BLUE);
    	canvas.drawCircle(0, 0, 100, mPaint);
    	mPaint.setColor(Color.RED);
    	mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC));
    	canvas.drawRect(0, 0, 200, 200, mPaint);
    	canvas.restoreToCount(layerID);
	先画的是DST 目标图 后画的是 SRC 源图 ，


- **Xfermode注意事项:**
通常情况下，我们**drawBitmap->setXfermode->drawBitma**p .这个流程我们往往得不到想要的结果，因为我们在第三部draw的时候，我们以为只是第一步draw的图案参与计算，实际上是 view的显示区域都参与计算！！ 并且默认的view底色并不是透明色。
 
	1. 用离屏缓冲(Off-screen BUffer)    
	
     解决办法就是使用离屏缓存，就是把内容绘制在额外的图层上，再把绘制好的内容贴回view中。  
  
     使用离屏缓存有俩种方式：  
     1.canvas.saveLayer()  

     	使用办法就是在 绘制代码前保存，绘制之后恢复  

     2.View.setLayerType

		 View.setLayerType()是把整个view都绘制在离屏缓存中  
		 View.setLayerType(LAYER_TYPE_HARDWARE)是使用GPU缓冲  
		 View.setLayerType(LAYER_TYPE_SOFTWARE)是直接使用一个bitmap来缓冲    
 
	2. 控制好透明区域  
     使用Xfermode来绘制内容，需要注意控制它的透明区域不要太小，要让它足够**覆盖和它结合绘制的内容**！！
	

- 源图像在运算时，只是在源图像所在区域与对应区域的目标图像做运算。所以**目标图像与源图像不相交的地方是不会参与运算的**！这一点非常重要！不相交的地方不会参与运算，所以不相交的地方的图像也不会是脏数据，也不会被更新，所以不相交地方的图像也永远显示的是目标图像。

- **PorterDuffColorFilter**中只能使用纯色彩，而且是完全覆盖在图片上方；而**setXfermode()**则不同，它只会在目标图像和源图像交合的位置起作用，而且源图像不一定是纯色的。

- **canvas脏区域更新原理** 
	Android在绘图时会先检查该画笔Paint对象有没有设置Xfermode，如果没有设置Xfermode，那么直接将绘制的图形覆盖Canvas对应位置原有的像素；如果设置了Xfermode，那么会按照Xfermode具体的规则来更新Canvas中对应位置的像素颜色。 


各种mode实现的效果:  

1. Mode.SRC_IN  dst和src相交区域会显示，dst图像不存在的区域不会显示(圆角图片，图片倒影)
2. Mode.SRC_OUT dst和src相交区域为空像素,当没有dst 只有src时 会显示src(橡皮擦效果，刮刮卡效果)
3. Mode.SRC_OVER 
4. Mode.SRC_ATOP 1、当透明度只有100%和0%时，SRC_ATOP是SRC_IN是通用的 2、当透明度不只有100%和0%时，SRC_ATOP相比SRC_IN源图像的饱和度会增加，即会显得更亮！
5. Mode.DST_IN dst和src相交区域会显示,src图像不存在的区域不会显示
6. Mode.DST_OUT 
7. Mode.DST_OVER
8. Mode.DST_ATOP
9. Mode.CLEAR 源图像所在区域都会变成空像素！ 
10. Mode.XOR  


- 1:DST相关模式是完全可以使用SRC对应的模式来实现的，只不过需要将目标图像和源图像对调一下即可。 
2:在SRC模式中，是以显示源图像为主，通过目标图像的透明度来调节计算结果的透明度和饱和度，而在DST模式中，是以显示目标图像为主，通过源图像的透明度来调节计算结果的透明度和饱和度。


- 可以从下面三个方面来决定使用哪一个模式：
1、首先，目标图像和源图像混合，需不需要生成颜色的叠加特效，如果需要叠加特效则从颜色叠加相关模式中选择，有Mode.ADD（饱和度相加）、Mode.DARKEN（变暗），Mode.LIGHTEN（变亮）、Mode.MULTIPLY（正片叠底）、Mode.OVERLAY（叠加），Mode.SCREEN（滤色） 
2、当不需要特效，而需要根据某一张图像的透明像素来裁剪时，就需要使用SRC相关模式或DST相关模式了。由于SRC相关模式与DST相关模式是相通的，唯一不同的是决定当前哪个是目标图像和源图像； 
3、当需要清空图像时，使用Mode.CLEAR




**canvas.saveLayer(null,null,Canvas.ALL_SAVE_FLAG);**
  
## 2.效果  
效果类的API,指的就是抗锯齿,填充/轮廓,线条宽度等等  
### 2.1 setAntiAlias(Boolean aa)设置抗锯齿  
Paint 构造方法里也可以打开。。  
### 2.2 setStyle(Paint.Style style)  
设置线条风格，  `FILL/STROKE/FILL_AND_STOKE  `
### 2.3 线条形状  
设置线条形状有四个方法:  
setStrokeWidth(float width)  
setStrokeCap(Paint.Cap cap)   
setStrokeJoin(Paint.Join join)  
setStrokeMiter(float miter)
  
#### 2.3.1 setStokeWidth(float width)
设置线条宽度，单位px,默认值0

>线条宽度 0 和 1 的区别

> 默认情况下，线条宽度为 0，但你会发现，这个时候它依然能够画出线，线条的宽度为 1 像素。那么它和线条宽度为 1 有什么区别呢？
> 
> 其实这个和后面要讲的一个「几何变换」有关：你可以为 Canvas 设置 Matrix 来实现几何变换（如放大、缩小、平移、旋转），在几何变换之后 Canvas 绘制的内容就会发生相应变化，包括线条也会加粗，例如 2 像素宽度的线条在 Canvas 放大 2 倍后会被以 4 像素宽度来绘制。而当线条宽度被设置为 0 时，它的宽度就被固定为 1 像素，就算 Canvas 通过几何变换被放大，它也依然会被以 1 像素宽度来绘制。Google 在文档中把线条宽度为 0 时称作「hairline mode（发际线模式）」。
 
#### 2.3.2 setStrokeCap(Paint.Cap cap)  
设置线头的形状。  
有三种形状： BUTT平头，ROUND 圆头，SQUARE方头。默认为BUTT
  
#### 2.3.3 setStrokeJoin(Paint.Join join)  
设置线条拐角的形状。  
有三个值： MITER 尖角，BEVEL 平角，ROUND 圆角。默认MITER
  
#### 2.3.4 setStrokeMiter(float miter)  
这个方法是对setStrokeJoin的一个补充，用于设置MITER型拐角的延长线的最大值。    
>所谓**延长线最大值**，就是当线条拐角设置为MITER时，拐角处 外边缘需要使用延长线来补偿。  
>但是这种延长线有一个缺陷，那就是当拐角角度过小，就有可能出现连接点过长的情况。所以为了避免这种情况，MITER型连接点有一个额外规则：当尖角过长时，自动改用BEVEL的方式来渲染连接点。那究竟角度达到什么程度需要由BEVEL控制，就是通过setStrokeMiter()设置的值来控制。    
>具体来讲，就是指尖角的外边缘点和内部拐角的距离/线条宽度 这个俩个长度的比。  


### 2.4 色彩优化
Paint 的色彩优化有两个方法： setDither(boolean dither) 和 setFilterBitmap(boolean filter) 。它们的作用都是让画面颜色变得更加「顺眼」，但原理和使用场景是不同的。  
#### 2.4.1 setDither(boolean dither)  
设置图像抖动。  
>所谓抖动，是指把图像从较高色彩深度（即可用的颜色数）向较低色彩深度的区域绘制时，在图像中有意地插入噪点，通过有规律地扰乱图像来让图像对于肉眼更加真实的做法。   
>
>比如向 1 位色彩深度的区域中绘制灰色，由于 1 位深度只包含黑和白两种颜色，在默认情况下，即不加抖动的时候，只能选择向上或向下选择最接近灰色的白色或黑色来绘制，那么显示出来也只能是一片白或者一片黑。而加了抖动后，就可以绘制出让肉眼识别为灰色的效果了  

在现在的Android版本的绘制，默认色彩深度已经是32位的ARGB_8888，效果已经足够清晰。只有在自建Bitmap，并选择ARGB_4444或RGB_565时，开启才会有较明显效果  

#### 2.4.2 setFilterBitmap(boolean filter)
设置是否使用双线性过滤来绘制Bitmap ，适合放大绘制Bitmap时开启。
图像在放大绘制的时候，默认使用最近邻插值过滤，但是会出现马赛克效果。如果开启双线性过滤，图像会更加平滑。  
  
### 2.5 setPathEffect(PathEffect effect)  
使用pathEffect来给图形的轮廓设置效果。对canvas所有的图形绘制都有效果。   
>**PathEffect 分为两类**，单一效果的  CornerPathEffect DiscretePathEffect DashPathEffect PathDashPathEffect ，和组合效果的  SumPathEffect ComposePathEffect。  


#### 2.5.1CornerPathEffect  
把所有的拐角变成圆角

#### 2.5.2 DiscretePathEffect  
把线条进行随机的偏离，让轮廓变得乱七八糟。

#### 2.5.3 DashPathEffect  
使用虚线来绘制线条  
在Paint.Style == FILL的时候无效   
另外构造方法中的phase意思是 起始位置的偏移量(**即线条向左整体位移的距离**)
  
#### 2.5.4 PathDashPathEffect  
使用一段path 来绘制虚线。  
PathDashPathEffect(Path shape, float advance, float phase, PathDashPathEffect.Style style)  
其构造方法中 最后一个参数，PathDashPathEffect.Style 是一个enum，有三个值，TRANSLATE位移，ROTATE旋转，MORPH变体  
advance 是两个相邻的 shape 段之间的间隔，不过注意，这个间隔是两个 shape 段的起点的间隔，而不是前一个的终点和后一个的起点的距离；  phase 和 DashPathEffect 中一样，是虚线的偏移  

![PathDashPathEffect](http://ww1.sinaimg.cn/large/6ab93b35gy1fhuzny03xgj20kn0h3myw.jpg)
  
#### 2.5.5 SumPathEffect  
组合效果类PathEffect。就是分别按照俩种PathEffect分别对目标进行绘制。。

#### 2.5.6 ComposePathEffect  
组合效果类PathEffect。是先对目标Path 使用一个PathEffect，然后对这个经过PathEffect处理的Path，再用另外一个PathEffect进行处理。  

>注意PathEffect在有些情况下，不支持硬件加速，需要关闭硬件加速才能正常使用  
>1. **canvas.drawLine()和canvas.drawLines()方法画直线时，setPathEffect()是不支持硬件加速。。 ** 
>2.PathDashPathEffect对硬件加速的支持有问题，所以在使用这个的时候，也需要关闭硬件加速  

#### 2.6 setShadowLayer(float radius,float dx,float dy,int shadowColor)  
作用是：在绘制的内容下面加一层阴影    
- 如果要清除阴影层，使用clearShadowLayer()  

- 在硬件加速开启的情况下，setShadowLayer()只支持文字的绘制，文字之外的绘制必须关闭硬件加速才能正常绘制阴影    

- 如果shadowColor 是半透明的，阴影的透明度 就是使用shadowColor自己的透明度。如果shadowColor是不透明的，那么阴影的透明度就是使用paint的透明度

- 如果绘制的内容是图片,则阴影是图片副本而不是我们设置的shadowColor


#### 2.7 setMaskFilter(MaskFilter maskfilter)  
为之后的绘制设置MaskFilter 。setShadowLayer是设置在绘制层下方的附加效果，而这个MaskFilter相反，是设置在绘制层上方的效果。  
>现在已经出现了俩个setXXXFilter ,一个是setColorFilter,是对每个像素的颜色进行过滤，而这里的setMaskFilter是基于整个画面来进行过滤  

MaskFilter有俩种：BlurMaskFilter和EmbossMaskFilter

##### 2.7.1 BlurMaskFilter  
产生一种模糊效果  
它的构造方法 BlurMaskFilter(float radius, BlurMaskFilter.Blur style) 中， radius 参数是模糊的范围， style 是模糊的类型。一共有四种:

- NORMAL:内部外部都模糊
- SOLID:内部正常绘制，外部模糊
- INNER:内部模糊，外部不绘制
- OUTER:内部不绘制，外部模糊

- 可以实现为图片生成指定颜色的阴影(setShadowLayer能生成阴影,但是为图片生成的阴影是经过高斯模糊的图片副本！),通过bitmap.extractAlpha()获取拥有原图alpha值的副本,然后通过mPaint设置颜色 用来设置绘制副本时的颜色！最后再同一位置 绘制原图即可。注意：通过位置控制偏移.


##### 2.7.2 EmbossMaskFilter
产生一种浮雕效果  

#### 2.8 获取绘制的path  
这是效果类唯一的一组get方法  
这组方法的作用是，根据paint的设置，计算出绘制Path或文字时的**实际path**    
##### 2.8.1 getFillPath(Path src,Path dst)  
获取实际Path。所谓实际Path 就是指 drawPath()的绘制内容的轮廓，要算上线条宽度和设置的PathEffect。  

默认情况下(线条宽度为0，没有PathEffect)，原Path和实际Path是一样的。而在线条宽度不为0(且模式为STROKE或FILL_AND_STROKE)，或者是设置了PathEffect的时候，实际Path就和原Path不一样了：  

![getFillPath](http://ww1.sinaimg.cn/large/6ab93b35gy1fhrmz7dwthj20rw0mewhi.jpg)
  
getFillPath(src,dst),src是原path，dst 是传入的空path，用来存储实际path。  

**注意:关闭硬件加速 setLayerType(View.LAYER_TYPE_SOFTWARE,null)**  

##### 2.8.2 getTextPath  
文字的绘制，虽然是使用Canvas.drawText()方法，但是在底层一点,文字信息全是被转化成图形，对图形进行绘制。  
getTextPath就是获取目标文字对应的path。  

![getTextPath](http://ww1.sinaimg.cn/large/6ab93b35gy1fhrodm10sfj20i005m74z.jpg)

## 3.drawText相关
Paint 有些设置是与文字绘制相关的，即和drawText()相关  
例如文字大小，间隔，文字效果等等  

## 4.初始化
也就是用来初始化Paint对象，或者是批量设置Paint的多个属性的方法  
###4.1 reset()  
重置Paint的所有属性为默认值，相当于重新new一个。不过性能肯定是更好的  
### 4.2 set(Paint src)  
把传入的Paint的属性赋值到当前Paint  
### 4.3 setFlags(int flags)  
批量设置flags。相当于调用他们的set方法。可以用**“|”**这个符号来设置多个。  
>例如：
>paint.setFlags(Paint.ANTI_ALIAS_FLAG|Paint.DITHER_FLAG)  
>相当于  
>paint.setAntiAlias(true)  
>paint.setDither(true)  
