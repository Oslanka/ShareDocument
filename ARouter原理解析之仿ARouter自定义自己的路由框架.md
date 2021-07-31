# ARouter原理解析之仿ARouter自定义自己的路由框架

### ARouter是什么？

ARouter是阿里开源的一款android路由框架，帮助 Android App 进行组件化改造的路由框架 —— 支持模块间的路由、通信、解耦；结合路由可以实现组件化。

<img src="https://www.hualigs.cn/image/610395ec18352.jpg" alt="WX20210730-113027@2x" style="zoom:50%;" />

### ARouter接入指北

[完整Arouter接入指南](https://github.com/alibaba/ARouter/blob/master/README_CN.md)，ARouter重度用户可以跳过，直接往后看

- 第一步，根build.gradle设置使用arouter-register

```groovy
apply plugin: 'com.alibaba.arouter'

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "com.alibaba:arouter-register:?"
    }
}
```

- 第二步，创建baselib，并加入dependencies

```groovy
api 'com.alibaba:arouter-api:x.x.x'
```

- 第三步，创建组件module，例如login 或者setting 组件

```groovy
android {
    defaultConfig {
        ...
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [AROUTER_MODULE_NAME: project.getName()]
            }
        }
    }
}

dependencies {
    // 替换成最新版本, 需要注意的是api
    // 要与compiler匹配使用，均使用最新版可以保证兼容
    //compile 'com.alibaba:arouter-api:x.x.x' 此移动到baselib中
    api project(path: ':baselib')
    annotationProcessor 'com.alibaba:arouter-compiler:x.x.x'
    ...
}

```

- 第四步，通过注解@Route 注册页面

```java
// 在支持路由的页面上添加注解(必选)
// 这里的路径需要注意的是至少需要有两级，/xx/xx
@Route(path = "/test/activity")
public class YourActivity extend Activity {
    ...
}
```

- 第五步，初始化

```java
if (isDebug()) {           // 这两行必须写在init之前，否则这些配置在init过程中将无效
    ARouter.openLog();     // 打印日志
    ARouter.openDebug();   // 开启调试模式(如果在InstantRun模式下运行，必须开启调试模式！线上版本需要关闭,否则有安全风险)
}
ARouter.init(mApplication); // 尽可能早，推荐在Application中初始化
```

- 第六步，使用ARouter

```java
ARouter.getInstance().build("/test/activity").navigation();
```

###  ARouter比传统Intent有哪些优点

- 传统intent的优点

  - 轻量
  - 简单

- 传统intent的缺点

  - 跳转过程无法控制，一旦调用了`startActivity(Intent)`便交由系统执行，中间过程无法插手
  - 跳转失败无法捕获、降级，出现问题直接抛出异常
  - 显示Intent中因为存在直接的类依赖关系，导致耦合严重

  ```java
  startActivity(new Intent(MainActivity.this, LoginActivity.class));//强依赖LoginActivity
  ```

  - 隐式Intent中会出现规则集中式的管理，导致协作困难，都需要在Manifest中进行配置，导致扩展性比较差

  ```java
  //隐式 比 显式更强一点，可以在两个无关子module 之间跳转，由于显式无法引入包，所以无法完成跳转
  Intent intent = new Intent();
  intent.setClassName(MainActivity.this,"com.cnn.loginplugin.ui.login.LoginActivity");//设置包路径
  startActivity(intent);
  ```

- ARouter优点

  - 模块间通信(后面讲原理)
  - 支持url 跳转 `build("/test/activity").navigation()`
  - 支持拦截器

  ```java
  // 比较经典的应用就是在跳转过程中处理登陆事件，这样就不需要在目标页重复做登陆检查
  // 拦截器会在跳转之间执行，多个拦截器会按优先级顺序依次执行
  @Interceptor(priority = 8, name = "测试用拦截器")
  public class TestInterceptor implements IInterceptor {
      @Override
      public void process(Postcard postcard, InterceptorCallback callback) {
      ...
      callback.onContinue(postcard);  // 处理完成，交还控制权
      // callback.onInterrupt(new RuntimeException("我觉得有点异常"));      // 觉得有问题，中断路由流程
  
      // 以上两种至少需要调用其中一种，否则不会继续路由
      }
  
      @Override
      public void init(Context context) {
      // 拦截器的初始化，会在sdk初始化的时候调用该方法，仅会调用一次
      }
  }
  ```

  - 参数注入，@Autowired注解实现，更方便，需要配合`ARouter.getInstance().inject(this);`一起使用

  ```java
  		@Autowired
      public String name;
      @Autowired
      int age;
      // 通过name来映射URL中的不同参数
      @Autowired(name = "girl") 
      boolean boy;
      // 支持解析自定义对象，URL中使用json传递
      @Autowired
      TestObj obj;
  // 使用 withObject 传递 List 和 Map 的实现了
      // Serializable 接口的实现类(ArrayList/HashMap)
      // 的时候，接收该对象的地方不能标注具体的实现类类型
      // 应仅标注为 List 或 Map，否则会影响序列化中类型
      // 的判断, 其他类似情况需要同样处理        
      @Autowired
      List<TestObj> list;
      @Autowired
      Map<String, List<TestObj>> map;
  ```

  - 支持外部url 跳转

  ```xml
  <activity android:name=".SchemeFilterActivity">
              <!-- Scheme -->
              <intent-filter>
                  <data
                      android:host="www.nativie.com"
                      android:scheme="arouter"/>
                  <action android:name="android.intent.action.VIEW"/>
                  <category android:name="android.intent.category.DEFAULT"/>
                  <category android:name="android.intent.category.BROWSABLE"/>
              </intent-filter>
  </activity>
  ```
  
  - 简单demo，github做简单静态界面服务器，并部署到https://oslanka.github.io/statichtml.github.io/，手机浏览器打开，并点击href实现html打通原生，按道理来说，所有未拦截的ARouter路径，均可被web浏览器跳转，如图：<img src="https://www.hualigs.cn/image/6103978d969d1.jpg" alt="WX20210730-141807@2x" style="zoom:50%;" />，html代码如下：
  
  ```html
  <html>
  <body>
  <p><a href="http://www.360.com/">测试跳转</a> </p>
  <p><a href="arouter://www.nativie.com/login/login">跳转登录android-ARouter</a></p>
  <p><a href="arouter://www.nativie.com/login/login?username=admin&password=123456">跳转登录android-ARouter 带参数</a></p>
  <p><a href="arouter://www.nativie.com/setting/setting">跳转android-ARouter 设置界面</a></p>
  <p><a href="arouter://www.nativie.com/web/web">跳转android-ARouter 设置界面</a></p>
  <p><a href="arouter://www.nativie.com/test/test">跳转android-ARouter 错误路径</a></p>
  </body>
  </html>
  ```

### 关于拦截器

- 拦截器(拦截跳转过程，面向切面编程)
- 什么是面向切面编程AOP？AOP为Aspect Oriented Programming的缩写，意为：[面向切面编程](https://baike.baidu.com/item/面向切面编程/6016335)，通过[预编译](https://baike.baidu.com/item/预编译/3191547)方式和运行期间动态代理实现程序功能的统一维护的一种技术。AOP是[OOP](https://baike.baidu.com/item/OOP)的延续，是软件开发中的一个热点，也是[Spring](https://baike.baidu.com/item/Spring)框架中的一个重要内容，是[函数式编程](https://baike.baidu.com/item/函数式编程/4035031)的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的[耦合度](https://baike.baidu.com/item/耦合度/2603938)降低，提高程序的可重用性，同时提高了开发的效率

```java
// 拦截器会在跳转之前执行，多个拦截器会按优先级顺序依次执行
@Interceptor(priority = 8, name = "测试用拦截器")
public class TestInterceptor implements IInterceptor {
    @Override
    public void process(Postcard postcard, InterceptorCallback callback) {
    ...
    callback.onContinue(postcard);  // 处理完成，交还控制权
    // callback.onInterrupt(new RuntimeException("我觉得有点异常"));      // 觉得有问题，中断路由流程

    // 以上两种至少需要调用其中一种，否则不会继续路由
    }

    @Override
    public void init(Context context) {
    // 拦截器的初始化，会在sdk初始化的时候调用该方法，仅会调用一次
    }
}
```

### 动态路由

- 动态注册路由信息 适用于部分插件化架构的App以及需要动态注册路由信息的场景，可以通过 ARouter 提供的接口实现动态注册 路由信息，目标页面和服务可以不标注 @Route 注解，**注意：同一批次仅允许相同 group 的路由信息注册**

```java
ARouter.getInstance().addRouteGroup(new IRouteGroup() {
        @Override
        public void loadInto(Map<String, RouteMeta> atlas) {
            atlas.put("/dynamic/activity",      // path
                RouteMeta.build(
                    RouteType.ACTIVITY,         // 路由信息
                    TestDynamicActivity.class,  // 目标的 Class
                    "/dynamic/activity",        // Path
                    "dynamic",                  // Group, 尽量保持和 path 的第一段相同
                    0,                          // 优先级，暂未使用
                    0                           // Extra，用于给页面打标
                )
            );
        }
    });
```

### ARouter详细API

```java

// 构建标准的路由请求，并指定分组
ARouter.getInstance().build("/home/main", "ap").navigation();
// 构建标准的路由请求，通过Uri直接解析
Uri uri;
ARouter.getInstance().build(uri).navigation();

// 构建标准的路由请求，startActivityForResult
// navigation的第一个参数必须是Activity，第二个参数则是RequestCode
ARouter.getInstance().build("/home/main", "ap").navigation(this, 5);

// 指定Flag
ARouter.getInstance()
    .build("/home/main")
    .withFlags();
    .navigation();

// 获取Fragment
Fragment fragment = (Fragment) ARouter.getInstance().build("/test/fragment").navigation();
                    
// 对象传递
ARouter.getInstance()
    .withObject("key", new TestObj("Jack", "Rose"))
    .navigation();

// 使用绿色通道(跳过所有的拦截器)
ARouter.getInstance().build("/home/main").greenChannel().navigation();

```

### 原理探索

- ARouter.init 时，通过获取`/data/app/包名/base.apk`来筛选出ARouter生成的类，如下图。

<img src="https://i0.hdslb.com/bfs/album/7aeb1191c49ec218a69ef4e754e9360f60b76628.png" alt="image-20210729163725845" style="zoom:50%;" />

- 对于Activity类型，跳转`ARouter.getInstance().build("/login/login").navigation();`，最终执行的是，如下：

```java
**
     * Start activity
     *
     * @see ActivityCompat
     */
    private void startActivity(int requestCode, Context currentContext, Intent intent, Postcard postcard, NavigationCallback callback) {
        if (requestCode >= 0) {  // Need start for result
            if (currentContext instanceof Activity) {//启动context 为Activity
                ActivityCompat.startActivityForResult((Activity) currentContext, intent, requestCode, postcard.getOptionsBundle());
            } else {
              // 启动context 为Application 时，不支持requestCode
                logger.warning(Consts.TAG, "Must use [navigation(activity, ...)] to support [startActivityForResult]");
            }
        } else {//启动context 为Application
            ActivityCompat.startActivity(currentContext, intent, postcard.getOptionsBundle());
        }

        if ((-1 != postcard.getEnterAnim() && -1 != postcard.getExitAnim()) && currentContext instanceof Activity) {    // Old version.
            ((Activity) currentContext).overridePendingTransition(postcard.getEnterAnim(), postcard.getExitAnim());
        }

        if (null != callback) { // Navigation over.
            callback.onArrival(postcard);
        }
    }
```

- 两个无关的module 如何跳转的呢？我们发现最终执行startActivity时，所用的context为Application，思路是这样的，子module启动另外无关子module时，将执行权，交还给主进程/主程序去处理

<img src="https://i0.hdslb.com/bfs/album/9fdc0573e616fa0f1d993f5cf4ad9414bbbfa754.png" alt="image-20210724170943112" style="zoom:50%;" />

- 打开生成路由文档,AROUTER_GENERATE_DOC="enable",会生成arouter-map-of-xx.json和3个java文件

```groovy
// 更新 build.gradle, 添加参数 AROUTER_GENERATE_DOC = enable
// 生成的文档路径 : build/generated/ap_generated_sources/(debug or release)/com/alibaba/android/arouter/docs/arouter-map-of-${moduleName}.json
android {
    defaultConfig {
        ...
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [AROUTER_MODULE_NAME: project.getName(), AROUTER_GENERATE_DOC: "enable"]
            }
        }
    }
}
//ARouter映射关系如何生成？Generated出三个文件
//ARouter$$Group$$login
//ARouter$$Providers$$loginplugin
//ARouter$$Root$$loginplugin
```

<img src="https://www.hualigs.cn/image/60fbcf1e3d1c5.jpg" alt="image-20210724163442843" style="zoom:50%;" />



```java
    atlas.put("/login/login", RouteMeta.build(RouteType.ACTIVITY, LoginActivity.class, "/login/login", "login", new java.util.HashMap<String, Integer>(){{put("password", 8); put("username", 8); }}, -1, -2147483648));

//map 存映射关系
//static Map<String, RouteMeta> routes = new HashMap<>();
```

- 以上三个文件是如何生成的呢？APT是Annotation Processing Tool的简称,即注解处理工具，apt是在编译期对代码中指定的注解进行解析，然后做一些其他处理（如通过javapoet生成新的Java文件）ARouter使用了两个库`auto-service` `javapoet`，来实现从注解到代码的注入，其中`auto-service`为注解处理器的库，`javapoet`为代码生成器

<img src="https://i0.hdslb.com/bfs/album/2d5e030f79cbea5654dadcf822d277e7bf2b5b36.png" alt="javaPoet" style="zoom: 67%;" />

### 通过例子了解APT

- 首先我们了解一下元注解，meta-annotation（元注解）

  -  @Target

    ```java
    TYPE, // 类、接口、枚举类 
    FIELD, // 成员变量（包括：枚举常量）
    METHOD, // 成员方法
    PARAMETER, // 方法参
    CONSTRUCTOR, // 构造方法
    LOCAL_VARIABLE, // 局部变量
    ANNOTATION_TYPE, // 注解类
    PACKAGE, // 可用于修饰：包
    TYPE_PARAMETER, // 类型参数，JDK 1.8 新增
    TYPE_USE // 使用类型的任何地方，JDK 1.8 新增
    ```

  -  @Retention

    ```java
    SOURCE,    只在本编译单元的编译过程中保留，并不写入Class文件中。
    CLASS,       在编译的过程中保留并且会写入Class文件中,但是JVM在加载类的时候不需要将其加载为运行时可见的（反射可见）的注解==是JVM在加载类时反射不可见。
    RUNTIME   在编译过程中保留，会写入Class文件，并且JVM加载类的时候也会将其加载为反射可见的注解。
    ```

  -  @Documented 注解的作用是：描述在使用 javadoc 工具为类生成帮助文档时是否要保留其注解信息.

  -  @Inherited 注解的作用是：使被它修饰的注解具有继承性（如果某个类使用了被@Inherited修饰的注解，则其子类将自动具有该注解）

- 通过元注解我们定义自己的注解

- [AutoService 注解处理器](https://github.com/google/auto/tree/master/service)

  ​		注解处理器是一个在javac中的，用来编译时扫描和处理的注解的工具。你可以为特定的注解，注册你自己的注解处理器。到这里，我假设你已经知道什么是注解，并且知道怎么申明的一个注解类型。

  一个注解的注解处理器，以Java代码（或者编译过的字节码）作为输入，生成文件（通常是.java文件）作为输出。

- 虚处理器`AbstractProcessor`
  - `init(ProcessingEnvironment env)`: 【核心】
    每一个注解处理器类都必须有一个空的构造函数。然而，这里有一个特殊的init()方法，它会被注解处理工具调用，并输入`ProcessingEnviroment`参数。`ProcessingEnviroment`提供很多有用的工具类`Elements`,`Types`和`Filer`
  - `process(Set< ? extends TypeElement> annotations, RoundEnvironment env)`:【核心】
    这相当于每个处理器的主函数main()。你在这里写你的扫描、评估和处理注解的代码，以及生成Java文件
  - `getSupportedAnnotationTypes()`
    这里你必须指定，这个注解处理器是注册给哪个注解的
  - `getSupportedSourceVersion()`
    用来指定你使用的Java版本。通常这里返回`SourceVersion.latestSupported()`

- APT 所用的代码生成器：**[JavaPoet](https://github.com/square/javapoet)** is a Java API for generating `.java` source files.（JavaPoet 是一个java api ，为了生成 .java源文件的）

- 官方helloworld

```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
    .returns(void.class)
    .addParameter(String[].class, "args")
    .addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(main)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.writeTo(System.out);
```

- 通过以上可生成以下java 文件

```java
package com.example.helloworld;

public final class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello, JavaPoet!");
  }
}
```

- `JavaPoet` 主要api

```java
- JavaFile 用于构造输出包含一个顶级类的Java文件 
- TypeSpec 生成类，接口，或者枚举  
- MethodSpec 生成构造函数或方法 
- FieldSpec 生成成员变量或字段 
- ParameterSpec  用来创建参数  
- AnnotationSpec 用来创建注解
```

- `JavaPoet` 主要占位符

```java
- $L(for Literals) 执行结构的字符或常见类型，或TypeSpec, $S(for Strings) 字符, $T(for Types) 类, $N(for Names) 方法 等标识符
  $L>$S
//1.Pass an argument value for each placeholder in the format string to `CodeBlock.add()`. In each example, we generate code to say "I ate 3 tacos"
CodeBlock.builder().add("I ate $L $L", 3, "tacos")
 //2.When generating the code above, we pass the hexDigit() method as an argument to the byteToHex() method using $N:
  MethodSpec byteToHex = MethodSpec.methodBuilder("byteToHex")
    .addParameter(int.class, "b")
    .returns(String.class)
    .addStatement("char[] result = new char[2]")
    .addStatement("result[0] = $N((b >>> 4) & 0xf)", hexDigit)
    .addStatement("result[1] = $N(b & 0xf)", hexDigit)
    .addStatement("return new String(result)")
    .build();
//=======================
public String byteToHex(int b) {
  char[] result = new char[2];
  result[0] = hexDigit((b >>> 4) & 0xf);
  result[1] = hexDigit(b & 0xf);
  return new String(result);
}

//$T for Types
//We Java programmers love our types: they make our code easier to understand. And JavaPoet is on board. It has rich built-in support for types, including automatic generation of import statements. Just use $T to reference types:
.addStatement("return new $T()", Date.class)== return new Date();
```

### 实战-自定义简易版路由-CRouter

- 新建name-annotation javaLib，定义CRoute注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.CLASS)
public @interface CRoute {
    String path();
}
```

- 新建name-compiler javaLib

```groovy
1.
dependencies {
    implementation project(path: ':TestRouter-annotation')
    annotationProcessor 'com.google.auto.service:auto-service:1.0-rc7'
    compileOnly 'com.google.auto.service:auto-service-annotations:1.0-rc7'

    implementation 'com.squareup:javapoet:1.8.0'
}
2.@AutoService(Processor.class)
public class TestRouteProcessor extends AbstractProcessor {
  @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
       //dosomething
    }
   @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
      //dosomething
    }
}
 
```

- 业务module执行顺序如下

```groovy
 1. annotationProcessor project(':TestRouter-compiler')
implementation project(':TestRouter-annotation')
2.添加注解@CRoute(path = "/csetting/csetting")
3.编译运行
4.业务module apt 生成的java 文件，如下：
public final class C$csettingC$csettingHelloWorld {
  public static String holder = "/csetting/csetting:com.cnn.settingplugin.SettingsActivity";

  public static void main(String[] args) {
    System.out.println("Hello, JavaPoet!");
  }
}
```

- 参考`ARouter-init` 方法，写出我们`CRouter-init`

<img src="https://www.hualigs.cn/image/610394704710e.jpg" alt="image-20210729165538859" style="zoom: 67%;" />

- 利用反射获取到注解对应映射关系，并参考ARouter存入HashMap

<img src="https://www.hualigs.cn/image/610394704a58f.jpg" alt="image-20210729172328967" style="zoom:67%;" />

- 通过隐式启动Activity模拟跳转

<img src="https://www.hualigs.cn/image/610394704cdb4.jpg" alt="image-20210729173011332" style="zoom:67%;" />

- 到此我们模拟出简易版本的ARouter，完整自定义CRouter

```java
/**
 * Created by caining on 7/29/21 16:09
 * E-Mail Address：cainingning@360.cn
 */
public class CRouter {
    private volatile static CRouter instance = null;
    private volatile static boolean hasInit = false;
    private static Application application;
    public static final String ROUTE_ROOT_PAKCAGE = "com.cnn.crouter";
    private static Map<String ,String> mapHolder = new HashMap<>();

    /**
     * Init, it must be call before used router.
     */
    public static void init(Application application) {
        if (!hasInit) {
            CRouter.application=application;
            hasInit=true;
            try {
                getFileNameByPackageName(application, ROUTE_ROOT_PAKCAGE);
            } catch (PackageManager.NameNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }

    /**
     * Get instance of router. A
     * All feature U use, will be starts here.
     */
    public static CRouter getInstance() {
        if (!hasInit) {
            throw new InitException("ARouter::Init::Invoke init(context) first!");
        } else {
            if (instance == null) {
                synchronized (CRouter.class) {
                    if (instance == null) {
                        instance = new CRouter();
                    }
                }
            }
            return instance;
        }
    }


    public void navigation(String path) {
         startActivity(path);
    }

    private void startActivity(String path) {
        String classPath
                = mapHolder.get(path);
        if (!TextUtils.isEmpty(classPath)) {
            Intent intent = new Intent();
            intent.setClassName(application, classPath);//设置包路径
            ActivityCompat.startActivity(application, intent, null);
        }else {
            Toast.makeText(application, "路径空啦", Toast.LENGTH_SHORT).show();
        }
    }


    /**
     * 通过指定包名，扫描包下面包含的所有的ClassName
     *
     * @param context     U know
     * @param packageName 包名
     * @return 所有class的集合
     */
    private static Set<String> getFileNameByPackageName(Context context, final String packageName) throws PackageManager.NameNotFoundException, IOException, InterruptedException {
        final Set<String> classNames = new HashSet<>();

        List<String> paths = getSourcePaths(context);
        final CountDownLatch parserCtl = new CountDownLatch(paths.size());

        for (final String path : paths) {
            DefaultPoolExecutor.getInstance().execute(new Runnable() {
                @Override
                public void run() {
                    DexFile dexfile = null;

                    try {
                        if (path.endsWith("EXTRACTED_SUFFIX")) {
                            //NOT use new DexFile(path), because it will throw "permission error in /data/dalvik-cache"
                            dexfile = DexFile.loadDex(path, path + ".tmp", 0);
                        } else {
                            dexfile = new DexFile(path);
                        }

                        Enumeration<String> dexEntries = dexfile.entries();
                        while (dexEntries.hasMoreElements()) {
                            String className = dexEntries.nextElement();
                            if (className.startsWith(packageName)) {
                                classNames.add(className);
                                try {
                                    Class clazz = Class.forName(className);
                                    Object obj = clazz.newInstance();
                                    Field field03 = clazz.getDeclaredField("holder"); // 获取属性为id的字段
                                    String value= (String) field03.get(obj);
                                    String[] split = value.split(":");
                                    if (split!=null&&split.length==2) {
                                        mapHolder.put(split[0],split[1]);
                                    }
                                    Log.i("test-->",mapHolder.toString());
                                } catch (ClassNotFoundException e) {
                                    e.printStackTrace();
                                } catch (IllegalAccessException e) {
                                    e.printStackTrace();
                                } catch (InstantiationException e) {
                                    e.printStackTrace();
                                } catch (SecurityException e) {
                                    e.printStackTrace();
                                } catch (NoSuchFieldException e) {
                                    e.printStackTrace();
                                } catch (IllegalArgumentException e) {
                                    e.printStackTrace();
                                }
                            }
                        }
                    } catch (Throwable ignore) {
                        Log.e("ARouter", "Scan map file in dex files made error.", ignore);
                    } finally {
                        if (null != dexfile) {
                            try {
                                dexfile.close();
                            } catch (Throwable ignore) {
                            }
                        }

                        parserCtl.countDown();
                    }
                }
            });
        }

        parserCtl.await();

        return classNames;
    }
    private static List<String> getSourcePaths(Context context) throws PackageManager.NameNotFoundException, IOException {
        ApplicationInfo applicationInfo = context.getPackageManager().getApplicationInfo(context.getPackageName(), 0);
        List<String> sourcePaths = new ArrayList<>();
        sourcePaths.add(applicationInfo.sourceDir); //add the default apk path
        return sourcePaths;
    }
}
```

### 总结

- ARouter使用指南
- ARouter拦截器
- SchemeFilte 实现外部html 跳转Native，打通WEB&Native
- 了解**[JavaPoet](https://github.com/square/javapoet)** &[AutoService 注解处理器](https://github.com/google/auto) apt原理
- 写出简易版CRouter，通过实战我们了解ARouter实现原理
- [项目demo地址](https://github.com/Oslanka/ArouterDemo)

### 问题

- 除了ARouter，你知道利用apt 实现的框架都有哪些？
- ARouter有没有什么缺点？

### 引用

- https://github.com/alibaba/ARouter

- https://github.com/square/javapoet
- https://github.com/google/auto
- https://github.com/Oslanka/statichtml.github.io
- https://github.com/Oslanka/ArouterDemo
