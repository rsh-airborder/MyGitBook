<!--
 * @Author: kendrick任
 * @Date: 2020-12-18 13:03:08
 * @LastEditTime: 2020-12-25 13:13:54
 * @Description: 版本申明
 * @FilePath: \gitbook\wxapp\CommonQuestion\Scan.md
 * @
-->
# 扫码

#### 小程序内部扫码

调用```wx.scanCode()```方法，[官方文档](https://developers.weixin.qq.com/miniprogram/dev/api/device/scan/wx.scanCode.html)，所有的成功回调都在```success```里，例如只允许扫小程序码，可以在返回值的```scanType```里判断，例：
```javascript
wx.scanCode({
    success: res => {
        let type = res.scanType
        if(type === 'WX_CODE'){
        	//执行其他操作
        }else{
        	//执行不是小程序码的操作，例如文字提示
        }
    }
})
```
##### 值的获取
正常情况下，二维码携带的值是在返回值的```result```里返回的，仅需```decodeURIComponent(res.result)```即可解码获取值。

还有一种情况，后端将值拼接在```path```里返回了，这种情况下，需用```split()```方法将传参先从```res.path```里切割下来，再用```decodeURIComponent()```方法解码获取值。

#### 微信扫码进小程序

微信扫码进入小程序需先判断[场景值](https://developers.weixin.qq.com/miniprogram/dev/reference/scene-list.html)，根据场景值列表存在三种场景，```1047:扫描小程序码```，```1048:长按图片识别小程序码```，```1049:扫描手机相册中选取的小程序码```，在```app.js```中的```onShow```方法里通过```scene```判断：
```javascript
onShow: function (res) {
	if(res.scene === 1047 || res.scene === 1048 || res.scene === 1049){
		//为外部扫码进入
	}else{
		//其他
	}
}
```
##### 值的获取
仍旧是在```app.js```中的```onShow```方法里，但通过```query.scene```判断:
```javascript
onShow: function (res) {
	if(res.scene === 1047 || res.scene === 1048 || res.scene === 1049){
		let scanResult = decodeURIComponent(res.query.scene)
		//scanResult为小程序码中的参数，执行后续操作
	}else{
		//其他
	}
}
```