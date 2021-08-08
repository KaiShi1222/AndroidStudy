1. **Android应用打包** 

   * 通过aapt打包资源文件, 这个过程中AndroidManifest.xml和布局XML都会被编译生成R.java文件

   * 通过aidl解析aidl文件生成相应的java文件

   * 通过java编译器将java文件编译成.class文件

   * 通过dex工具将.class文件, jar, aar等转换成可以在Android虚拟机上可以执行的文件

   * 通过apkbuilder打包成apk文件

   * 将apk进行签名, 分为release和debug, 调试阶段，比如我们直接通过AS run的就是debug版本

     release需要开发者具体配置

   * 最后会通过zipalign对资源进行优化，减少内存的使用

     资源文件距离文件起始偏移为4字节整数倍, 这样不需要遍历，只需要跳转4*N访问资源即可

   

   Ref:

   1. https://www.jianshu.com/p/019c735050e0

   2. https://www.cnblogs.com/xunbu7/p/7345912.html

      

2.  **Android加载大图**

   * 官方文档建议使用Glide或者其他第三方库

   * 通过Bitmap加载

     * 读取Bitmap尺寸和类型

       ```java
       BitmapFactory.Options options = new BitmapFactory.Options();
       // 避免内存分配
       options.inJustDecodeBounds = true;
       BitmapFactory.decodeResource(getResources(), R.id.myimage, options);
       int imageHeight = options.outHeight;
       int imageWidth = options.outWidth;
       String imageType = options.outMimeType;
       ```

     * 计算采样率

       根据采样率的规则和目标View的大小, 计算出采样率

       ```java
          public static int calculateInSampleSize(
                       BitmapFactory.Options options, int reqWidth, int reqHeight) {
               // Raw height and width of image
               final int height = options.outHeight;
               final int width = options.outWidth;
               int inSampleSize = 1;
       
               if (height > reqHeight || width > reqWidth) {
       
                   final int halfHeight = height / 2;
                   final int halfWidth = width / 2;
       
                   // Calculate the largest inSampleSize value that is a power of 2 and keeps both
                   // height and width larger than the requested height and width.
                   while ((halfHeight / inSampleSize) >= reqHeight
                           && (halfWidth / inSampleSize) >= reqWidth) {
                       inSampleSize *= 2;
                   }
               }
       
               return inSampleSize;
           }
       ```

     * 按照新的采样率加载Bitmap

       ```java
           public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
                   int reqWidth, int reqHeight) {
       
               // First decode with inJustDecodeBounds=true to check dimensions
               final BitmapFactory.Options options = new BitmapFactory.Options();
               options.inJustDecodeBounds = true;
               BitmapFactory.decodeResource(res, resId, options);
       
               // Calculate inSampleSize
               options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
       
               // Decode bitmap with inSampleSize set
               options.inJustDecodeBounds = false;
               return BitmapFactory.decodeResource(res, resId, options);
           }
           
       ```

       

3. **Android Clean Architecture**

   这个架构是bob大叔提出的, 和其他架构一样, 为了解决代码的耦合性, 维护性, 方便测试.

   它通过分层来实现, 具体分多少层和具体的使用场景相关. 

   可以把这些层看成多个同心圆，越外层越具体，越内存越抽象. 

   如果这个架构用在Android上来说, 外层是Android相关代码，比如Activity, Fragment, 内层是纯java或者kotlin代码

   * 依赖规则

     层与层的依赖规则是内层不能有外层的引用，变量，函数等等.

     每层只能依赖它的内部下一层

   * Android分层

     在Android中可以通过创建不同的module来实现分层, 具体怎么分没有绝对的标准, 选择适合项目的即可

     比如可以创建domain, data. app这几个Module.

     * Domain

       内部在分为entities, repositories （接口）, usecase等

     * Data

       可以包括具体的network, db操作，并且实现了定义的domain的repository接口
       
       build.gradle中
       
       ```groovy
       implementation project(':domain')
       ```
     
     * App
     
       这一层为Android相关代码，包括Activity, Fragment, Application, Service等. 可以通过MVP, MVVM进一步解耦合
     
       在初始化的适合创建Database, 创建网络请求对象, 创建usecase对象
     
       在生命周期方法中发送数据请求，如果使用MVVM, 通过livedata postValue发送返回的数据, 并在observer方法中接受数据，最终显示在UI上
     
     Ref: 
     
     1. https://medium.cobeisfresh.com/developing-android-apps-with-kotlin-and-clean-architecture-21bc21b2aac2
     2. https://medium.com/android-dev-hacks/detailed-guide-on-android-clean-architecture-9eab262a9011
     3. 

4. 横竖屏切换不重启Activity**

   设置Activity的android:configChanges=”**orientation |keyboardHidden**”时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法

5. **XML解析**

   * [XmlPullParser](https://developer.android.com/reference/org/xmlpull/v1/XmlPullParser)

     通过流的形式，读入要解析的xml, 并解析xml的标签，得到想要的属性值

     比较省内存

   * SAX

     基于事件的解析, 通过读xml, 当收到对应事件的时候，回调方法处理

     通过继承**DefaultHandler**实现自定义handler并实现处理方法来做解析

     startDocument endDocument, startElement, endElement

   * DOM解析

     将整个文档读入内存, 根据节点生成一个DOM树, 把没个节点当作一个对象，进行解析

6. **Android 更新UI方式**
   * runOnUiThread
   * View.post(runnable)
   * AsyncTask
   * Handler
   * RxJava
   * LiveData

7. Retrofit 配置多个baseUrl

   https://juejin.cn/post/6844903635126583303





Ref:

1. https://github.com/JsonChao/Awesome-Android-Interview/blob/master/Android%E7%9B%B8%E5%85%B3/Android%E5%9F%BA%E7%A1%80%E9%9D%A2%E8%AF%95%E9%A2%98.md