# xcx_authorization
微信小程序授权


//放在App.js

```js
/*****************zly *********/
  //获取用户授权信息
  newUserLogin: function (callback = function (res) { }) {
    var that = this
    //1、调用微信登录接口，获取code
    wx.login({
      success: function (r) {
        var code = r.code;//登录凭证
        if (code) {
          //2、调用获取用户信息接口
          wx.getUserInfo({
            withCredentials:true,
            success: function (res) {
              //3.请求自己的服务器，解密用户信息 获取unionId等加密信息
              wx.request({
                url: CONFIG.API_URL.getSessionNew,//自己的服务接口地址
                method: 'post',
                header: {
                  'content-type': 'application/x-www-form-urlencoded'
                },
                data: {
                  encryptedData: res.encryptedData,
                  iv: res.iv,
                  code: code,
                  avatarUrl: res.userInfo.avatarUrl,
                  nickName: res.userInfo.nickName,
                  gender: res.userInfo.gender
                },
                success: function (data) {
                  console.log("返回", data)
                  //4.解密成功后 获取自己服务器返回的结果
                  if (data.data.status == 1) {
                    var userInfo_ = data.data.userInfo;
                    wx.setStorageSync('userInfo', userInfo_)//存缓存
                    that.globalData.userInfo = userInfo_;//存全局
                    //查找权限
                    // that.getUserAuth(userInfo_.id);
                    if (that.globalData.sid){
                      that.saveSid();//储存上级用户sid
                    }
                    callback(userInfo_);
                  } else {
                    console.log('解密失败1')
                  }

                },
                fail: function () {
                  console.log('系统错误')
                }
              })
            },
            fail: function () {
              that.newUserLogin(callback);

              ////重新授权
              // wx.openSetting({
              //   success: (data) => {
              //     if (data) {
              //       if (data.authSetting["scope.userInfo"] == true) {
              //         wx.getUserInfo({
              //           withCredentials: false,
              //           success: function (data) {
              //             console.info("3成功获取用户返回数据");

              //             //3.请求自己的服务器，解密用户信息 获取unionId等加密信息
              //             wx.request({
              //               url: CONFIG.API_URL.getSessionNew,//自己的服务接口地址
              //               method: 'post',
              //               header: {
              //                 'content-type': 'application/x-www-form-urlencoded'
              //               },
              //               data: {
              //                 code: code,
              //                 avatarUrl: data.userInfo.avatarUrl,
              //                 nickName: data.userInfo.nickName,
              //                 gender: data.userInfo.gender
              //               },
              //               success: function (data) {

              //                 //4.解密成功后 获取自己服务器返回的结果
              //                 if (data.data.status == 1) {
              //                   var userInfo_ = data.data.userInfo;
              //                   wx.setStorageSync('userInfo', userInfo_)
              //                   that.globalData.userInfo = userInfo_;
              //                   if (that.globalData.sid) {
              //                     that.saveSid();//储存上级用户sid
              //                   }
              //                   callback(userInfo_);
              //                 } else {
              //                   console.log('解密失败2')
              //                 }

              //               },
              //               fail: function () {
              //                 console.log('系统错误')
              //               }
              //             })

              //           },
              //           fail: function () {
              //             console.info("3授权失败返回数据");

              //           }
              //         });
              //       }
              //     }
              //   }
              // });

            }
          })

        } else {
          console.log('获取用户登录态失败！' + r.errMsg)
        }
      },
      fail: function () {
        console.log('登陆失败')
      }
    })
  },
```

Wxml文件：

```html
      <button class='topinfo-1' open-type="getUserInfo" bindtap='getAuth' plain='true' style='border:none;'>
        <image src='{{userdata.avatarUrl}}' id='Headportrait'></image>
        <text class='overflow'>{{userdata.nickname}}</text>
      </button>
```

Js文件：

```js
  //点击授权
  getAuth:function(){
    var that=this;
app.newUserLogin(function (res) {
	console.log(res)
    })
  },
```



