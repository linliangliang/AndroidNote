<center>Android 屏幕适配方案</center>

**重要概念：**  
android中的dp在渲染前会将dp转为px，计算公式：  
px = density * dp;  
density = dpi / 160;  
px = dp * (dpi / 160); 

常见设备的dp、px、density的关系
| 1 | 分辨率 | density | dpi |
| ---- | ---- | ---- | ---- |
| hdpi | 480 * 800 | 1.5 | 160dpi~**240**dpi |
| xhdpi | 720 * 1280 | 2.0 | 240dpi~**320**dpi |
| xxhdpi | 1080 * 1920 | 3.0 | 320dpi~**480**dpi |
| xxxhdpi | 1440 * 2560 | 4.0 | 480dpi~**640**dpi |


## 1、添加不同布局,diments适配屏幕
1、手机针对不同分辨率的设备添加不同的资源到xxhdpi下面，

创建多个不同的 values文件夹,在下面添加dimens.xml文件  
values-800\*480  
values-1184\*720  
values-1280\*720  

2、大屏设备，如平板：  
分辨率布局：layout-sw480dp layout-sw600dp-land layout-sw720dp-port

事实上,sw不是software的，意思，我猜是shortest width，最短的宽度。  
**sw-xxxx-dp的计算公式是  sw\*160/dpi**  
比如&emsp;1280\*800, sw 是800  
&emsp;&emsp;&emsp;480\*800， sw 是480  


| 机型 | 分辨率 | sw | dpi=ro.sf.lcd_density|sw *160/dpi=dp|
|---------|----------|---------|---------|---------|
| H7(Android5.1) | 1280\*720 | 720| 213 | 720\*160/213=540.84=sw340dp |
| H7(Android4.4) |1280\*720 | 720 | 160 | 720\*160/160=720=sw720dp |
| A33 astar-y2 | 1024\*768 | 768 | 160 | 768\*160/160 =768=sw720dp |

## 2、修改全局density 适配不同屏幕
&emsp;&emsp;美工给我们设计图大部分公司的给的是px单位的，并且只给一套UI图，我们需要适配屏幕分辨率各种各样的手机，这里推荐一个UI和开发方便沟通平台：https://lanhuapp.com/ UI给我们的设计图是一个固定值，我们需要以屏幕的宽最为参考，计算一个比例，然后将计算得到的density设置给Activity，注意在setContentView之前设置。  
&emsp;&emsp;我们将修改Density的方法抽成工具类，需要注意的是当我们在系统中修改系统字体大小后，系统的scaledDensity会发生改变，因此我们需要监听用户修改系统字体，然后重新设置scaledDensity，代码很简单，直接上工具类。


```java
public class EDensityUtils {
​
    //    private static final float  WIDTH = 480;//参考设备的宽，单位是dp DPI:640
    //    private static final float  WIDTH = 640;//参考设备的宽，单位是dp DPI:480
    //    private static final float  WIDTH = 960;//参考设备的宽，单位是dp DPI:320
    private static final float  WIDTH = 1920;//参考设备的宽，单位是dp DPI:160时
    private static float appDensity;//表示屏幕密度
    private static float appScaleDensity; //字体缩放比例，默认appDensity
​
    private EDensityUtils() {
        throw new UnsupportedOperationException("you can't instantiate EDensityUtils...");
    }

    public static void setDensity(final Application application, Activity activity){
        //获取当前app的屏幕显示信息
        DisplayMetrics displayMetrics = application.getResources().getDisplayMetrics();
        if (appDensity == 0){
            //初始化赋值操作
            appDensity = displayMetrics.density;
            appScaleDensity = displayMetrics.scaledDensity;
 
            //添加字体变化监听回调
            application.registerComponentCallbacks(new ComponentCallbacks() {
                @Override
                public void onConfigurationChanged(Configuration newConfig) {
                    //字体发生更改，重新对scaleDensity进行赋值
                    if (newConfig != null && newConfig.fontScale > 0){
                        appScaleDensity = application.getResources().getDisplayMetrics().scaledDensity;
                    }
                }
 
                @Override
                public void onLowMemory() {
 
                }
            });
        }
 
        //计算目标值density, scaleDensity, densityDpi
        float targetDensity = displayMetrics.widthPixels / WIDTH; // 1920 / 1920 = 1.0
        float targetScaleDensity = targetDensity * (appScaleDensity / appDensity);
        int targetDensityDpi = (int) (targetDensity * 160);
 
        //替换Activity的density, scaleDensity, densityDpi
        DisplayMetrics dm = activity.getResources().getDisplayMetrics();
        dm.density = targetDensity;
        dm.scaledDensity = targetScaleDensity;
        dm.densityDpi = targetDensityDpi;
    }
 
}
```
使用也十分简单，只需要在BaseActivity的onCreate方法中调用setDensity方法即可，注意的是应该在setContentView之前设置。  
```java
public class BaseActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EDensityUtils.setDensity(getApplication(),this);
    }
}
```