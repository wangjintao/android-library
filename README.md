# Android MapBox Library

这个库是基于MapBox SDK V11开发的地图管理工具，简化使用流程

## 集成方法

### 1. 在app的res/values/string.xml里添加MapBox的Token

```
<string name="mapbox_access_token">你的token</string>
```

****这个必须加，否则MapBox的SDK会报错导致APP崩溃****

### 2. 在`settings.gradle.kts`里加MapBox和本library的仓库地址

```
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        //注意：MapBox的仓库地址也要加上，没托管在MavenCenter中央仓库
         maven {
            url = uri("https://api.mapbox.com/downloads/v2/releases/maven")
        }

        // 👇 加在这里
        maven {
            url = uri("https://raw.githubusercontent.com/wangjintao/android-library/main")
        }
    }
}
```

### 3. 在`build.gradle.kts`里写依赖

```agsl
dependencies {
    implementation("com.aiforcetech:maplib:${版本号}")
}
```

## 使用方法

### 1. 在显示地图的Activity/Fragment里初始化 MapBox

```agsl
private val mapBox: MapBox by lazy { MapBox() }
```

### 2. 在`onCreate`中调用`mapBox.initMapBox`初始化

```
/**
     * @param mapView 布局中的 MapView
     * @param config 地图的一些显示配置
     * @param scope 当前Activity/Fragment的LifecycleScope，后边点击查询对象在子线程中查找用
     * @param onMapClickListener 点击事件
     * @param finished 初始化结束的回调
     */
    fun initMapBox(mapView: MapView, config: MapConfig = MapConfig(), scope: CoroutineScope,
        onMapClickListener: ((MapClickResult) -> Unit)? = null, finished: () -> Unit = {}) 
```

初始化完成后就可以通过`mapBox`使用了

### 3. 在`onDestroy()`中调用`mMapBox.release()`释放资源

```kotlin
 override fun onDestroy() {
        super.onDestroy()
        mMapBox.release()
    }
```

## 支持的功能

目前支持在地图上操作以下几种元素：图片（ImageOptElement），圆点（CircleOptElement），线（LineOptElement），多边形（PolygonOptElement）。但是多边形里边可能有一些形状要显示，这里命名为障碍物（PolygonObstacleElement），
障碍物是依赖于多边形存在的。障碍物分为两类，分别是圆形障碍物（ObstacleCircle）和多边形障碍物（ObstaclePolygon）。其中障碍物都是实心的形状

### 1. 显示地图

```kotlin
mapBox.initMapBox(mViewBinding.mapView, scope = lifecycleScope)
```

没传MapConfig，显示默认样式的地图，无指南针，无比例尺，不显示logo，禁止旋转

![图片](./images/show_map.png)


### 2. 移动视角和缩放
移动地图的视角和缩放比例提供了两种方法`cameraTo`和`flyTo`，两者的区别是一个有动画一个无动画
```kotlin
    /**
    * 不带动画显示地图区域
    * @param center 移动到的中心点，只进行缩放的时候可不传
    * @param zoom 地图缩放的级别，只移动显示区域时可传-1.0
    */
    fun cameraTo(center: Point? = null, zoom: Double = -1.0)
```
```kotlin
    /**
     * 带动画移动地图显示区域
     * @param center 移动到的中心点，只进行缩放的时候可不传
     * @param zoom 地图缩放的级别，只移动显示区域时可传-1.0
     * @param duration 动画执行时间，默认5000毫秒
     */
    fun flyTo(center: Point? = null, zoom: Double = -1.0, duration: Long = 5000) 
```
![图片](./images/move_camera.gif)

### 3. 添加图片
```kotlin
    /**
     * @param point 图片要添加的位置
     * @param bitmap 要显示的图片
     * @param scale 对图片缩放比例，默认为1.0，不缩放
     *
     * @return 返回添加的这个图片，后续可对其进行修改、删除操作
     */
    fun addImage(point: Point, bitmap: Bitmap, scale: Double = 1.0): OptElement.ImageOptElement?
```


### 4. 修改图片
* #### 修改图片位置
```kotlin
    /**
     * @param element 要修改的元素
     * @param newPosition 要更新到的位置
     */
    fun updateImage(element: OptElement.ImageOptElement, newPosition: Point)
```
* #### 修改图片大小
```kotlin
    /**
     * @param element 要修改的元素
     * @param newSize 新的大小
     */
    fun updateImageSize(element: OptElement.ImageOptElement, newSize: Double)
```
### 5. 删除元素
```kotlin
/**
     * @param element 要删除的元素
     */
    fun removeElement(element: OptElement?)
```

### 6. 添加圆点

* #### 添加一个圆点
```kotlin
    /**
     * 添加一个圆点
     * @param point 圆点所在位置
     * @param color 圆点颜色
     * @param radius 圆点半径
     * @return 添加的这个圆点，后续可对其进行其他操作
     */
    fun addCircle(point: Point, color: Int = Color.BLUE, radius: Double = 5.0): OptElement.CircleOptElement?
```
* #### 添加多个圆点
```kotlin
    /**
     * 添加多个圆点
     * @param points 圆点所在位置集合
     * @param color 圆点颜色
     * @param radius 圆点颜色
     * @return 所添加圆点的集合
     */
    fun addCircles(points: List<Point>, color: Int = Color.BLUE,
        radius: Double = 5.0): MutableList<OptElement.CircleOptElement>
```
### 7. 修改圆点
修改圆点提供了3个方法，分别是修改位置，半径和颜色
* #### 修改位置
```kotlin
    /**
     * 修改圆点的位置
     * @param element 要修改的圆点
     * @param newPosition 新位置
     */
    fun updateCirclePosition(element: OptElement.CircleOptElement, newPosition: Point)
```
* #### 修改半径
```kotlin
    /**
     * 修改圆点半径
     * @param element 要修改的圆点
     * @param newRadius 新半径
     */
    fun updateCircleRadius(element: OptElement.CircleOptElement, newRadius: Double) 
```
* #### 修改颜色
```kotlin
    /**
     * 修改圆点颜色
     * @param element 要修改的圆点
     * @param newColor 新颜色
     */
    fun updateCircleColor(element: OptElement.CircleOptElement, newColor: Int)
```
### 8.添加线
* #### 添加一条线
```kotlin
    /**
     * 添加一条线
     * @param points 要添加的线的点集
     * @param color 要添加的线的颜色
     * @param width 要添加的线的宽度
     * @return 所添加的线，方便后续对其进行操作
     */
    fun addLine(points: List<Point>, color: Int = Color.BLUE, width: Double = 3.0): OptElement.LineOptElement? 
```
* #### 添加多条线
```kotlin
    /**
     * 添加多条线
     * @param lines 要添加的多条线的集合
     * @param color 线的颜色
     * @param width 线的宽度
     * @return 所添加的线的集合
     */
    fun addLines(lines: List<List<Point>>, color: Int = Color.BLUE,
        width: Double = 3.0): MutableList<OptElement.LineOptElement> 
```

### 9.线后加点（延长）
```kotlin
   /**
     * 在线后加一个点（线变长）
     * @param element 操作的线
     * @param newPoint 新添加的点
     */
    fun appendPoint(element: OptElement.LineOptElement, newPoint: Point)
```
### 10.修改线的样式（颜色和粗细）
```kotlin
    /**
     * 修改线的样式（颜色和宽度）
     * @param element 要修改的线
     * @param color 修改成的颜色
     * @param width 修改成的宽度
     */
    fun updateLineStyle(element: OptElement.LineOptElement, color: Int? = null, width: Double? = null) 
```


## 部分参数说明

### 1.  `MapConfig`

```kotlin
data class MapConfig(
    var showCompass: Boolean = false,
    var scaleBarPosition: Int = -1,
    var showLogo: Boolean = false,
    var rotateEnabled: Boolean = false,
)
```

showCompass-是否显示指南针（MapBox默认正上方为北的时候不显示）<br>
scaleBarPosition-是否显示比例尺<br>
showLogo-是否显示MapBox的logo<br>
rotateEnabled-是否能旋转地图<br>

### 2. `MapClickResult`

```kotlin
data class MapClickResult(
    val point: Point?,           // 点击位置
    val element: OptElement?,    // 命中的元素
    val feature: Feature?     // 原始Feature（调试/扩展用）
)
```

### 3.`ObstacleCircle`

```kotlin
data class ObstacleCircle(
    val center: Point,//中心点
    val diameter: Double,//直径
    val color: Int//颜色
) : Obstacle()
```

### 4. `ObstaclePolygon`

```kotlin
data class ObstaclePolygon(
    val points: List<Point>,//外圈点集
    val color: Int//颜色
) : Obstacle()
```
