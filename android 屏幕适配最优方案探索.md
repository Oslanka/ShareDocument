## android 屏幕适配最优方案探索-三种屏幕适配方案对比

https://developer.android.com/guide/practices/screens_support

为什么要做适配

1.ui 设计稿只有一套

2.手机屏幕分辨率千千万万

3.一套尺寸无法满足

那么问题来了。。。如何屏幕适配呢？

最简单的 px 适配 生成多套尺寸

![image-20210312154839729](/Users/caining/Library/Application Support/typora-user-images/image-20210312154839729.png)



### 1. 基础概念

【屏幕尺寸】：物理尺寸 单位是inch英寸，对角线长度；

【屏幕分辨率】：手机在横向、纵向上的像素点数总和，例如 1080x1920

【屏幕像素密度】：每英寸像素点数，单位：dpi，dpi为DPI是Dots Per Inch，与ppi区别 Pixels Per Inch也叫像素密度；；

### 2. dpi vs ppi

dots per inch Vs pixels per inch 

区别1，ppi常用于ui设计 dpi常用于平面印刷

区别2 ： android 这边用dpi & iOS 用ppi

（这里不深究 点 与像素区别别，是不是一个dot=一个pixel 这跟物理设备有关，有兴趣同学自己研究一下）这里姑且轮dpi==ppi

### 3. 结论公式dpi（ppi）公式

$$
【公式一:通用算法】 dpi(ppi) = \frac{分辨率}{inch}
$$



区别  

公式参考：

http://www.360doc.com/content/17/0107/17/14106735_620766397.shtml

### 

$$
勾股定理：    a^2+b^2=c^2
$$

$$
【公式二:手机对角线】 dpi(ppi)=\frac{\sqrt{长度像素^2+宽度像素^2}}{对角inch}
$$


- <img src="/Users/caining/Documents/share/seinfo.png" style="zoom:80%;" />

  我们从上面提炼出来 关键信息  1.ppi=326 2.分辨率1334*750 3.对角尺寸是4.7英寸

  我们验证一下dpi ppi 公式

- $$
  dpi(ppi)=\frac{\sqrt{1334^2+750^2}}{4.7}=325.6122832234166
  $$


如果电脑屏幕是2K分辨率，即1920×1080像素，它的图像宽为1920像素。而如果这个电脑屏幕的物理宽度是19.2英寸，电脑屏幕是分辨率就是1920/19.2=100PPI

***density解析：density在每个设备上都是固定的，density=dpi/160，意思就是1dp占多少像素，这个值是可以在代码中改变的\***

头条方案



**px = density \* dp**
dp : 安卓开发人员常常挂在嘴上的长度单位
px : 设计人员眼中的长度单位
**density = dpi / 160**
因此，**px = dp \* (dpi/160)**
dpi : 根据屏幕真实分辨率和尺寸计算得出
举个例子：屏幕分辨率为 1920 *1080，屏幕尺寸为5寸(屏幕斜边长度cm/0.3937), 则 \*dpi = √(宽度²+ 高度²)/屏幕尺寸*



头条 链接   

https://mp.weixin.qq.com/s/d9QCoBP6kV9VSWvVldVVwA





![图片](https://mmbiz.qpic.cn/mmbiz_jpg/5EcwYhllQOgM19n6iawpWQRCfcibxicoBYGhxiapFRVjOtiaWzcERXwjaRDJgyyoIibSq2AJrby8q2aExttHeZfk0VZQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 1x屏幕=160dpi=mdpi，（mi）

 2x屏幕=160dpi=hdpi，（High）

 3x屏幕=160dpi=xhdpi

 4x屏幕=160dpi=xxhdpi

![image-20210312155929343](/Users/caining/Library/Application Support/typora-user-images/image-20210312155929343.png)

```
36x36 (0.75x) - 低密度 (ldpi)
48x48（1.0x 基准）- 中密度 (mdpi)
72x72 (1.5x) - 高密度 (hdpi)
96x96 (2.0x) - 超高密度 (xhdpi)
144x144 (3.0x) - 超超高密度 (xxhdpi)
192x192 (4.0x) - 超超超高密度 (xxxhdpi)
//        ldpi  适用于低密度 (ldpi) 屏幕 (~ 120dpi) 的资源。
//        mdpi 适用于中密度 (mdpi) 屏幕 (~ 160dpi) 的资源（这是基准密度）。
//        hdpi 适用于高密度 (hdpi) 屏幕 (~ 240dpi) 的资源。
//        xhdpi    适用于加高 (xhdpi) 密度屏幕 (~ 320dpi) 的资源。
```

4.dp Density-independent pixel (dp)独立像素密度。标准是160dip.即1dp对应1个pixel，计算公式如：px = dp * (dpi / 160)，屏幕密度越大，1dp对应 的像素点越多。

```java
 // 系统的Density
 private static float sNoncompatDensity;
 // 系统的ScaledDensity
 private static float sNoncompatScaledDensity;

 public static void setCustomDensity(Activity activity, Application application) {
        DisplayMetrics displayMetrics = application.getResources().getDisplayMetrics();
        if (sNoncompatDensity == 0) {
            sNoncompatDensity = displayMetrics.density;
            sNoncompatScaledDensity = displayMetrics.scaledDensity;
            // 监听在系统设置中切换字体
            application.registerComponentCallbacks(new ComponentCallbacks() {
                @Override
                public void onConfigurationChanged(Configuration newConfig) {
                    if (newConfig != null && newConfig.fontScale > 0) {
                        sNoncompatScaledDensity=application.getResources().getDisplayMetrics().scaledDensity;
                    }
                }

                @Override
                public void onLowMemory() {

                }
            });
        }
        // 此处以360dp的设计图作为例子
        float targetDensity=displayMetrics.widthPixels/360;
        float targetScaledDensity=targetDensity*(sNoncompatScaledDensity/sNoncompatDensity);
        int targetDensityDpi= (int) (160 * targetDensity);
        displayMetrics.density = targetDensity;
        displayMetrics.scaledDensity = targetScaledDensity;
        displayMetrics.densityDpi = targetDensityDpi;

        DisplayMetrics activityDisplayMetrics = activity.getResources().getDisplayMetrics();
        activityDisplayMetrics.density = targetDensity;
        activityDisplayMetrics.scaledDensity = targetScaledDensity;
        activityDisplayMetrics.densityDpi = targetDensityDpi;
    }
```



1. ### px

2. ### 头条适配 https://blog.csdn.net/wy391920778/article/details/81939233

3. https://github.com/JessYanCoding/AndroidAutoSize

   1. 

4. ### sw 适配











https://www.cnblogs.com/weekbo/p/9013388.html