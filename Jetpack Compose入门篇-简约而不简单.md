## Jetpack Compose入门篇-简约而不简单

### Compose简介

- Jetpack Compose：利用声明式编程构建Android原生界面（UI）的 工具包

### 优势

- 更少的代码、代码量锐减
- 强大的工具/组件支持
- 直观的 Kotlin API
- 简单易用

### Compose 编程思想

- 声明性编程范式：声明性的函数构建一个简单的界面组件，无需修改任何 XML 布局，也不需要使用布局编辑器，只需要调用 Jetpack Compose 函数来声明想要的元素，Compose 编译器即会完成后面的所有工作

- 举个栗子：简单的可组合函数

  ```kotlin
  class MainActivity : ComponentActivity() {
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContent {
              Text("Hello world!")
          }
      }
  }
  ```

<img src="http://p0.qhimg.com/t0122be6f534c404bc5.png" alt="image-20210831180043978" style="zoom:60%;" />

- 动态 ：组合函数是用 Kotlin 而不是 XML 编写，见上`$name` 的传入

- 需要注意的事项：
  - 可组合函数可以按任何顺序执行

    ```kotlin
    //可以按任何顺序进行,不能让 StartScreen() 设置某个全局变量（附带效应）并让 MiddleScreen() 利用这项更改。相反，其中每个函数都需要保持独立。
    @Composable
    fun ButtonRow() {
        MyFancyNavigation {
            StartScreen()
            MiddleScreen()
            EndScreen()
        }
    }
    ```

  - 可组合函数可以并行执行

    - Compose 可以通过并行运行可组合函数来优化重组，这样一来，Compose 就可以利用多个核心，并以较低的优先级运行可组合函数（不在屏幕上）

    - 这种优化意味着，可组合函数可能会在后台线程池中执行，如果某个可组合函数对 ViewModel 调用一个函数，则 Compose 可能会同时从多个线程调用该函数
    - 调用某个可组合函数时，调用可能发生在与调用方不同的线程上，这意味着，应避免使用修改可组合 lambda 中的变量的代码，既因为此类代码并非线程安全代码，又因为它是可组合 lambda 不允许的附带效应

    ```kotlin
    //此代码没有附带效应
    @Composable
    fun ListComposable(myList: List<String>) {
        Row(horizontalArrangement = Arrangement.SpaceBetween) {
            Column {
                for (item in myList) {
                    Text("Item: $item")
                }
            }
            Text("Count: ${myList.size}")
        }
    }
    ```

    ```kotlin
    //如果函数写入局部变量，则这并非线程安全或正确的代码：
    @Composable
    @Deprecated("Example with bug 有问题的代码")
    fun ListWithBug(myList: List<String>) {
        var items = 0
    
        Row(horizontalArrangement = Arrangement.SpaceBetween) {
            Column {
                for (item in myList) {
                    Text("Item: $item")
                    items++ // Avoid! Side-effect of the column recomposing.
                }
            }
            Text("Count: $items")
        }
    }
    //每次重组时，都会修改 items。这可以是动画的每一帧，或是在列表更新时。但不管怎样，界面都会显示错误的项数。因此，Compose 不支持这样的写入操作；通过禁止此类写入操作，我们允许框架更改线程以执行可组合 lambda。
    ```

  - 重组会跳过尽可能多的 可组合函数和 lambda

  - 重组是乐观的操作，可能会被取消

  - 可组合函数可能会像动画的每一帧一样非常频繁地运行

### 环境准备

- 已了解的同学，可直接跳过

- 需要升级到Arctic Fox 2020-3-1 版本以上，此版本以下Android studio 无此支持-[【下载最新Android studio】](https://developer.android.com/studio)

  <img src="http://p0.qhimg.com/t0103f18682c8eb95a9.png" alt="ComposeActivititysupport" style="zoom:50%;" />

- 我们注意到此项目只支持Kotlin 最低sdk 版本为21，Android 5.0

  <img src="http://p0.qhimg.com/t0111390156e50e462f.png" alt="support01" style="zoom:50%;" />

- Gradle Compose相关依赖

```groovy
implementation "androidx.compose.ui:ui:$compose_version"
    implementation "androidx.compose.material:material:$compose_version"
    implementation "androidx.compose.ui:ui-tooling-preview:$compose_version"
    implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.3.1'
    implementation 'androidx.activity:activity-compose:1.3.0-alpha06'
    testImplementation 'junit:junit:4.+'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
    androidTestImplementation "androidx.compose.ui:ui-test-junit4:$compose_version"
    debugImplementation "androidx.compose.ui:ui-tooling:$compose_version"
```

```groovy
 kotlinOptions {
        jvmTarget = '1.8'
        useIR = true//在 Gradle 构建脚本中指定额外编译器选项即可启用新的 JVM IR 后端
    }
composeOptions {
        kotlinCompilerExtensionVersion compose_version
        kotlinCompilerVersion '1.5.10'
    }
buildFeatures {
        compose true
    }
packagingOptions {
        resources {
            excludes += '/META-INF/{AL2.0,LGPL2.1}'
        }
    }
```

- 由于新版本邀请java 11，安装 java 8 环境的需要以下修复

```groovy
Android Gradle plugin requires Java 11 to run. You are currently using Java 1.8.
     You can try some of the following options:
       - changing the IDE settings.
       - changing the JAVA_HOME environment variable.
       - changing `org.gradle.java.home` in `gradle.properties`.      
```

```groovy
org.gradle.java.home=/Library/Java/JavaVirtualMachines/jdk-11.0.11.jdk/Contents/Home
```

<img src="http://p0.qhimg.com/t01eb6f1c0661656b49.png" alt="image-20210831152915085" style="zoom: 33%;" />

- @Preview起作用，环境正常![image-20210831152638422](http://p0.qhimg.com/t019834eddb2afd1570.png)

### 布局

- Android 传统从xml-状态的变更关系，程序员需要大量的代码维护Ui 界面，以达到界面状态的正确性，费时费力，即便是借助MVVM架构，一样需要维护状态，因为布局只有一套

  ![image-20210831162447670](http://p0.qhimg.com/t0142cb4cc360fb2183.png)

- 声明式与传统XML 实现区别，Compose 声明式布局，是直接重建了UI，所以不会有状态问题

  <img src="http://p0.qhimg.com/t011cecaa35ea292d96.png" alt="image-20210831162854191" style="zoom:50%;" />

  <img src="/Users/caining/Library/Application Support/typora-user-images/image-20210831162914660.png" alt="image-20210831162914660" style="zoom:50%;" />

- **Text**：Compose 提供了基础的 `BasicText` 和 `BasicTextField`，它们是用于显示文字以及处理用户输入的主要函数。Compose 还提供了更高级的 `Text` 和 `TextField`

  ```kotlin
  Text("Hello World")
  ```

- **重组Text->Button**

  ```kotlin
  @Composable
  fun ClickCounter(clicks: Int, onClick: () -> Unit) {
      Button(onClick = onClick) {
          Text("I've been clicked $clicks times")
      }
  }
  ```

- **Modifier**可以修改控件的位置、高度、边距、对齐方式等等

  ```kotlin
  //`padding` 设置各个UI的padding。padding的重载的方法一共有四个。
  Modifier.padding(10.dp) // 给上下左右设置成同一个值
  Modifier.padding(10.dp, 11.dp, 12.dp, 13.dp) // 分别为上下左右设值
  Modifier.padding(10.dp, 11.dp) // 分别为上下和左右设值
  Modifier.padding(InnerPadding(10.dp, 11.dp, 12.dp, 13.dp))// 分别为上下左右设值
  //这里设置的值必须为`Dp`，`Compose`为我们在Int中扩展了一个方法`dp`，帮我们转换成`Dp`。
  //`plus` 可以把其他的Modifier加入到当前的Modifier中。
  Modifier.plus(otherModifier) // 把otherModifier的信息加入到现有的modifier中
  //`fillMaxHeight`,`fillMaxWidth`,`fillMaxSize` 类似于`match_parent`,填充整个父layout。
  Modifier.fillMaxHeight() // 填充整个高度
  //`width`,`heigh`,`size` 设置Content的宽度和高度。
  Modifier.width(2.dp) // 设置宽度
  Modifier.height(3.dp)  // 设置高度
  Modifier.size(4.dp, 5.dp) // 设置高度和宽度
  //`widthIn`, `heightIn`, `sizeIn` 设置Content的宽度和高度的最大值和最小值。
  Modifier.widthIn(2.dp) // 设置最大宽度
  Modifier.heightIn(3.dp) // 设置最大高度
  Modifier.sizeIn(4.dp, 5.dp, 6.dp, 7.dp) // 设置最大最小的宽度和高度
  //`gravity` 在`Column`中元素的位置。
  Modifier.gravity(Alignment.CenterHorizontally) // 横向居中
  Modifier.gravity(Alignment.Start) // 横向居左
  Modifier.gravity(Alignment.End) // 横向居右
  //`rtl`, `ltr` 开始布局UI的方向。
  Modifier.rtl  // 从右到左
  //更多Modifier学习：https://developer.android.com/jetpack/compose/modifiers-list
  ```

- `Column` 线性布局≈ `Android LinearLayout-VERTICAL` 

- `Row` 水平布局≈`Android LinearLayout-HORIZONTAL`

- `Box`帧布局≈`Android FrameLayout`，可将一个元素放在另一个元素上，如需在 `Row` 中设置子项的位置，请设置 `horizontalArrangement` 和 `verticalAlignment` 参数。对于 `Column`，请设置 `verticalArrangement` 和 `horizontalAlignment` 参数

- 相对布局，需要引入  `ConstraintLayout`

  - 引入

  ```groovy
  implementation "androidx.constraintlayout:constraintlayout-compose:1.0.0-beta02"
  ```

  - `constraintlayout-compose`用法流程

  <img src="http://p0.qhimg.com/t015842a53addb7d63c.png" alt="graphics-drawcircle" style="zoom:100%;" />

  - 完整用法示例

  ```kotlin
  @Composable
  fun testConstraintLayout() {
      ConstraintLayout() {
          //通过createRefs创建三个引用
          val (imageRef, nameRef) = createRefs()
          Image(painter = painterResource(id = R.mipmap.test),
              contentDescription = "图",
              modifier = Modifier
                  .constrainAs(imageRef) {//通过constrainAs将Image与imageRef绑定,并增加约束
                      top.linkTo(parent.top)
                      start.linkTo(parent.start)
                      bottom.linkTo(parent.bottom)
                  }
                  .size(100.dp)
                  .clip(shape = RoundedCornerShape(5)),
              contentScale = ContentScale.Crop)
          Text(
              text = "名称",
              modifier = Modifier
                  .constrainAs(nameRef) {
                      top.linkTo(imageRef.top, 2.dp)
                      start.linkTo(imageRef.end, 12.dp)
                      end.linkTo(parent.end)
                      width = Dimension.fillToConstraints
                  }
                  .fillMaxWidth(),
              fontSize = 18.sp,
              maxLines = 1,
              textAlign = TextAlign.Left,
              overflow = TextOverflow.Ellipsis,
          )
      }
  }
  ```

### 列表

- 可以滚动的布局

```kotlin
//我们可以使用 verticalScroll() 修饰符使 Column 可滚动
Column (
        modifier = Modifier.verticalScroll(rememberScrollState())){
        messages.forEach { message ->
            MessageRow(message)
        }
    }
```

- 但以上布局并无法实现重用，可能导致性能问题，下面介绍我们重点布局，列表

- `LazyColumn/LazyRow==RecylerView/listView`  列表布局，解决了滚动时的性能问题，`LazyColumn` 和 `LazyRow` 之间的区别就在于它们的列表项布局和滚动方向不同

  - 内边距

    ```kotlin
    LazyColumn(
        contentPadding = PaddingValues(horizontal = 16.dp, vertical = 8.dp),
    ) {
        // ...
    }
    
    ```

  - item间距

    ```kotlin
    LazyColumn(
        verticalArrangement = Arrangement.spacedBy(4.dp),
    ) {
        // ...
    }
    ```

  - 浮动列表的浮动标题，使用 `LazyColumn` 实现粘性标题，可以使用实验性 `stickyHeader()`函数

    ```kotlin
    @OptIn(ExperimentalFoundationApi::class)
    @Composable
    fun ListWithHeader(items: List<Item>) {
        LazyColumn {
            stickyHeader {
                Header()
            }
    
            items(items) { item ->
                ItemRow(item)
            }
        }
    }
    ```

- 网格布局`LazyVerticalGrid` 

  ```kotlin
  @OptIn(ExperimentalFoundationApi::class)
  @Composable
  fun PhotoGrid(photos: List<Photo>) {
      LazyVerticalGrid(
          cells = GridCells.Adaptive(minSize = 128.dp)
      ) {
          items(photos) { photo ->
              PhotoItem(photo)
          }
      }
  }
  ```

### 自定义布局

- 通过重组基础布局实现

- Canvas

  ```kotlin
  Canvas(modifier = Modifier.fillMaxSize()) {
      val canvasWidth = size.width
      val canvasHeight = size.height
      drawCircle(
          color = Color.Blue,
          center = Offset(x = canvasWidth / 2, y = canvasHeight / 2),
          radius = size.minDimension / 4
      )
  }
  //drawCircle 画圆
  //drawRectangle 画矩形
  //drawLine //画线
  ```

<img src="http://p0.qhimg.com/t016c9e51a22db27de0.png" alt="graphics-drawcircle" style="zoom:20%;" />

### 动画

- 动画Api 选择

<img src="http://p0.qhimg.com/t0157e4e0cae28d912f.png" alt="animation-flowchart" style="zoom:50%;" />

### 其他库支持

- [导航栏](https://developer.android.com/jetpack/compose/navigation)

  ```groovy
  implementation("androidx.navigation:navigation-compose:2.4.0-alpha05")
  ```

### 总结

- Compose总体来说，对于Android-Native布局实现上更加简单高效，值得大家一学
- Compose 写法与Flutter-Dart 有高度类似的情况，后面我们可以做一篇与Flutter-Dart 语音写布局的一些对比

### 引用

- https://developer.android.com/jetpack/compose
