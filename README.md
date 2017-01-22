### 微信小程序用户资源上传七牛云示例

示例展示了在微信小程序中，用户如何将资源上传到七牛云的存储空间

准备工作

1. 七牛账号 [点击注册](https://portal.qiniu.com/signup/choice)

2. 登陆 [七牛云控制台](portal.qiniu.com) 点击 对象存储 新建 存储空间

3. 开发设置中的服务器域名需配置如下

   | request合法域名                              | uploadFile合法域名                           |
   | ---------------------------------------- | ---------------------------------------- |
   | [https://api.qiniudemo.com](https://api.qiniudemo.com) 服务器端获取UpToken地址 | [https://upload.qbox.me](https://upload.qbox.me) 空间所在机房对应的上传域名，参考文档 [上传域名](https://support.qiniu.com/question/226014) |
   |                                          |                                          |

### 代码结构

```
├── app.js
├── app.json
├── app.wxss
├── LICENSE
├── pages
│   └── index
│       ├── index.js
│       ├── index.wxml
│       └── index.wxss
└── utils
    └── util.js
```



### 核心代码

pages/index/index.js

```javascript
//index.js
//获取应用实例
var app = getApp()
Page({
  data: {
    motto: 'Click Me!',
    userInfo: {}
  },
  //事件处理函数
  bindViewTap: function() {
    wx.navigateTo({
      url: '../logs/logs'
    })
  },
  onLoad: function () {
    console.log('onLoad')
    var that = this
    //调用应用实例的方法获取全局数据
    app.getUserInfo(function(userInfo){
      //更新数据
      that.setData({
        userInfo:userInfo
      })
    })
  },
  bindUploadFile: function(){
    wx.chooseImage({
      count: 2, 
      sizeType: ['original', 'compressed'], // original 原图，compressed 压缩图，默认二者都有
      sourceType: ['album', 'camera'], // album 从相册选图，camera 使用相机，默认二者都有
      success: function(res){
        // success
        var tempFilePaths=res.tempFilePaths
        var count=tempFilePaths.length;
        //依次上传选中的文件
        console.log("select file count "+count);
        for(var i=0;i<count;i++){
          //获取文件临时路径
          var origFilePath=tempFilePaths[i];
          console.log("temp file paths "+origFilePath);
         
          var fileKey="2017/01/qiniu/demo/test"+i+".jpg";

           // 该uptoken请从业务服务器端的API获取
          wx.request({
            url: 'https://api.qiniudemo.com/upload/api/simple_upload/overwrite_existing_file_upload_token.php',
            data: {
              'key': fileKey,
            },
            method: 'GET', 
            header: {
              'Content-Type':'application/x-www-form-urlencoded'
            }, // 设置请求的 header
            success: function(res){
               // success
               var upToken=res.uptoken;
               //开始上传文件
                console.log("start to upload file "+origFilePath);
                wx.uploadFile({
                //根据空间选择对应的https上传加速域名
                url: 'https://upload.qbox.me',
                //上传原始文件
                filePath: origFilePath,
                //名称必须是file
                name:'file',
                // header: {}, // 设置请求的 header
                formData: {
                  'token': upToken,
                  //该名称为文件保存在七牛空间中的文件名，可以客户端指定
                  'key': fileKey,
                  //扩展参数，根据需要设置，不需要则删除
                  'x:year':2017,
                  'x:author':'jemygraw',
                  'x:firm':'qiniu cloud'
                }, // HTTP 请求中其他额外的 form data
                success: function(res){
                  // success，打印七牛回复或者业务服务器的回调七牛服务器的JSON内容
                  console.log("upload file "+origFilePath+" success");
                  console.log(res);
                },
                fail: function() {
                  // fail
                  console.log("upload file "+origFilePath+" failed");
                },
                complete: function() {
                  // complete
                  console.log("upload file "+origFilePath+" complete");
                }
              })
            },
            fail: function() {
              // fail
            },
            complete: function() {
              // complete
            }
          })
        }
      },
      fail: function() {
        // fail
      },
      complete: function() {
        // complete
      }
    })
  }
})
```
