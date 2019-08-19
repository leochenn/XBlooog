[TOC]

## Drawable

#### Drawable绘制流程

使用Drawable最通常的步骤：
通过Resource获取Drawable实例
将获取的Drawable实例当做背景设置给View或者作为ImageView的src进行显示:

- 通过Resource实例加载一张资源图片：.9图返回1个NinePatchDrawable实例；普通图片返回1个BitmapDrawable实例. ([引用](https://www.jianshu.com/p/2213c62e4738))

```java
private static Drawable drawableFromBitmap(Resources res, Bitmap bm, byte[] np,
        Rect pad, Rect layoutBounds, String srcName) {

    if (np != null) {
        return new NinePatchDrawable(res, bm, np, pad, layoutBounds, srcName);
    }

    return new BitmapDrawable(res, bm);
}
```

- 将获取的Drawable实例当做背景设置给View

  targetView.setBackgroundDrawable(Drawable bg)

  View实例的背景Drawable实例最终还是调用自身的Drawable.draw方法绘制到屏幕上

  ```java
  public class BitmapDrawable extends Drawable {
      @Override
      public void draw(Canvas canvas) {
          ****
          if (shader == null) {
              ****
              //最终调用了Canvas.drawBitmap方法，将Drawable实例中的bitmap绘制到View实例关联的画布上
              canvas.drawBitmap(bitmap, null, mDstRect, paint);
              if (needMirroring) {
                  canvas.restore();
              }
          } ****
      }
  }
  ```

- 总结

  **1：setBackgroundDrawable方法，调用了Drawable的一系列方法，设置了Drawable实例一系列属性值，最终引发了View实例的重新布局（requestLayout()），重绘（invalidate(true)）及重建View实例的外部轮廓（invalidateOutline()）**

  **2：在View实例重绘过程的第一步，将得到的Drawable实例（View实例的背景）绘制到屏幕上，实质是调用了Drawable.draw(@NonNull Canvas canvas)**

  **3：Drawable.draw本身是个抽象方法，绘制具体逻辑由其子类实现。 我们以之前获得的BitmapDrawable为例进行分析： 最终调用了Canvas.drawBitmap方法，将Drawable实例中的bitmap绘制到View实例关联的画布上**

  作者：幻海流心

  链接：https://www.jianshu.com/p/2213c62e4738

  来源：简书

  简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。



#### ImageView设置src Drawable

首先ImageView构造方法中，会解析xml配置的src资源生成Drawable实例，然后调用setImageDrawable方法进行设置；初始化时，ImageView默认设置的ScaleType为Fit_Center

```java
 public void setImageDrawable(@Nullable Drawable drawable) {
        if (mDrawable != drawable) {
            ****
            //对src生成的Drawable实例设置一系列属性
            updateDrawable(drawable);
            ****
            //最后调用invalidate()触发draw
            invalidate();
        }
    }

    private void updateDrawable(Drawable d) {
        ****
        //将ImageView实例之前关联的Drawable实例的动画监听移除，
        //并停止其已经在执行的动画，解除其所有事件
        if (mDrawable != null) {
            mDrawable.setCallback(null);
            unscheduleDrawable(mDrawable);
            if (isAttachedToWindow()) {
                mDrawable.setVisible(false, false);
            }
        }
        //将mDrawable赋值为通过src属性生成的Drawable实例
        mDrawable = d;
        if (d != null) {
            //为通过src生成的Drawable实例设置动画监听为ImageView实例自身；
            //并设置其布局方向，状态数组，Drawable动画是否开启，Drawable的level值。
            d.setCallback(this);
            d.setLayoutDirection(getLayoutDirection());
            if (d.isStateful()) {
                d.setState(getDrawableState());
            }
            if (isAttachedToWindow()) {
                d.setVisible(getWindowVisibility() == VISIBLE && isShown(), true);
            }
            d.setLevel(mLevel);
            mDrawableWidth = d.getIntrinsicWidth();
            mDrawableHeight = d.getIntrinsicHeight();
            //根据当前ImageView实例的ColorStateList对其进行着色
            applyImageTint();
            //未执行实质代码
            applyColorMod();
            //设置Drawable实例的绘制范围不变，并根据ImageView实例内容区域和
            //Drawable实例原始绘制范围，确定Drawable实例在实际绘制时候的缩放。
            configureBounds();
        } else {
            mDrawableWidth = mDrawableHeight = -1;
        }
    }
```

重要：上述代码涉及到如何解除ImageView与Drawable的绑定关系

```java
private void configureBounds() {
        //通过src生成的Drawable实例 原始宽高
        final int dwidth = mDrawableWidth;
        final int dheight = mDrawableHeight;
        //ImageView实例的内容区域宽高(去除了padding值)
        final int vwidth = getWidth() - mPaddingLeft - mPaddingRight;
        final int vheight = getHeight() - mPaddingTop - mPaddingBottom;
        ****
        if (dwidth <= 0 || dheight <= 0 || ScaleType.FIT_XY == mScaleType) {
            //当ImageView设置过android:scaleType="fitXY" 或setScaleType(ScaleType.FIT_XY)，
            //则将此Drawable实例的绘制范围设定为ImageView实例的内容区域
            mDrawable.setBounds(0, 0, vwidth, vheight);
            mDrawMatrix = null;
        } else {
            //对应我们例子中，未设置android:scaleType情况下，
            //通过src生成的Drawable实例的绘制范围就是其原始范围
            mDrawable.setBounds(0, 0, dwidth, dheight);
            //下面代码设置了mDrawMatrix的属性
            //在initImageView()方法中已知：
            //ImageView中mScaleType默认就是ScaleType.FIT_CENTER
            if (ScaleType.MATRIX == mScaleType) {
                ****
            } else if (fits) {
                ****
            } else if (ScaleType.CENTER == mScaleType) {
                ****
            } else if (ScaleType.CENTER_CROP == mScaleType) {
                ****
            } else if (ScaleType.CENTER_INSIDE == mScaleType) {
                ****
            } else {
                //ImageView中mScaleType默认就是ScaleType.FIT_CENTER
                //则根据ImageView实例内容区域的范围和Drawable实例实际宽高来设置mDrawMatrix
                mTempSrc.set(0, 0, dwidth, dheight);
                mTempDst.set(0, 0, vwidth, vheight);
                mDrawMatrix = mMatrix;
                mDrawMatrix.setRectToRect(mTempSrc, mTempDst, scaleTypeToScaleToFit(mScaleType));
            }
        }
    }
}
```

重点：ScaleType如何在ImageView中生效




ImageView实例生成后，肯定还是执行onDraw方法将自身绘制到屏幕上，继续追踪代码：


```java
public class ImageView extends View {
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        ****
        //在上面分析过 mDrawMatrix不为null
        //mDrawMatrix的属性根据ImageView实例内容区域的范围和Drawable实例实际宽高来配置
        if (mDrawMatrix == null && mPaddingTop == 0 && mPaddingLeft == 0) {
            //如果矩阵mDrawMatrix为空，且ImageView的上下padding值都为0
            //则直接将Drawable实例绘制到画布上
            mDrawable.draw(canvas);
        } else {
            ****
            //我们例子中，矩阵mDrawMatrix不为空，则将其设置到ImageView的画布上
            if (mDrawMatrix != null) {
                canvas.concat(mDrawMatrix);
            }
            //然后在画布上面绘制Drawable实例
            mDrawable.draw(canvas);
            canvas.restoreToCount(saveCount);
        }
    }
}
```



## ImageView

#### ImageView设置图片的几种方式

附上一张ImageView设置图片的不同方式

在布局文件中设置属性 android:src=“@drawable/resId” 加载本地图片

setImageResource(int resId);  加载drawable文件夹中的资源文件

setImageURI(Uri);  加载手机内存卡中的图片格式文件

setImageBitmap(Bitmap);  加载 Bitmap

setImageDrawable(Drawable); 加载 Drawable

作者：Coralline_xss

链接：https://www.jianshu.com/p/835297601f84

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

![setImage](extra\Drawable-ImageView调用不同方法设置图片.png)

通过上述流程图，可以得知：

- 所有的图片格式，不管是资源，还是 Uri 还是 Bitmap ，都会转化为 Drawable；
- setImageURI(Uri) 方法，涉及到将文件转化为文件流，然后将文件流解析为 Bitmap 操作，需要注意的是在主线程中做这些操作，可能会造成延时；
- 针对性能方面，给上述加载图片方法排序，从劣到优，可以这样来：setImageURI() < setImageBitmap() < setImageRecource() < 属性设置 < setImageDrawable() ，能肯定的是，setImageDrawable() 是最优设置方法。



#### ImageView 几种ScaleType

链接：[ImageView的ScaleType原理及效果分析](https://www.jianshu.com/p/fe5d2e3feed3)

![ImageView-ScaleType](extra\ImageView-ScaleType.png)





#### updateDrawable具体工作

![ImageView-updateDrawable](extra\ImageView-updateDrawable.png)





#### 图片与内存、分辨率关系

``缩放类型不会影响图片占用内存大小，只会影响 Drawable 在绘制图片到屏幕时的区域大小，图片最终是以 Bitmap 形式存在于内存中的，ImageView 的缩放类型只是将图片展示的区域按缩放规则进行划定，并没有对图片本身产生作用，所以 scaleType 缩放对图片占用的内存大小并没有什么关系``

- 加载资源文件时，计算的图片占用内存大小同图片所在drawable 文件夹和展示图片设备的分辨率有关；
- 加载内部存储中的图片文件时，则同设备屏幕密度无关，为图片本身大小。





## 图片的压缩

在 Android 设备中，图片有如下三种存在形式：

1. 在硬盘上时，图片展现的方式是 File

2. 在网络传输时，图片展现的方式是 Stream

3. 在内存中，图片展现的方式是 Stream 或 Bitmap

   

因此，我们既要压缩图片 File 大小（减轻服务器存储压力），又要压缩图片 Pixel（避免 OOM）

Android 中有两种压缩图片的方法：

1. 质量压缩（不改变图片的尺寸）

   质量压缩一般可用于上传大图前的处理，这样就可以节省一定的流量。
   所谓的质量压缩，它其实只能实现对 File 的影响。
   缺点：照片失真、耗时

2. 尺寸压缩（相当于是像素上的压缩）

   尺寸压缩一般可用于生成缩略图
   通过缩放图片像素来减少图片占用内存大小



首先要知道 Bitmap 所占内存大小计算方式：图片长度 x 图片宽度 x 一个像素点占用的字节数

长宽可以通过尺寸压缩来实现，而字节数则通过Bitmap.Config的四个值来实现



关于大图加载，首先需要了解局部加载``BitmapRegionDecoder``([鸿洋的文章](https://blog.csdn.net/lmj623565791/article/details/49300989))



参考：

1.如何进行压缩、以及执行顺序，和加载大图（[引用](https://imeiji.github.io/2018/07/08/%E8%B0%88%E8%B0%88%20Android%20%E5%8E%8B%E7%BC%A9%E5%92%8C%E5%8A%A0%E8%BD%BD%E5%A4%A7%E5%9B%BE%E7%89%87/)）

2.Android图片压缩（质量压缩和尺寸压缩）&Bitmap转成字符串上传（[引用](https://blog.csdn.net/jdsjlzx/article/details/44228935)）



## 性能优化：Bitmap详解&你的Bitmap占多大内存？

#### DisplayMetrics

屏幕密度相关类，可以获取屏幕高宽、屏幕密度density,每英寸点数densityDpi（160,240,320...640）

在160dpi屏幕上 1dp = 1px， 320dpi  1dp= 2px； densityDpi 与density线性相关

density 的计算： 1dp = density px，   densityDpi（px）= density（dp）* 160



#### 资源文件夹代表的密度

| 密度类型         | 分辨率    | 对角长 | 像素密度dpi | px/dp      | 比例 |
| ---------------- | --------- | ------ | ----------- | ---------- | ---- |
| 低密度ldpi       | 240*320   | 400px  | 120         | 1dp=0.75px | 3    |
| 中mdpi           | 320*480   | 576    | 160         | 1=1        | 4    |
| 高hdpi           | 480*800   | 932    | 240         | 1=1.5      | 6    |
| 超高xhdpi        | 720*1280  | 1468   | 320         | 1=2        | 8    |
| 超超高密度xxhdpi | 1080*1920 | 2202   | 480         | 1=3        | 12   |



#### [Bitmap ImageView大小的一些秘密](https://juejin.im/post/5bc437695188255c9e02fba3)



#### 读取res目录下的图片流程

res中的图片显示到imageview（wrap_content），图片的大小和imageview的大小是如何计算的

通过详解代码来演示了这一计算过程：

BitmapFactory.decodeResource-decodeResourceStream-decodeStream



总结：

在读取资源时，使用openRawResource 方法，然后会对TypedValue赋值，其中包含了原始资源的density等信息，也就是文件夹代表的density；

调用decodeResourceStream对原始资源进行适配和解码，实际是原始资源density到设备屏幕density的映射 （下面Bitmap的创建流程代码里可见具体逻辑）



1.对于Bitmap，大小等于 rawSize * targetDensity ／ rawDensity，targetDensity是目标的density， rawDensity是原始资源的density，当然这两个值都可以通过options进行修改，其实从这里也可以看出图片资源放在适合的资源夹的重要性，如果图片资源放的文件夹density太小，会导致解码的bitmap放大，从而导致内存增加，毕竟解码之后的面积变大了，单位面积的占用内存又不变。

2.对于ImageView，我们可以知道，即使我们对bitmap进行了缩放，在内存的drawable又会重新进行缩放，以用来适应实际大小。缩放比例我们还是可以通过targetDensity，inDensity修改进行控制的。



#### Bitmap创建流程

BitmapFactory 提供了五种方式来创建Bitmap，分别是：**decodeFile, decodeResource, decodeByteArray, decodeStream, decodeFileDescription**.



最常用的三个方法：**decodeFile, decodeResource, decodeStream**，前两个最终调用的是 **decodeStream**;

**decodeStream, decodeByteArray, decodeFileDescription **这三个内部则调用的是 native 方法来创建 Bitmap的【有种说法，Bitmap是Android中唯一通过 native 方法创建的类】；

 **decodeResourceStream**主要做了两件事：一是对 opts.inDensity 赋值，没有设置默认值 160；二是对 opts.inTargetDensity 赋值，没有赋值为当前设备 densityDpi；

 **decodeStream**主要也做了两件事：一是调用 native 方法解析 Bitmap；二是对解析得到的 Bitmap 调用 setDensityFraomOptions(bmp, opts) 进行设置；

 **setDensityFraomOptions(bmp, opts)**主要做了这样几件事：一是当opts.inDensity != opts.inTargetDensity || opts.inDensity != opts.inScreenDensity && (inScaled = true || isNinePatch) 时，将设置 outputBitmap.mDensity = inTargetDensity；

链接：https://www.jianshu.com/p/4ba3e63c8cdc

```java
public static Bitmap decodeResourceStream(Resources res, TypedValue value,
            InputStream is, Rect pad, Options opts) {

        if (opts == null) {
            opts = new Options();
        }

        if (opts.inDensity == 0 && value != null) {
            final int density = value.density;
            if (density == TypedValue.DENSITY_DEFAULT) {
                opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;
            } else if (density != TypedValue.DENSITY_NONE) {
                opts.inDensity = density;
            }
        }
        
        if (opts.inTargetDensity == 0 && res != null) {
            opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
        }
        
        return decodeStream(is, pad, opts);
    }
```

value.density 资源文件夹密度

DisplayMetrics.DENSITY_DEFAULT 屏幕默认密度=160

res.getDisplayMetrics().densityDpi 当前设备的显示密度

结论：inDensity即原始资源文件夹密度， inTargetDensity为当前设备显示密度



```java
private static void setDensityFromOptions(Bitmap outputBitmap, Options opts) {
        if (outputBitmap == null || opts == null) return;

        final int density = opts.inDensity;
        if (density != 0) {
            outputBitmap.setDensity(density);
            final int targetDensity = opts.inTargetDensity;
            if (targetDensity == 0 || density == targetDensity || density == opts.inScreenDensity) {
                return;
            }

            byte[] np = outputBitmap.getNinePatchChunk();
            final boolean isNinePatch = np != null && NinePatch.isNinePatchChunk(np);
            if (opts.inScaled || isNinePatch) {
                outputBitmap.setDensity(targetDensity);
            }
        } else if (opts.inBitmap != null) {
            // bitmap was reused, ensure density is reset
            outputBitmap.setDensity(Bitmap.getDefaultDensity());
        }
    }
```



一张图片对应 Bitmap 占用内存 = mBitmapHeight * mBitmapWidth * 4byte(ARGB_8888)；
 Native 方法中，

mBitmapWidth = mOriginalWidth * (scale = opts.inTargetDensity / opts.inDensity) * 1/inSampleSize，
 mBitmapHeight = mOriginalHeight * (scale = opts.inTargetDensity / opts.inDensity) * 1/inSampleSize。

现在针对介绍的几种场景，会得到这样的结论：

1. 将一张 720x1080图片放在 drawable-xhdpi 目录下（inDensity = 320）， 
   - 在 720x1080 手机上加载（inTargetDensity = 320），图片不会被压缩；
   - 在 480x800 手机上加载（inTargetDensity = 240），图片会被压缩 9/16；
   - 在 1080x1920 手机上加载（inTargetDensity = 480），图片会被放大 2.25；
2. 切不通分辨率大小的图片放到对应文件夹下，会根据屏幕获取对应文件夹的图片，就不存在加载图片时压缩和放大（针对标准屏）;

*注意，上述计算方式是在通过 decodeResource() 方法获取 Bitmap 的情况下得出，其他几种方式获取Bitmap，最后得到占用内存Size不会跟资源文件目录相关联*

- 将一张分辨率为 720x1080 的图片放到 xxhdpi 或者 hdpi ，同放在 xhdpi 标准文件夹下，对于同一台手机占用内存大小是否有变化？

  手机屏幕大小 1280 x 720（inTarget = 320），加载 xxhdpi （inDensity = 480）中的图片 1920 x 1080，scale = 320 / 480，inSampleSize = 1，最终获得的 Bitmap 的图像大小是 ：
   mBitmapWidth = opts.outWidth = 1080 * (320 / 480) * 1/1 = 720，
   mBitmapHeight = opt.outHeight = 1920 * (320 / 480) * 1/1 = 1280，
   getAllocatedMemory() = mBitmapWidth * mBitmapHeight * 4 = Bitmap占用内存

  

- 使用 decodeResource()  和 decodeStream() 有什么区别？

（1）decodeResource() 流程，会先用 TypedValue 保存图片信息，然后会根据条件设置 opts.inDensity = value.inDensity，为0则设置为默认 160dpi； 文件夹代表密度
 Opts.inTargetDensity = getDisplayMetrics().densityDpi； 屏幕密度
 设置完上述参数后，最终还是会调用 decodeStream() 方法；

（2）decodeStream() native 方法得到 Bitmap后，调用 setDensityFromOptions() 方法来设置 Bitmap.mDensity:
 若 opts.inDensity != 0，bitmap.mDensity = opts.inDensity;
 若 opts.inTargetDensity != 0 && inDensity != targetDensity && inDensity != screenDensity，继续判断，如果 opts.inScaled || isNinePatch，bitmap.mDensity = targetDensity;

所以，
 （1）若使用 decodeResource() 加载本地图片，inDensity 为加载图片所在的文件夹代表的 dpi，inTargetDensity 为目标屏幕密度（or 图片真实像素密度？），
 最终 bitmap.mDensity = targetDensity。

（2）若使用 decodeStream() 则不会先记录图片信息，得到bitmap 后，直接调用 setDensityFromOptions() 方法，所以最终 bitmap.mDensity = defaultDensity() = DENSITY_DEVICE。