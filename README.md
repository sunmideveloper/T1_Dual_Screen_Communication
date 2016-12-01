# T1_Dual_Screen_Communication
the basic use of DSD

# 商米T1有两种双屏配置：

1 . 主屏14寸，副屏7寸。


![Overload Sunmi](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/1.png) 


2 .主屏14寸，副屏14寸。

![Overload Sunmi](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/2.png) 


主副屏运行的都是SUNMIUI系统，通过上商米封装好的接口完成T1双屏通讯。

# 关于商米T1副屏内置显示程序

通常开发者的业务app运行在主屏，同时需要副屏显示清单信息、宣传图片、宣传幻灯片、宣传视频等内容。开发者有两种方式使用T1副屏显示内容：

1 商米在T1副屏系统中内置了1个显示程序(以下简称'副显程序')，主屏的app遵照商米统一的规范，向副显程序发特定格式的数据，就可以让副显程序显示内容。（开发成本低，推荐）。
2 开发者自己开发副屏显示app（开发成本高，不推荐）。
# 如何使用商米副显程序
商米规定了副显程序能显示的布局，开发者修改自己的主屏app的代码，向副显程序传递相关格式的数据即可。

下面将按照如下三段讲解如何使用副显程序：
一. 初始化配置。
二 .7寸屏的3种布局和传递数据的代码。
三 .14寸屏的7种布局和传递数据的代码。

## 一.初始化配置

操作7寸和操作14寸的初始化流程一样，请下载资源文件，参照Demo项目，直接将jar包导入到主屏app项目中或在Android Studio的app module下的build.gradle文件中声明。
```
dependencies {
    compile 'com.sunmi:DS_Lib:1.0.2'
    compile 'com.google.code.gson:gson:2.6.2'//gson任意版本
}
```
在清单文件AndroidMainfest.xml的<application>节点下配置以下声明：
```
<application>
  ....
          <receiver
              android:name="sunmi.ds.MsgReceiver">
              <intent-filter >
                  <action android:name="com.sunmi.hcservice"></action>
                  <action android:name="com.sunmi.hcservice.status"></action>
              </intent-filter>
          </receiver>
  </application>
  ```
  
  
在适当的地方初始化SDK代码如下，可以参考Demo中的代码，
``` 
DSKernel mDSKernel =DSKernel.newInstance();
mDSKernel.init(context, mConnCallback);
mDSKernel.addReceiveCallback(mReceiveCallback); 
```
