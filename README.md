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

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/3.jpg) 

完成了以上的操作，就可以向副屏发送数据了，以下分别讲解7寸屏和14寸屏的显示布局和各布局下要传递的数据格式。

### 二、7寸副显程序能显示以下三种布局
1.全屏显示单张图片
2.全屏只显示简单文字
3.在屏幕部分区域显示图片+文字

以下讲解7寸屏副显程序显示各个布局时，主屏app需要向副显程序传递的数据内容：

#### 1.全屏显示主屏指定的图片(7寸屏)
效果图如下所示：

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/4.jpg) 

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
注意：在所有要发送文件(图片，视频)的地方，建议开发者在发送文件之前调用DSKernel.checkFileExist(long fileId, final ICheckFileCallback callback)检查该文件在副屏是否有缓存，如果有该文件，则可以不用重复传递。检查的代码如下：

```
//说明：该方法检查fileId对应的文件在副屏是否存在，第一个参数为之前发送文件时副屏返回的文件id，该id是所有的文件唯一的，第二个参数为结果回调
mDSKernel.checkFileExist(fileId, new ICheckFileCallback(){
     @Override
     public void onCheckFail() {}
     
     @Override
     public void onResult(boolean arg0) {
        //返回值为true是存在
        //返回值false是不存在  
         }                                    
    });
```
#### 2.全屏只显示简单文字(7寸屏)

显示标题和内容两行字符，布局格式是商米规定好的，主屏app只要向副显程序发送json格式的数据即可。

效果图如下：

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/5.jpg) 

发数据时代码如下：
```
JSONObject json = new JSONObject();
json.put("title", title);//title为上面一行的标题内容
json.put("content", content);//content为下面一行的内容
String jsonStr = json.toString();
//构建DataPacket类
DataPacket packet = UPacketFactory.buildShowText(
        DSKernel.getDSDPackageName(), jsonStr, callback);//第一个参数是接收数据的副显应用的包名，这里参照Demo就可以,第二个参数是要显示的内容字符串，第三个参数为结果回调。

mDSKernel.sendData(packet);//调用sendData方法发送文本
```

#### 3.在屏幕部分区域显示图片+文字(7寸屏)

布局格式也是商米规定好的，目前商米建议7寸屏的该种布局用来显示如下的信息：图片为二维码，右边的文字只有标题和内容两行。

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/7.jpg) 

发数据代码如下：

```
JSONObject json = new JSONObject();
json.put("title", title);//title为上面一行的标题内容
json.put("content", content);//content为下面一行的内容
String titleContentJsonStr= json.toString();
 mDSKernel.sendFile(DSKernel.getDSDPackageName(), titleContentJsonStr,filePath, new ISendCallback() {
    @Override
    public void onSendSuccess(long fileId) {
        showQRCode(fileId);//二维码图片发送
    }
    public void onSendFail(int i, String s) {
      //错误回调
    }
    public void onSendProcess(long l, long l1) {
    //发送状态回调
    }
});

private void showQRCode(long fileId) {
      String json = UPacketFactory.createJson(sunmi.ds.data.DataModel.QRCODE,"");
      mDSKernel.sendCMD(DSKernel.getDSDPackageName(), json, fileId, null);
```
    
### 三、14寸副显app能显示以下7种布局

1. 全屏只显示复杂的表格字符数据
2. 屏幕左边显示图片，右边显示复杂的表格数据
3. 屏幕左边显示幻灯片，右边显示复杂的表格数据
4. 屏幕左边显示视频，右边显示复杂的表格数据
5. 全屏显示单张图片
6. 全屏显示幻灯片
7. 全屏显示视频

以下讲解14寸屏副显程序显示各个布局时，主屏app需要向副显程序传递的数据内容：

#### 1.全屏只显示复杂的表格字符数据(14寸屏)

全屏显示表格数据，该显示布局固定三个显示区域：标题区，清单区，结算区。标题区只能显示一串字符；清单区域会显示一个表格，可以显示1到8列字段，行数不限制，但是超过屏幕能显示的行，将自动滚动到最下面的行；结算区域可以显示1到8个字段。格式是商米固定的，但是字段标题和数据内容是主屏app传递的。

效果图如下：

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/7.jpg) 

代码如下：

```
/**
*receiverPackageName 接收数据的副屏显示app的包名DataType.DATA 
*DataType.DATA 
*DataModel.TEXT 
*jsonStr 显示的数据内容,具体格式参见下面的框
*callback 回调
*/
DataPacket pack = buildPack(receiverPackageName, DataType.DATA, DataModel.TEXT, jsonStr, callback);//
mDSKernel.sendData(pack);
```

上面的jsonStr的数据格式如下框所示，请开发者参照如下的格式生成JSON格式的数据.其中title定义标题区显示的内容，head定义清单区中的字段标题，请按参照Demo生成相应的数据，请注意对应的字段个数符合规范，以下字符的key值：title，head，list，KVPlist，name，value是固定的写法，param1,param2...param8是顺序递增的，如果格式不对，将无法显示效果。

```
{
        "title": "三米奶茶店欢迎您", 
        "head": {
            "param1": "序列号",
            "param2": "商品名",
            "param3": "单价"，
            "param4": "数量"，
            "param5": "小结"
        },
        "list": [
            {
              "param1": "1",
              "param2": "华夫饼",
              "param3": "10.00"，
              "param4": "1"，
             "param5":"10.00"
            },
            {
              "param1": "1",
              "param2": "吞拿鱼华夫饼",
              "param3": "12.00"，
              "param4": "1"，
             "param5":"12.00"
            }
            ... ...//这里是相同格式的数据
        ],
        "KVPList": [
            {
                "name": "收款",
                "value": "￥40.00"
            },
            {
                "name": "优惠",
                "value": "￥3.00"
            }，
           {
                "name": "找零",
                "value": "￥3.00"
            }，
           {
                "name": "实收",
                "value": "￥37.00"
            }
        ]
}

数据格式说明：
1:title字段为标题
2:head字段是表头字段
3:list是商品列表，表头和表的字段数量要一致 
4:KVPList为结算键值对列表

规则:
当只显示购物清单时：head的params赋值个数最小1个最大8个;list中每个元素的params赋值个数最小1个最大8个；KVPList的size最小1个最大8个。

```

#### 2.屏幕左边显示图片，右边显示复杂的表格数据(14寸屏)

左边区域显示图片,建议的图片大小为1186*1080；右边区域显示复杂的表格数据，同样也分三个显示区：标题区，清单区，结算区。标题区只能显示一串字符；清单区域会显示一个表格，最多显示4列字段，超过屏幕范围的行数据可滚动显示；结算区域可以显示1到4个字段。

效果图：

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/7.jpg) 

代码如下：

```
String filePath = "xxx/img.png";//显示的图片路径
mDSKernel.sendFile(DSKernel.getDSDPackageName(), filePath, new ISendCallback() {
    public void onSendSuccess(long fileId) {
        show(fileId, jsonStr);//图片发送成功，显示文字内容
    } 
    public void onSendFail(int errorId, String errorInfo) {}
    public void onSendProcess(long total, long sended) {}
});

void show(long fileId, String jsonStr){
    jsonStr = UPacketFactory.createJson(DataModel.SHOW_IMG_LIST, jsonStr);//第一个参数DataModel.SHOW_IMG_LIST为显示布局模式，jsonStr为要显示的内容字符
    mDSKernel.sendCMD(DSKernel.getDSDPackageName(), jsonStr, fileId,null);
}

```

上面的jsonStr的数据格式如下框所示，请开发者参照如下的格式生成JSON格式的数据，其中title定义标题区显示的内容，head定义清单区中的字段标题，请按参照Demo生成相应的数据，请注意对应的字段个数符合规范，以下字符的key值：title，head，list，KVPlist，name，value是固定的写法，param1,param2...param4是顺序递增的。

```
{
        "title": "商米奶茶店欢迎你",
        "head": {
          "param1": "商品名",
            "param2": "单价"，
            "param3": "数量"，
           "param4":"小结"
        },
        "list": [
            {
           
              "param1": "华夫饼",
              "param2": "10.00"，
              "param3": "1"，
             "param4":"10.00"
            },
            {
            
              "param1": "吞拿鱼华夫饼",
              "param2": "12.00"，
              "param3": "1"，
             "param4":"12.00"
            }
            ... ...
        ],
        "KVPList": [
            {
                "name": "收款",
                "value": "￥40.00"
            },
            {
                "name": "优惠",
                "value": "￥3.00"
            }，
           {
                "name": "找零",
                "value": "￥3.00"
            }，
           {
                "name": "实收",
                "value": "￥37.00"
            }
        ]
}

JSON数据格式说明：
1:title字段为标题
2:head字段是表头字段
3:list是商品列表，表头和表的字段数量要一致 
4:KVPList为结算键值对列表

规则:
当显示图文混合时：head的params赋值个数最小1个最大4个;list中每个元素的params赋值个数最小1个最大4个；KVPList的size最小1个最大4个。

```

#### 3.屏幕左边显示幻灯片，右边显示复杂的表格数据(14寸屏)

左边区域显示幻灯片,建议的图片大小为1186*1080；右边区域显示复杂的表格数据，同样也分三个显示区：标题区，清单区，结算区。标题区只能显示一串字符；清单区域会显示一个表格，最多显示4列字段，超过屏幕范围的行数据可滚动显示；结算区域可以显示1到4个字段。


效果图：

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/7.jpg) 

代码如下：

```
List<String> paths = new ArrayList<>();
paths.add(Environment.getExternalStorageDirectory().getPath() + "/sunmi1.png");
paths.add(Environment.getExternalStorageDirectory().getPath() + "/sunmi2.png");
paths.add(Environment.getExternalStorageDirectory().getPath() + "/sunmi3.png");
paths.add(Environment.getExternalStorageDirectory().getPath() + "/sunmi4.png");

mDSKernel.sendFiles(DSKernel.getDSDPackageName(), "", paths, new ISendFilesCallback() {
    @Override
    public void onAllSendSuccess(long fileId) {               
      sendImgsListCMD(fileId,json);                           
    }

    @Override
    public void onSendSuccess(String path, long taskId) {}
    @Override
    public void onSendFaile(int errorId, String errorInfo) {}
    @Override
    public void onSendFileFaile(String path, int errorId, String errorInfo) {}
    @Override
    public void onSendProcess(String path, long totle, long sended) {}
    });
    
    void sendImgsListCMD(long fileId, String jsonStr){   
      jsonStr= UPacketFactory.createJson(DataModel.SHOW_IMGS_LIST, json);
        mDSKernel.sendCMD(DSKernel.getDSDPackageName(), jsonStr, fileId,null);
    }
```
上面的jsonStr的数据格式规则和布局2中的规则一样，在此不做赘述。

#### 4.屏幕左边显示视频，右边显示复杂的表格数据


左边区域显示视频，建议的分辨率大小为1186*1080；右边区域显示复杂的表格数据，同样也分三个显示区：标题区，清单区，结算区。标题区只能显示一串字符；清单区域会显示一个表格，最多显示4列字段，超过屏幕范围的行数据可滚动显示；结算区域可以显示1到4个字段。

效果图：

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/7.jpg)

```
String filePath = "xxx/video.mp4";//显示的视频路径
mDSKernel.sendFile(DSKernel.getDSDPackageName(), filePath, new ISendCallback() {
    public void onSendSuccess(long fileId) {
        show(fileId, jsonStr);//视频发送成功
    } 
    public void onSendFail(int errorId, String errorInfo) {}
    public void onSendProcess(long total, long sended) {}
});

void show(long fileId, String jsonStr){
    jsonStr = UPacketFactory.createJson(DataModel.SHOW_VIDEO_LIST, jsonStr);
    mDSKernel.sendCMD(DSKernel.getDSDPackageName(), jsonStr, fileId,null);
}
```

上面的jsonStr的数据格式规则和布局2中的规则一样，在此不做赘述。
#### 5.全屏显示单张图片(14寸屏)

效果图：
![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/7.jpg)

和7寸屏的写法类似，先发送图片，发送成功后，在回调里面发送控制指令显示图片，代码如下：
```
String filePath = "/sdcard/img.png";
mDSKernel.sendFile(DSKernel.getDSDPackageName(), filePath, new ISendCallback(){
    public void onSendSuccess(long l) {
       showImg(fileId);//先发送图片
    }
    public void onSendFail(int i, String s) {}
    public void onSendProcess(long l, long l1) {}
});

void showImg(long fileId){
    String json = UPacketFactory.createJson(DataModel.SHOW_IMG_WELCOME, "def");
    mDSKernel.sendCMD(DSKernel.getDSDPackageName(), json, fileId, null);
}
```

#### 6.全屏显示视频(14寸屏)

效果图：

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/7.jpg)


第一次显示视频的时候，传递的过程可能会很长，开发者应该在适当的时候先将视频传递给副显程序，副显程序将缓存视频，代码如下：
```
String filePath = "/sdcard/onepiece.mp4";
mDSKernel.sendFile(DSKernel.getDSDPackageName(), filePath, new ISendCallback(){
    public void onSendSuccess(long l) {
       playVideo(fileId);
    }
    public void onSendFail(int errorId, String errorInfo) {}
    public void onSendProcess(long total, long sended) {}
});

void playVideo(long fileId){
    String json = UPacketFactory.createJson(DataModel.VIDEO, "");
    
#### 7.全屏显示幻灯片(14寸屏)

效果图如下：

![Alt SUNMI](https://github.com/sunmideveloper/T1_Dual_Screen_Communication/blob/master/image/7.jpg)


开发者需要准备好多张图片，调用sendFiles方法传递多张图片的路径给副显程序，幻灯片的默认切换时间是10秒，主屏app可以通过传递参数改变时间，详情请参考如下代码
```
JSONObject json = new JSONObject();
json.put("interval",5000); //幻灯片的切换时间，用毫秒计算，如果不传默认是10000毫秒
List<String> pathList = new ArrayList<>();
pathList.add("/sdcard/img1.png");
pathList.add("/sdcard/img2.png");
...
mDSKernel.sendFiles(DSKernel.getDSDPackageName(), json.toString(), pathList, new ISendFilesCallback() {
    public void onAllSendSuccess(long fileId) {
        show(fileId);
    }
    public void onSendSuccess(final String s,final long l) {}
    public void onSendFaile(int errorId, String errorInfo) {}
    public void onSendFileFaile(String path, int errorId, String errorInfo){}
    public void onSendProcess(String path, long total, long sended) {}
});

private void show(long fileId) {
    String json = UPacketFactory.createJson(DataModel.IMAGES,"");
    mDSKernel.sendCMD(DSKernel.getDSDPackageName(),json,fileId,null);
}
```
    mDSKernel.sendCMD(DSKernel.getDSDPackageName(), json, fileId, null);
}
```

补充说明：
以上为商米T1内置显示程序目前支持的布局，更多的布局将在后续添加，开发者也可以自己开发适配商米T1的副显程序。
