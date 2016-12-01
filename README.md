# T1双屏通信
the basic use of DSD

## 商米T1有两种双屏配置：

1 . 主屏14寸，副屏7寸。


![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/1.png) 


2 .主屏14寸，副屏14寸。

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/2.png) 


主副屏运行的都是SUNMIUI系统，通过上商米封装好的接口完成T1双屏通讯。

## 关于商米T1副屏内置显示程序

通常开发者的业务app运行在主屏，同时需要副屏显示清单信息、宣传图片、宣传幻灯片、宣传视频等内容。开发者有两种方式使用T1副屏显示内容：

1 商米在T1副屏系统中内置了1个显示程序(以下简称'副显程序')，主屏的app遵照商米统一的规范，向副显程序发特定格式的数据，就可以让副显程序显示内容。（开发成本低，推荐）。
2 开发者自己开发副屏显示app（开发成本高，不推荐）。
## 如何使用商米副显程序
商米规定了副显程序能显示的布局，开发者修改自己的主屏app的代码，向副显程序传递相关格式的数据即可。

下面将按照如下三段讲解如何使用副显程序：
一. 初始化配置。
二 .7寸屏的3种布局和传递数据的代码。
三 .14寸屏的7种布局和传递数据的代码。

### 一.初始化配置

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

注：主副屏通信相关的接口请看资源文件中的T1双屏通信接口文档，副屏客显操作指令工具包见Demo：sunmi.ds.data中的代码是对操作副屏客显指令的封装也就是框架图中的APP协议层。

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/3.png) 

完成了以上的操作，就可以向副屏发送数据了，以下分别讲解7寸屏和14寸屏的显示布局和各布局下要传递的数据格式。

### 二、7寸副显程序能显示以下三种布局
1.全屏显示单张图片
2.全屏只显示简单文字
3.在屏幕部分区域显示图片+文字

以下讲解7寸屏副显程序显示各个布局时，主屏app需要向副显程序传递的数据内容：

#### 1.全屏显示主屏指定的图片(7寸屏)
效果图如下所示：

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/4.png) 

要显示以上布局，只要调用DSKernel的sendFile方法向副显程序发一张图片即可。建议图片分辨率1024*600，或者相应长宽比的图片，否则将被缩放，图片是即时传即时显示的，并会被缓存起来重复使用，也就是说，理论上可以不用每次向副显程序传递图片。

可以参加见Demo中关于7寸副显相关的代码：

发数据时代码如下：
```
String filePath = "xxx/img.png";//这个是要显示的图片的本地路径
//通过DSKernel的sendFile发送文件的方法向副显程序发图片
mDSKernel.sendFile(DSKernel.getDSDPackageName(), filePath, new ISendCallback(){
    public void onSendSuccess(long l) {
    //发送成功的回调，参数long l为文件在副屏的唯一表示符，开发者可以将该字段保存在本地，下次可以通过该字段查询该文件在副显程序中是否存在。
       showImg(fileId);//发送命令，让副屏显示图片
    }
    public void onSendFail(int i, String s) {
    //发送失败的回调
    }
    public void onSendProcess(long l, long l1) {
    //发送状态的回调
    }
});

void showImg(long fileId){
    String json = UPacketFactory.createJson(DataModel.SHOW_IMG_WELCOME, "def");
    mDSKernel.sendCMD(DSKernel.getDSDPackageName(), json, fileId, null);//该命令会让副屏显示图片
}
```
